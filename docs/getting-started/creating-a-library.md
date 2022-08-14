# Creating a library

Wumpy exposes many lower-level aspects which can be used to implement another
library on-top. It is usually on the higher-most level which differs between
implementations - don't waste time reimplementing aspects that everybody else
has.

This tutorial will walk through using the lower-level `wumpy-gateway` and
`wumpy-rest` to create an incredibly beginner-friendly API.
