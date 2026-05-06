# Research: Concurrency patterns in mini-redis

Summary of findings (scanned repository for tokio concurrency patterns):

- src/server.rs
  - Uses `tokio::select!` to multiplex between socket reads and shutdown signals.
  - Spawns per-connection tasks with `tokio::spawn`.
  - Concurrency points: task spawning, select branches, connection state updates.

- src/db.rs
  - Uses `tokio::select!` and spawns `purge_expired_tasks(shared.clone())`.
  - Timer-based cleanup + concurrent access to key store.

- src/clients/buffered_client.rs
  - Uses `tokio::sync::mpsc::channel`, Sender/Receiver and `tokio::spawn` to run client loop.
  - Concurrency: channel close/flush, message ordering under backpressure.

- tests/ (integration tests)
  - Many tests spawn the server with `tokio::spawn` and interact concurrently.

Notes / quick observations
- No direct matches for `tokio::sync::Mutex` or `RwLock` were found in the codebase (search hit only docs). Shared state is primarily accessed via tasks and channels.
- High-value concurrency hotspots: connection handling (server), expiry/purge in db, mpsc channel usage in buffered client.

Recommendations
- Prioritize modeling `src/server.rs` (connection handling + select!), then `src/db.rs` purge/expiry, then `buffered_client` channels.
- For each, create a DesCartes spec that models select! branches, channel closes, and timer expirations. Add yield points to maximize interleavings.
