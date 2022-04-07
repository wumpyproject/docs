# Gateway limiter

Another entrypoint in the library is how gateway limiting is done. This may not
see as much use as the [REST ratelimiter](rest-ratelimiter.md) but is still
available if needed.

For example, you can use this to implement logging on all gateway commands sent
to get statistics into Grafana. Another use-case is implementing
`max_concurrency` for bigger bots using multiple shards.

## Implementing the interface

The gateway limiter is not very complicated and is built around two
asynchronous context managers:

- It all starts with a callable given the max concurrency bucket calculated by
  `shard_id % max_concurrency` (per the Discord API). This bucket allows
  gateway limiters to follow max concurrency limits. The callable is not
  `await`ed but should return the first asynchronous context manager. Note that
  it is not until the second asynchronous manager detects an IDENTIFY opcode
  that max concurrency ratelimiting should be done. This callable is just the
  easiest way to let the gateway limiter know of which bucket it is in.

- The first asynchronous context manager should be used to setup background
  tasks or setup other required connections. This in turn needs to return
  a callable which takes an `Opcode` instance (or `int`) representing the
  opcode that is about to be sent over the gateway.

- The callable in turn returns the second asynchronous context manager, the
  purpose of which is to do any necessary sleeping before sending the command.
  This time, the asynchronous context manager does not need to return anything
  as the result has no use.

### Interface example

Consider starting with the following bit of code that implements the interface
used by `wumpy-gateway`:

```python
class GatewayLimiterImplementation:

    def __call__(
        self,
        shard: int
    ) -> AsyncContextManager[
        Callable[[Opcode], AsyncContextManager[None]]
    ]:
        # We have no use for the shard ID
        return self

    async def __aenter__(self) -> Callable[[Opcode], AsyncContextManager[None]]:
        return self.acquire

    async def __aexit__(
        self,
        exc_type: Optional[Type[BaseException]],
        exc_val: Optional[BaseException],
        exc_tb: Optional[TracebackType]
    ) -> None:
        pass

    @asynccontextmanager
    async def acquire(self, opcode: Opcode) -> AsyncGenerator[None, None]:
        # Note that the return type is for @asynccontextmanager, it transforms
        # it into an actual asynchronous context manager.

        ...  # Code that delays the gateway command

        yield
```