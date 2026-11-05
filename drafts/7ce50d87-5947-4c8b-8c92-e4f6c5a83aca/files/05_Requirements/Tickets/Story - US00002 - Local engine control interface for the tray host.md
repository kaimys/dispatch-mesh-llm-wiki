---
id: US00002
type: story
status: "Backlog"
priority: high
rank: 1000
version_target: v0.2.5
size:
assignee:
owner: "[[Kai]]"
open_questions: 6
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

- [ ] **Model.** Should the host embed the engine in-process, supervise a `run` child process, or talk to an independently started runner? Embedding is the simplest; supervising isolates a UI crash from capture.
- [ ] **Observation.** Can hosts poll the committed SQLite snapshot cheaply enough (and how often), or does the engine need to push changes (in-process channel, named pipe, file-change watch on the state DB)?
- [ ] **Waking and stopping a runner from another process.** Is a control channel needed in 0.3 at all (for example, CLI `session finish` waking the tray-owned runner to publish immediately), or is "the next sync cycle picks it up" acceptable?
- [ ] **Channel, if any.** Named pipe on Windows and a Unix socket elsewhere, or a localhost TCP port? Either way, how is access restricted to the current user?
- [ ] **Library API shape.** Should the crate expose a blocking API, an async (tokio) API, or both? The current run loop is tokio-based and `main.rs`-internal.
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
