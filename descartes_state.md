---
name: DesCartes Squad State
description: Persistent state for DesCartes concurrency exploration of mini-redis
type: project
---

# DesCartes Squad — mini-redis Exploration State

**Last updated:** 2026-05-04 (Run 2)

## Targets Identified

| ID | Name | Phase | File | Notes |
|----|------|-------|------|-------|
| T1 | db_key_expiry | Spec | src/db.rs | get() doesn't check expires_at; background purge via Notify |
| T2 | pubsub | Spec | src/db.rs, src/cmd/subscribe.rs | broadcast channels; silent lag drop; channel never cleaned up |
| T3 | server_shutdown | Research | src/server.rs, src/shutdown.rs | broadcast + mpsc graceful shutdown |
| T4 | buffered_client | Research | src/clients/buffered_client.rs | mpsc + oneshot request buffering |
| T5 | connection_limit | Research | src/server.rs | Semaphore permit lifecycle |

## Tests Written

None yet.

## Bugs Found

None yet — DesCartes tests not yet written/run.

## Exploration Statistics

- Seeds explored: 0
- Decision points found: 0
- Tests compiled: 0

## Key Findings from Research

- `Db::get()` does NOT check `entry.expires_at` — returns stale data if background task hasn't purged yet
- All `Db` state mutations use `std::sync::Mutex` (never held across await)
- Background purge task uses `tokio::select!{sleep_until | notified()}` loop
- `set()` drops the mutex before calling `notify_one()` — correct but creates a notify-race window
- Pub/sub silently drops lagged messages (`RecvError::Lagged` → continue)
- `pub_sub` HashMap entries are NEVER removed — channel names accumulate forever (memory leak)
- `BufferedClient` uses mpsc(32) + oneshot per request
- Shutdown: broadcast signals handlers, mpsc drain waits for all handlers to finish

## Spec Files

- `descartes-exploration/specs/db_key_expiry.md` — complete spec with 5 interleaving scenarios
- `descartes-exploration/specs/pubsub.md` — complete spec with 6 interleaving scenarios, 3 known issues

## Files Created This Run

- `descartes-exploration/RESEARCH.md`
- `descartes-exploration/TARGETS.md`
- `descartes-exploration/specs/db_key_expiry.md` (recreated — dir was missing)
- `descartes-exploration/specs/pubsub.md` (new)

## Next Run Priorities

1. Task 3: Write DesCartes test for T1 (db_key_expiry) — spec is ready
2. Task 4: Run exploration on T1 test
3. Task 3: Write DesCartes test for T2 (pubsub) — spec is ready
