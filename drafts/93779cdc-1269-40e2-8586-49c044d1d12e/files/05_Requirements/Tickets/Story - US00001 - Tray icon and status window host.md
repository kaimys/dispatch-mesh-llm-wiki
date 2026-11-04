---
id: US00001
type: story
status: "Backlog"
priority: high
rank: 1000
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 6
open_tests:
open_findings:
discussion:
updated: 2026-11-04
completed:
frozen:
---
Give Dispatch Mesh its first Windows desktop shell. A tray-resident host runs the existing engine for a workspace, shows a tray icon whose state changes meaningfully, and opens a status window. Closing that window leaves Mesh running; only **Quit** stops the host. This is step 1 of [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]. It is the frame that the later 0.3 steps (workspace state detail, pause/resume and Finish session, conflicts, presence, notifications) fill in.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a Mesh user on Windows, I want Mesh to live in the system tray with an icon that tells me its state and a status window I can open and close, so that synchronization runs quietly in the background without a console window, and I can see it is working and stop it deliberately.

## Context

- Milestone source: [[Dispatch Mesh milestone plan]] → *0.3 — tray application*, first work item: "Add a tray icon with meaningful state changes and a status window. Closing the window leaves Mesh running; **Quit** stops the host." The same milestone requires the tray to "use the same engine operations and state as the CLI" and to "prevent duplicate engine instances for a workspace".
- Requirements: [[Dispatch Mesh requirements and architecture]] calls for a small desktop interface that shows saved locally, waiting to sync, synced and needs attention. It must offer pause, resume, quit and recovery actions, and closing the status window must have defined behavior distinct from quitting synchronization. The tray icon indicates when conflicts need attention, and clicking it opens the status window.
- The milestone plan's *Decisions to close during delivery* lists "Tray UI toolkit, local control interface, pause semantics, and notification behavior" as needed **before implementing 0.3**. The toolkit and the control interface are prerequisites for this ticket (see Open questions).
- Code today (0.2.4 console preview):
  - `src/main.rs` implements the `run` command: watcher, capture loop and synchronization scheduling on a tokio runtime. It takes an exclusive `fs2` lock on `.mesh/runner.lock`, which already blocks a second runner per workspace.
  - `status` reads a committed SQLite snapshot without the operation lock, and reports `capture_error`, `local_changes_waiting`, `recovery_pending` and the unresolved conflict IDs. This is the natural data source for icon state.
  - Engine logic lives in `src/engine.rs`, reused by the CLI.
- Invariants that apply (from `AGENTS.md`): never discard uncaptured edits, and keep capture going while network operations or AI requests are pending. Quit must therefore behave like today's Ctrl+C, which captures saved changes and stops without waiting for a last network request.

## Scope

- A Windows tray host process that runs the engine's watch/capture/sync loop for a workspace, reusing the same code path as `dispatch-mesh run` rather than a parallel implementation.
- A tray icon with distinct states derived from engine status (at least: synced/idle, working (saved locally or waiting to sync), needs attention), updated as status changes, with a tooltip naming the state.
- A tray menu with **Open status window** and **Quit**. Left-click opens the status window.
- A status window shell that shows the workspace and its current overall state. Closing it hides the window and leaves the host and synchronization running.
- **Quit** stops the host the same way Ctrl+C stops `run` today: saved changes are captured, and in-flight state stays recoverable.
- The host refuses to start a second engine for a workspace that already has a runner, whether that runner is a CLI `run` or another tray host, and tells the user why.
- CLI commands (`status`, `sync`, `session finish`, intent commands) keep working against a workspace while the tray host runs it.

## Out of scope

Covered by later 0.3 tickets:

- per-stage workspace detail (capture, draft sync, finalization, publication, attention);
- pause/resume, **Finish session** and recovery actions;
- the conflict list and resolution UI;
- recent-save presence;
- desktop notifications.

Also out of scope:

- guided setup (0.5);
- bundled Git and installer (0.4);
- login startup (0.6);
- macOS/Linux (0.7/0.8).

The host works on a workspace already created with `dispatch-mesh init`.

## Open questions

- [ ] **UI toolkit.** Which Rust toolkit for the tray and status window (e.g. `tray-icon` + `tao`/`winit` with a webview, Tauri, egui/eframe, native Win32)? It must keep 0.7 macOS and 0.8 Linux viable. This is a milestone "decision to close" before implementing 0.3.
- [ ] **Local control interface.** Should the tray host embed the engine in-process, or supervise a separate `run` process and talk to it over a local IPC channel (named pipe or localhost socket)? Either way the CLI must keep working alongside it.
- [ ] **Single executable or separate binary.** Should this be a new `dispatch-mesh tray` subcommand in the existing binary, or a separate `dispatch-mesh-tray.exe`?
- [ ] **Workspaces per host.** One host per workspace (`--workspace`, like the CLI), or one host for several workspaces from the start? The milestone says "each workspace's … state", which suggests several. Choosing one now affects the icon-state aggregation and the window layout.
- [ ] **Icon state set.** What exact states does the icon show, and how do they map to the `status` fields? Is "paused" reserved now, although pause lands in a later ticket?
- [ ] **Quit with pending work.** When Quit is chosen while a session is open or a publication is in flight, should it quit immediately after capture (as Ctrl+C does), or offer to finish or publish first?

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

- [[Dispatch Mesh milestone plan]] — 0.3 tray application; Decisions to close during delivery
- [[Dispatch Mesh requirements and architecture]] — desktop interface, tray and status window
- Code: `src/main.rs` (`run`, `.mesh/runner.lock`, `status`), `src/engine.rs`

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
