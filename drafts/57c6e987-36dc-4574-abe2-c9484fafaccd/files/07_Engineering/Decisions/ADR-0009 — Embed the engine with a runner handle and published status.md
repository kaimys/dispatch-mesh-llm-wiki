---
id: ADR-0009
status: accepted
date: 2026-11-05
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0009 — Embed the engine with a runner handle and published status

## Context

The 0.3 tray needs one way to observe and control a workspace engine, while CLI intent, finish and status commands keep working, with no duplicate engines and no lost edits. Before 0.2.5, the run loop lived in `src/main.rs`. It stopped only on Ctrl+C, which cannot be sent reliably to another process on Windows, and most runner failures reached only the console.

Decision provenance: Kai's answers recorded in [[Story - US00002 - Local engine control interface for the tray host|US00002]] on 2026-11-05. Recorded as `proposed` on 2026-11-05.

**Amended 2026-11-05 and accepted:** the decision below describes the API as built and verified under US00002 (commits `f2b3e1c`, `0e4c9a6`, reviewed clear, manual test plan passed, CI green on Windows and Linux). The amendment adds the `acquire`/`run(lease, …)` split, the optional initial snapshot, stop-on-drop and the runtime construction site. None of these changes the agreed model.

## Decision

The engine runs **in-process** in the host. The run loop is a library module (`dispatch_mesh::runner`) shared by CLI `run` and desktop hosts:

- **`acquire(root) -> RunnerLease`** takes the exclusive `.mesh/runner.lock` and starts the file watcher. When another runner holds the lock (CLI `run` or another handle, in this or another process), it fails with the typed error **`AlreadyRunning`** and starts nothing. A host explains the refusal and lets the user retry. There is no observer mode and no takeover.
- **`run(lease, shutdown, status) -> Result<()>`** is the async loop. `shutdown` is any future: CLI `run` passes Ctrl+C, and a handle passes its stop flag. When `shutdown` completes, the loop captures saved changes, waiting up to 10 s for a briefly held operation lock, and returns. It never awaits in-flight Git or AI work.
- **`Runner::start(root) -> RunnerHandle`** calls `acquire` and builds a current-thread Tokio runtime on the calling thread, so both errors are returned synchronously. It then moves both into a background thread that runs `run`. The caller needs no Tokio runtime. After the loop ends, the runtime is shut down in the background, so `join()` never waits for detached network work.
- **The `RunnerHandle` methods:**
  - `status()` returns a `watch::Receiver<Option<RunnerSnapshot>>`. Its value is `None` only until the first tick, which follows start immediately.
  - `stop()` is idempotent.
  - `join()` returns how the loop ended.
  - Dropping the handle stops and joins it, so a host that exits still gets the final capture.
- **Snapshot publication.** After each tick, the runner computes the workspace `Status` (ADR-0010) from the state it has just loaded, which includes changes made by CLI processes. It does so through `Workspace::status_from(LocalObservation)`, so publication costs no extra folder scan. When the operation lock is busy, it reads the committed state lock-free instead (ADR-0007). The `RunnerSnapshot` adds the runner's current capture, sync and summary errors, and it is replaced only when it differs from the last one. A lock that is merely busy is never reported as an error.
- **Failure reporting.** AI-summary failures are persisted in workspace state, and CLI `status` shows them. Messages never contain credentials or provider response bodies. There is no separate event stream.
- **CLI coexistence** stays on-disk: the existing operation and network locks, with no IPC. CLI changes reach a running engine within its tick and sync cadence.

## Consequences

**Positive**

UI toolkits keep their main thread and never need Tokio. One loop serves CLI and hosts, so their behavior cannot diverge. Hosts are notified only on change, at no extra scan cost. Shutdown no longer fails when another operation briefly holds the lock, which also fixes CLI `run`.

**Negative**

- A crash of the host process also stops capture until the host restarts. Captured work survives through the existing journals (ADR-0007).
- A detached Git process can outlive `join()` for as long as Git's kill timeout. The locks serialize it against a restarted runner.
- The receiver offers no push callback: a UI event loop checks `has_changed()` itself (an in-memory check) or awaits `changed()`. An `on_change` callback is an additive extension for US00001 if a toolkit needs it.
- The loop still prints its transition messages to stdio, which a GUI-subsystem host discards.
- Hosts cannot control a runner they did not start.

## Alternatives considered

- **A supervised child `run` process or an independently started runner with local IPC:** deferred. It would isolate a UI crash from capture, but CLI changes are already picked up within the sync cadence, and no 0.3 flow needs cross-process control. **Extension point:** a named pipe on Windows or a Unix socket elsewhere, restricted to the current user, if a later milestone needs to attach to a CLI-started runner (for example, the 2.0 MCP host).
- **UI polling of the SQLite state:** rejected in favor of runner-published snapshots, which cost no extra reads and give no stale intervals.
- **An event stream of failures:** rejected. Persisted state plus the snapshot already reach every reader, including CLI `status`.
- **Observer attachment, or automatic takeover of a locked runner:** rejected. The host explains and retries after the existing owner stops.
- **A single `run(root, …)` entry point that takes the lock itself:** replaced by `acquire` + `run(lease, …)`, so that the CLI can print its banner between taking the lock and starting the loop, and `Runner::start` can fail synchronously.

## Related

[[Story - US00002 - Local engine control interface for the tray host]] (origin and verification); [[Story - US00001 - Tray icon and status window host]] (consumer); [[ADR-0010 — Report workspace status and session stages separately]] (the `Status` carried); [[ADR-0007 — Persist state transactionally with recovery journals]] (lock-free reads, survival of captured work); [[ADR-0005 — Make AI summaries configurable and opt-in]] (summary failures never block publication).
