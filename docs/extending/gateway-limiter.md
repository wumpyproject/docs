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

- The ratelimiter itself is an asynchronous context manager which is entered
  at startup and exited when the gateway disconnects at shutdown. This
  asynchronous context manager allows you to setup background tasks or create
  connections used for the ratelimiting.

- Every time a gateway command is sent over the gateway, the ratelimiter will
  be called with the opcode. This is when the second asynchronous context
  manager should be returned and do the pre-emptive waiting upon being entered.

### Interface example

Consider starting with the following bit of code that implements the interface
used by `wumpy-gateway`:

```python
class GatewayLimiterImplementation:

    @asynccontextmanager
    async def __call__(self, opcode: Opcode) -> AsyncGenerator[None, None]:
        # Note that the return type is for @asynccontextmanager, it transforms
        # it into an actual asynchronous context manager.

        ...  # Code that delays the gateway command

        yield

    async def __aenter__(self) -> Self:
        return self

    async def __aexit__(
        self,
        exc_type: Optional[Type[BaseException]],
        exc_val: Optional[BaseException],
        exc_tb: Optional[TracebackType]
    ) -> None:
        pass
```
