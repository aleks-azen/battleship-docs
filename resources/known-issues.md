# Known Issues

## Lambda reserved concurrency quota too low

The architecture relies on Lambda reserved concurrency set to 1 (ADR-08) to serialize all requests through a single instance, which makes the in-memory per-gameId locking work correctly. During AWS account setup, the unreserved account concurrency ended up at 10 — below the standard default — and AWS won't allow setting reserved concurrency to 1 when the unreserved pool is this low (it needs headroom for other functions).

A quota increase has been requested. Until it goes through, the Lambda runs with unreserved concurrency, meaning multiple instances could theoretically handle concurrent requests to the same game. In practice this is unlikely to cause issues — turn-based games rarely produce truly simultaneous requests — but there's a small window for race conditions on rapid-fire polling during multiplayer.

**Status:** Quota increase pending. Once approved, `reservedConcurrentExecutions: 1` will be applied via CDK redeploy.

