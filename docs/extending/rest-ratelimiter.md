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
class Ratelimiter(Protocol):
    async def __aenter__(self) -> object:
        ...

    async def __aexit__(
        self,
        exc_type: Optional[Type[BaseException]] = None,
        exc_val: Optional[BaseException] = None,
        exc_tb: Optional[TracebackType] = None
    ) -> Optional[bool]:
        ...

    def __call__(self, route: Route) -> AsyncContextManager[
        Callable[[Mapping[str, str]], Awaitable[object]]
    ]:
        ...
```

It has been designed to allow a variety of uses and implementations, and is by
choice very vague by only using magic methods. In the following examples, the
ratelimiter don't do any ratelimiting - it only implements the right interface.
To get started, create an asynchronous context manager by defining
`__aenter__()` and `__aexit__()`. It should look somewhat like this:

```python
from types import TracebackType
from typing import Optional, Type


class NoOpRatelimiter:
    """Ratelimiter implementation that does nothing; a no-op implementation."""

    async def __aenter__(self) -> object:
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

For every request made (including retries), the ratelimiter will be called
with the `Route` used for the request. This method must be synchronous and
return an asynchronous context manager. If you need `await` during this step,
delay that until the asynchronous context manager is entered by returning a
proxy object or using [`@asynccontextmanager`][ctxlib-acm].

```python
    @asynccontextmanager
    async def __call__(self, route: Route) -> AsyncGenerator[
        Callable[[Mapping[str, str]], Coroutine],
        None
    ]:
        # The return type may look a little weird, but this is how
        # @asynccontextmanager works. You pass it a function that returns an
        # async generator.
        ...
```

It is within this second asynchronous context manager that you acquire the
underlying lock and have the pre-emptive waiting take place for the ratelimits.
Once completed, this should return an asynchronous callable. The point of this
callable is that the ratelimiter gets updated with the headers of the request
where the ratelimit information can be found.

```python
    async def update(self, headers: Mapping[str, str]) -> object:
        pass

    # The return type may look a little weird, but this is how
    # @asynccontextmanager works. You pass it a function that returns an
    # async generator (which yields what the asynchronous context manager
    # then returns).
    @asynccontextmanager
    async def __call__(self, route: Route) -> AsyncGenerator[
        Callable[[Mapping[str, str]], Coroutine],
        None
    ]:
        # This is a no-op ratelimiter, but this would be the place to do the
        # pre-emptive waiting for ratelimits.
        ...

        yield self.update
```

This callable will be called after each request, after which the asynchronous
context manager will be exited and the response returned to the user. This
means that if updating the ratelimiter or exiting the asynchronous context
manager takes a lot of time, it might be best to launch a different task so
that it does not slow down the requester.

Here is a dummy example of a ratelimiter that does no actual ratelimiting,
all it does is implement the necessary behaviour:

```python
class NoOpRatelimiter:
    """Ratelimiter implementation that does nothing; a no-op implementation."""

    async def __aenter__(self) -> Self:
        return self

    async def __aexit__(
        self,
        exc_type: Optional[Type[BaseException]],
        exc_val: Optional[BaseException],
        exc_tb: Optional[TracebackType]
    ) -> object:
        pass

    # The return type may look a little weird, but this is how
    # @asynccontextmanager works. You pass it a function that returns an
    # async generator (which yields what the asynchronous context manager
    # then returns).
    @asynccontextmanager
    async def __call__(self, route: Route) -> AsyncGenerator[
        Callable[[Mapping[str, str]], Coroutine],
        None
    ]:
        # This is a no-op ratelimiter, but this would be the place to do the
        # pre-emptive waiting for ratelimits.
        ...

        yield self.update

    async def update(self, headers: Mapping[str, str]) -> object:
        pass
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
