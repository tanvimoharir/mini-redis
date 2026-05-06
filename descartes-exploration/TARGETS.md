# Targets (prioritized)

1) server-connection-handling
- Path: src/server.rs
- Concurrency interactions: per-connection `tokio::spawn` tasks, `tokio::select!` between socket IO and shutdown signals, shared connection state updates (e.g., frame parsing + write path).
- Invariants: no dropped frames, correct request/response pairing, no double-close or use-after-close of connection state.
- Complexity: medium (many tasks, each connection spawns its own task; interactions limited to per-connection channels and global shutdown).
- Potential bugs: races between shutdown and connection cleanup, lost responses when channel closes, select! branch starvation.

2) db-purge-expiry
- Path: src/db.rs
- Concurrency interactions: background purge task spawned, timers for TTL, concurrent get/put/delete operations.
- Invariants: expired keys are removed exactly once, no stale reads after delete/expire, atomicity of updates.
- Complexity: medium (background task vs client operations; timers introduce ordering).
- Potential bugs: stale reads if purge runs concurrently with write, double-free or missed expiry under races.

3) buffered-client-mpsc
- Path: src/clients/buffered_client.rs
- Concurrency interactions: tokio mpsc Sender/Receiver, spawned client worker tasks.
- Invariants: messages delivered in order, no message loss on shutdown, proper backpressure.
- Complexity: low-medium (channel buffer sizes, close sequencing matter).
- Potential bugs: dropped messages when sender is closed, incorrect handling of Receiver end when task panics.
