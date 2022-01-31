# REST ratelimiter

The `wumpy-rest` subpackage - containing support for the Discord REST API -
allows you to override how it handles the ratelimits Discord enforces.

The default implementation (`DictRatelimiter`) uses in-memory
`WeakValueDictionary`s from [`weakref`][weakref] to store routes and ratelimit
buckets. It supports concurrent requests to the same ratelimit bucket (if
that's allowed according to the headers) and correctly locks appropriate locks
when running into a `429` response. This all makes it a good default that suits
most bots' needs.

If you want different behaviour than what the default implementation provides,
that's when you want to write your own *ratelimiter*.

  [weakref]: https://docs.python.org/3/library/weakref.html

## Understanding Discord ratelimits

Discord's ratelimits can often be seen as very confusing, but you first need to
understand how they work before you can write a pre-emptive ratelimiter.

### Discord's ratelimit buckets

It all starts with a route, which is the method (`GET`, `POST`, `PATCH`, etc.)
plus the path (like `/users/@me`). We can visualize it like this:

```
   Route
---^^^^^-----
Method + Path
```

The next part is a special header that Discord returns - the
`X-RateLimit-Bucket` hash. This hash can be shared by multiple routes, which
means those routes share the same ratelimit:

```
            Hash
   ---------^^^^-------
   Route      +   Route
---^^^^^-----   ---^^^^^----
Method + Path   Method + Path
```

Finally, the very last part that makes up a ratelimit is the major parameters.
This means that two requests - made to two different channels for example -
*go into different ratelimits buckets*. We can now assemble the full ratelimit:

```
                      Ratelimit
            ----------^^^^^^^^^------------
            Hash          +     Major param
   ---------^^^^-------
   Route      +   Route
---^^^^^-----   ---^^^^^----
Method + Path   Method + Path 
```

This assembled information is enough to individually identify the ratelimit
bucket on Discord's end that will affect a particular route.

### Miscellaneous implementation advice

Learning from the implementation of Wumpy's default implementation of the
ratelimiter as well as other libraries' implementations there's a few things
they all share. While it may seem like stating the obvious; they all have two
separated mappings:

- The first one maps a route (method + path) to its `X-RateLimit-Bucket` hash
  that's been received from Discord. This mapping stores its items
  indefinitely, as it can only grow to the size of all routes ever used.

- The second one maps the assembled ratelimit from above to the locks* that
  will correctly queue up requests. This mapping must automatically evict
  items as they are no longer used.

  *It does not need to be a lock; this is implementation-specific. In some
  implementations this is a semaphore, capacity limiter, or queue - it does not
  matter from the point of view of Wumpy.

## Implementing the right interface

From the point of view of `wumpy-rest`, the ratelimiter is any object that
implements the correct magic methods. The official typing annotation for it
looks like this:

```python
# The type allows the usage to the right of the code:
Ratelimiter = AsyncContextManager[  # async with ratelimiter as rl:
    Callable[[Route], AsyncContextManager[  # async with rl(route) as lock:
        Callable[[Mapping[str, str]], Awaitable]  # await lock(headers)
    ]]
]
```

It has been designed to allow a variety of uses and implementations, and is by
choice very vague. To get started, create an asynchronous context manager by
defining `__aenter__()` and `__aexit__()`. It should look somewhat like this:

```python
class NoOpRatelimiter:
    """Ratelimiter implementation that does nothing; a no-op implementation."""

    async def __aenter__(self) -> object:  # TODO
        ...

    async def __aexit__(
        self,
        exc_type: Optional[Type[BaseException]],
        exc_val: Optional[BaseException],
        exc_tb: Optional[TracebackType]
    ) -> object:
        ...
```

The point of this first asynchronous context manager is to allow you make
initial asynchronous setup, such as setting up connections or background tasks.
It is entered on startup of the requester and only exits when the requester is
being closed.

It is now important that this in turn returns a callable that takes a `Route`,
which will be called every time a request is about to be made with the `Route`
instance that will be used. This callable **cannot** be defined with
`async def`; if you need to use `await` during this step consider using
[`@asynccontextmanager`][ctxlib-acm] which also fulfills the next step -
returning *another* asynchronous context manager.

```python
    @asynccontextmanager
    async def acquire(self, route: Route) -> AsyncGenerator[
        Callable[[Mapping[str, str]], Coroutine],
        None
    ]:
        # The return type may look a little weird, but this is how
        # @asynccontextmanager works. You pass it a function that returns an
        # async generator.
        ...
```

The point of this second asynchronous context manager is now to acquire the
underlying lock, or make the necessary request. This is where the pre-emptive
waiting should take place! Again, this should *in turn* return another
callable - but this time it has to be defined using `async def` so that it is
*awaitable*. This callable is how your ratelimiter is notified of the returned
ratelimit headers and should be where you update internal structures. Note that
the response will not return to the user before this has exited, so consider
starting a task if you know it will take a lot of time to complete.

```python
    async def update(self, headers: Mapping[str, str]) -> object:
        pass

    @asynccontextmanager
    async def acquire(self, route: Route) -> AsyncGenerator[
        Callable[[Mapping[str, str]], Coroutine],
        None
    ]:
        # The return type may look a little weird, but this is how
        # @asynccontextmanager works. You pass it a function that returns an
        # async generator (which yields what the asynchronous context manager
        # then returns).
        yield self.update
```

Finally, the second asynchronous context manager is exited and the response is
returned to the user.

Here is a dummy example of a ratelimiter that does no actual ratelimiting,
all it does is implement the necessary behaviour:

```python
class NoOpRatelimiter:
    """Ratelimiter implementation that does nothing; a no-op implementation."""

    async def __aenter__(self) -> Callable[
        [Route], AsyncContextManager[
            Callable[[Mapping[str, str]], Awaitable]
        ]
    ]:
        return self.acquire

    async def __aexit__(
        self,
        exc_type: Optional[Type[BaseException]],
        exc_val: Optional[BaseException],
        exc_tb: Optional[TracebackType]
    ) -> object:
        pass

    async def update(self, headers: Mapping[str, str]) -> object:
        pass

    @asynccontextmanager
    async def acquire(self, route: Route) -> AsyncGenerator[
        Callable[[Mapping[str, str]], Coroutine],
        None
    ]:
        # The return type may look a little weird, but this is how
        # @asynccontextmanager works. You pass it a function that returns an
        # async generator (which yields what the asynchronous context manager
        # then returns).
        yield self.update

```

  [ctxlib-acm]: https://docs.python.org/3/library/contextlib.html#contextlib.asynccontextmanager

## Handling unexpected responses

Because of how the ratelimiter is built on asynchronous context managers, there
is also a built-in way to handle unexpected responses. Even though the focus of
the ratelimiter is to handle `429` responses, it can also handle any other
unexpected responses as they are also raised as exceptions and passed through
the asynchronous context manager. For example, the default implementation
applies an exponential backoff for `500`, `502` and `504` responses.

The requester will raise a `Ratelimited` exception containing information
returned by the Discord API on specifically `429` responses. This should be
correctly handled according to the Discord API. To then retry the request you
will need to supress the exception **by returning a truthy value**.
`wumpy-rest` will now call the first callable again and enter the asynchronous
context manager (as if it was a new request).

Note that this is how officially asynchronous context managers work -
`@asynccontextmanager` works slightly different. There you want to have a
`try`/`except` block around the `yield`. If an exception is *not* propogated
(meaning that it was captured and handled) then `@asynccontextmanager` will
return a truthy value in your stead.

## Ensuring compatibility

There exists standardized tests for ensuring compatibility with the
`wumpy-rest` usage inside of
[`wumpy-testing`](https://github.com/wumpyproject/wumpy-testing). Refer to the
API reference for how to use it.
