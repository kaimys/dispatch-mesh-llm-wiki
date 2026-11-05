---
id: US00002
type: story
status: "Refinement"
priority: high
rank: 1000
version_target: v0.2.5
size:
assignee:
owner: "[[Kai]]"
open_questions: 1
open_tests:
open_findings:
discussion:
updated: 2026-11-05
completed:
frozen:
---
Define, and record as an ADR, how a desktop host drives the Mesh engine for a workspace: how it reads state, triggers actions such as sync and finish session, and is stopped, while CLI intent and finish commands keep working. Then implement the minimum the 0.3 tray needs. This is the 0.2.x milestone item "Define the local engine/control interface that the tray host will use", and it settles open question 2 of [[Story - US00001 - Tray icon and status window host|US00001]].

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As the developer of the Mesh tray host, I want one defined way to observe and control a running workspace engine, so that the tray, the CLI and later agents all use the same engine operations and state, with no duplicate engines and no lost edits.

## Context

- Milestone source: [[Dispatch Mesh milestone plan]] → *0.2.x — engine stabilization*: "Define the local engine/control interface that the tray host will use. Keep CLI intent and finish actions usable while Mesh runs." The same plan lists "local control interface" among the decisions to close before implementing 0.3. *Product decisions* require the engine to stay independent of the desktop host.
- 0.3 depends on it: the tray must "use the same engine operations and state as the CLI" and "prevent duplicate engine instances for a workspace" ([[Dispatch Mesh milestone plan#0.3 — tray application|0.3]]).
- How processes coexist today (0.2.4):
  - `run` holds an exclusive `fs2` lock on `.mesh/runner.lock`, which already rules out a second runner.
  - Commands that change state take the workspace operation lock (`.mesh/lock`); the capture loop retries contention.
  - Network work takes `.mesh/network.lock`.
  - `status` reads a committed SQLite snapshot without the operation lock.

  So the CLI already works alongside a runner through shared on-disk state, with no IPC. Tests cover this: `runner_startup_survives_a_concurrent_workspace_operation` and `status_remains_readable_while_an_operation_holds_the_workspace_lock` (`tests/cli.rs`).
- What on-disk coexistence does not give a host:
  - **Change notification.** `status` must be polled.
  - **Waking the runner.** For example, to publish right after `session finish` instead of waiting for the next cycle.
  - **Stop and pause.** A way to tell a runner it does not own to stop or pause.
- The decision interacts with US00001's open questions on in-process embedding vs. a supervised `run` process, and on one binary vs. two (a `windows_subsystem = "windows"` binary has no console for the CLI).
- **Refinement findings (2026-11-05, from `src/main.rs` `watch`):**
  - **No wake channel needed.** The runner ticks every second, calls `close_idle` on every tick, and starts a new `sync()` whenever none is running, no capture is pending, and 3 s have passed since the last one. A CLI `session finish` is therefore picked up within seconds without any signal.
  - **Ctrl+C is the only shutdown path.** `watch` stops only on `tokio::signal::ctrl_c()`, which captures and returns. A host needs a shutdown trigger it can fire. Sending Ctrl+C to another process on Windows is unreliable: `GenerateConsoleCtrlEvent` requires a shared console. That favors an in-process engine or an explicit stop command.
  - **Runner failures are console-only.** Except for sync failures, they reach only the console. `report_failure` prints capture, sync and summary failures. Only sync failures are also persisted, as `state.attention = "Synchronization waiting: …"`. AI summary failures ("Summary deferred") are never written to state, so a host reading the status snapshot cannot see them. Offline appears as a sync-failure attention.
  - **The run loop is not in the library.** `watch`, `summarize_ready` and `show_status` live in `src/main.rs`; `src/lib.rs` exports only `ai`, `codec`, `engine`, `git`, `model`, `storage`, `transport`. Sharing them with a host requires moving them into the crate.
- Invariants that bind the design (`AGENTS.md`):
  - Never discard uncaptured edits.
  - Keep capture going while network or AI requests are pending.
  - Never expose credentials or response bodies in diagnostics.
  - Users never need Git knowledge.

## Scope

- An ADR in `07_Engineering/Decisions/` choosing the host↔engine model: in-process library, a supervised runner with local IPC, or on-disk state plus a small signal channel. It covers how state is read, how actions are invoked, how state changes are observed, how a host stops a runner it owns, and how the design extends to macOS and Linux (0.7/0.8).
- Library boundary: the operations a host calls (open, run loop, status snapshot, sync now, finish session, shutdown with capture) exposed from the crate so `main.rs` and a host share one implementation.
- Whatever the ADR selects as the minimum for US00001: state observation without unbounded polling cost, plus orderly shutdown that captures saved changes the way Ctrl+C does.
- Duplicate-engine prevention stays in force for any host, with a clear user-facing error.
- Regression tests: CLI `status`, `session finish` and intent commands work while a host-owned engine runs, and shutdown preserves captured work.

## Out of scope

- The tray UI itself ([[Story - US00001 - Tray icon and status window host|US00001]]).
- The richer status states ([[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]]); this ticket transports whatever the status model contains.
- Pause semantics, which are defined with the 0.3 pause/resume work.
- Remote or authenticated access (cloud MCP, 2.0).

## Open questions

- [x] **Model.** Should the host embed the engine in-process, supervise a `run` child process, or talk to an independently started runner? Embedding is the simplest; supervising isolates a UI crash from capture.
  **Answer (Kai, 2026-11-05):** in-process. The run loop moves into the library with a shutdown handle and a status/event channel. The tray calls it directly, and the CLI keeps working against the same workspace through the existing on-disk locks. Accepted trade-off: a UI crash stops capture until the host restarts, and captured work survives through the existing journals.
- [x] **Observation.** Can hosts poll the committed SQLite snapshot cheaply enough (and how often), or does the engine need to push changes (in-process channel, named pipe, file-change watch on the state DB)?
  **Answer (Kai, 2026-11-05):** the runner publishes. After each tick it computes the status snapshot (the same data `status` returns) and publishes it on a watch channel, but only when it differs from the last one. Changes made by the CLI in another process are included, because the runner re-reads workspace state on every tick. Latency is at most about one tick (1 s), and the host does no polling of its own.
- [x] **Waking and stopping a runner from another process.** Is a control channel needed in 0.3 at all (for example, CLI `session finish` waking the tray-owned runner to publish immediately), or is "the next sync cycle picks it up" acceptable?
  **Answer (follows from the in-process decision and the run-loop finding, 2026-11-05):** no cross-process channel in 0.3. The host stops its own in-process engine through the shutdown handle. The runner already picks up CLI `session finish` within its ~3 s sync cadence.
- [x] **Channel, if any.** Named pipe on Windows and a Unix socket elsewhere, or a localhost TCP port? Either way, how is access restricted to the current user?
  **Deferred (2026-11-05):** not needed with the in-process model. Revisit if a later milestone needs cross-process control (e.g. attaching to a CLI-started runner, or the 2.0 MCP host). The ADR records this as a known extension point.
- [x] **Library API shape.** Should the crate expose a blocking API, an async (tokio) API, or both? The current run loop is tokio-based and `main.rs`-internal.
  **Answer (Kai, 2026-11-05):** both, layered.
  - `Runner::start(root) -> RunnerHandle` runs the loop on a background thread with its own tokio runtime. The handle exposes `status()` (watch receiver of the status snapshot), `stop()` (capture saved changes, then end the loop, as Ctrl+C does today) and `join()`.
  - Underneath, a public async `run(root, shutdown, status_tx)` is what both the handle and the CLI's `run` command use.
  - UI toolkits keep the main thread, and UI code never needs tokio.
- [x] **Failure reporting to hosts.** Should the runner persist capture and AI-summary failures into engine state (as sync failures already are), or emit them as events, so that a host can show the cause of a failed operation? Today they are printed only.
  **Answer (Kai, 2026-11-05):** persist, and include in the snapshot.
  - The latest AI-summary failure is persisted in workspace state, as sync failures already are, so CLI `status` shows it too. It is cleared when a summary succeeds.
  - The runner's current capture, sync and summary errors are part of the status snapshot it publishes.
  - No separate event stream.
  - How these failures are classified and presented is part of the status model in [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]]. This ticket only guarantees they reach the snapshot and state.
  - Per the invariants, messages must not contain credentials or provider response bodies.
- [ ] **Lock ownership on handover.** If a host starts while a CLI `run` holds `runner.lock`, does the host attach to that runner as an observer, or refuse to start and offer to take over?

## Acceptance criteria

<!-- GUIDE: Testable statements, each checkable by someone without this ticket's history. -->

## Implementation plan

<!-- GUIDE: Written by /implementation-plan — ordered steps, files, risks, verification,
     and the reason behind any order that matters. -->

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

<!-- GUIDE: Written by /test-plan — `- [ ]` manual checks only, excluding what the
     automated suites already assert. Keep `open_tests:` in sync as items are ticked. -->

## As-built notes

<!-- GUIDE: Where the implementation deviated from the plan, and why. Dated entries. -->

## Follow-ups

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[Dispatch Mesh milestone plan]] — 0.2.x engine stabilization; 0.3 tray application; Decisions to close during delivery
- [[Story - US00001 - Tray icon and status window host|US00001]] — consumer of this interface
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] — status model this interface transports
- Code: `src/main.rs` (`run`, `.mesh/runner.lock`, `show_status`), `src/engine.rs`, `src/lib.rs`; tests `tests/cli.rs`

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
