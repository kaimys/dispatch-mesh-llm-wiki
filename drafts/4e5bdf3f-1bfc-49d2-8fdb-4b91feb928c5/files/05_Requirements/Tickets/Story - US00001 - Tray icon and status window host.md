---
id: US00001
type: story
status: Refinement
priority: high
rank: 1024
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings:
discussion:
updated: 2026-11-05
completed:
frozen:
---
Give Dispatch Mesh its first Windows desktop shell. A tray-resident host runs the existing engine for one or more workspaces, shows a tray icon whose state changes meaningfully, and opens a status window. Closing that window leaves Mesh running; only **Quit** stops the host. This is step 1 of [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]. It is the frame that the later 0.3 steps (workspace state detail, pause/resume and Finish session, conflicts, presence, notifications) fill in.

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
- The milestone plan's *Decisions to close during delivery* lists "Tray UI toolkit, local control interface, pause semantics, and notification behavior" as needed **before implementing 0.3**. The toolkit was settled on 2026-11-05 in this ticket: Tauri 2 (see Open questions). The control interface was settled on 2026-11-05 by [[ADR-0009 — Embed the engine with a runner handle and published status]] and is implemented under [[Story - US00002 - Local engine control interface for the tray host|US00002]].
- Code today (0.2.4 console preview):
  - `src/main.rs` implements the `run` command: watcher, capture loop and synchronization scheduling on a tokio runtime. It takes an exclusive `fs2` lock on `.mesh/runner.lock`, which already blocks a second runner per workspace.
  - `status` reads a committed SQLite snapshot without the operation lock, and reports `capture_error`, `local_changes_waiting`, `recovery_pending` and the unresolved conflict IDs. This is the natural data source for icon state.
  - Engine logic lives in `src/engine.rs`, reused by the CLI.
  - *Verified in refinement, 2026-11-04:* the watch/capture/sync loop (`watch()`) and the status derivation (`show_status()`) live in the **binary** (`src/main.rs`), not in the library (`src/lib.rs` exports `ai`, `codec`, `engine`, `git`, `model`, `storage`, `transport`). Reusing "the same code path as `run`" therefore means moving both into the library first. The loop reports state only through `println!`/`eprintln!`, and its only stop trigger is `tokio::signal::ctrl_c()`; a host needs state as data and a programmatic stop.
  - `status` already derives an aggregate `status` field with three values: `needs_attention` (attention set, capture error, or an unresolved conflict), `waiting_to_sync` (local changes not yet captured, or an unsettled session from this device), and `synced`. It does not separate the requirements' *saved locally* from *waiting to sync*, and has no *offline* state.
  - A failed background sync (other than `WorkspaceBusy`) writes `state.attention = "Synchronization waiting: …"`, cleared only by the next successful `sync()`. Under today's mapping, being offline therefore reads as `needs_attention`.
  - CLI commands already coexist with `run`: they take the short-lived `.mesh/lock` operation lock and `.mesh/network.lock`, and `status` reads a read-only SQLite snapshot (`Workspace::inspect`). No IPC exists today.
  - The 0.2.x milestone lists "Define the local engine/control interface that the tray host will use" as stabilization work. No such interface exists in code or docs; this ticket has to close it (see Open questions).
- **Governing decisions (2026-11-05).** The three bullets above describe 0.2.4 and are being replaced:
  - [[ADR-0009 — Embed the engine with a runner handle and published status]] (proposed; built under US00002): the tray embeds the engine through `Runner::start(root) -> RunnerHandle`, receives changed status snapshots on a watch channel, and stops it with `stop()`, which captures saved edits without waiting for the network. A held runner lock yields an explicit, retryable "already running" error.
  - [[ADR-0010 — Report workspace status and session stages separately]] (proposed; built under [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]]): the workspace summary becomes `needs_attention` > `offline` > `saved_locally` > `draft_uploaded` > `waiting_for_finalization` > `synced`, with a sanitized `reason` and per-session stages. Offline is no longer reported as `needs_attention`.
- Invariants that apply (from `AGENTS.md`): never discard uncaptured edits, and keep capture going while network operations or AI requests are pending. Quit must therefore behave like today's Ctrl+C, which captures saved changes and stops without waiting for a last network request.

## Scope

- A Windows tray host process, `dispatch-mesh-tray.exe` (Tauri 2), separate from the CLI program, that runs the engine's watch/capture/sync loop for one or more workspaces, one runner each, reusing the same code path as `dispatch-mesh run` rather than a parallel implementation.
- A tray icon with distinct states derived from engine status (at least: synced/idle, working (saved locally or waiting to sync), needs attention), updated as status changes, with a tooltip naming the state.
  *Settled 2026-11-05 (see Open questions → Icon state set):* four icons, Up to date, Working, Offline and Needs attention; the tooltip names the exact ADR-0010 state.
- With several workspaces, the tray shows the highest-precedence state (Needs attention > Offline > Working > Up to date), and the tooltip names the workspace and reason that set it.
- A tray menu with **Open status window** and **Quit**. Left-click opens the status window.
- A status window shell that shows each workspace's name, folder path and current overall state, with the `reason` for *needs attention* and *offline*. With several workspaces it lists them, most urgent first; with one it opens straight on it. Closing it hides the window and leaves the host and synchronization running; the first close says so ("Closing this window doesn't stop Mesh. It keeps syncing from the tray.").
- **Quit** stops the host the same way Ctrl+C stops `run` today: saved changes are captured, and in-flight state stays recoverable. It never prompts; with a session open, the menu hint says edits stay saved and the session continues next start.
- The host refuses to start a second engine for a workspace that already has a runner, whether that runner is a CLI `run` or another tray host, tells the user why and offers **Retry**. Its other workspaces run normally.
- CLI commands (`status`, `sync`, `session finish`, intent commands) keep working against a workspace while the tray host runs it.

## Out of scope

Covered by later 0.3 tickets:

- per-stage workspace detail (capture, draft sync, finalization, publication, attention);
- pause/resume, **Finish session** and recovery actions, including the action buttons the design puts in the status card (Open conflicts, Retry now, Sign in again);
- the conflict list and resolution UI;
- recent-save presence;
- desktop notifications.

Also out of scope:

- guided setup (0.5), including adding a workspace from the UI;
- bundled Git and installer (0.4);
- login startup (0.6);
- macOS/Linux (0.7/0.8).

The host works on a workspace already created with `dispatch-mesh init`.

## Open questions

- [x] **UI toolkit.** Which Rust toolkit for the tray and status window (e.g. `tray-icon` + `tao`/`winit` with a webview, Tauri, egui/eframe, native Win32)? It must keep 0.7 macOS and 0.8 Linux viable. This is a milestone "decision to close" before implementing 0.3.
  **Answer (Kai, 2026-11-05):** **Tauri 2.**
  - The Rust side embeds the engine through `RunnerHandle` (ADR-0009) on the same tokio runtime. The window is a small HTML/CSS frontend in the system webview (WebView2 on Windows) built from the token sheet in [[Dispatch Mesh desktop app.pdf]] (deliverable 6), as a compact utility window, not a web-app shell.
  - Why: the design carries over almost directly, text and side-by-side comparison are cheap to render, and the maintained plugins (tray, notifications, autostart, installer bundling, updater) cover planned work in 0.3 notifications, 0.4 packaging, 0.6 login startup and the 0.7/0.8 macOS and Linux ports.
  - Accepted costs: a JS/TS frontend toolchain beside Cargo, the WebView2 memory footprint, and WebKitGTK variance on Linux.
  - Considered: egui/eframe + `tray-icon` (runner-up: all Rust and small, but non-native text, hand-built comparison/toasts/autostart/installer, and hidden-window and Linux-tray friction), Slint (smaller ecosystem), native Win32 (rules out 0.7/0.8).
  - Before the implementation plan relies on it, a short spike confirms on Windows 11: tray icon with state changes, window hidden on close, embedded runner, a toast, and Narrator reading the icon and window labels. The implementation plan records the decision as an ADR and closes the milestone's "Tray UI toolkit" item.
- [x] **Local control interface.** Should the tray host embed the engine in-process, or supervise a separate `run` process and talk to it over a local IPC channel (named pipe or localhost socket)? Either way the CLI must keep working alongside it. Note: the CLI already coexists with a runner through file locks and the read-only snapshot, so in-process embedding needs no new IPC for this ticket; IPC would only be needed if a separate process must be controlled. This also closes the 0.2.x "control interface" item.
  **Answer (Kai, 2026-11-05, via US00002 refinement; recorded in [[ADR-0009 — Embed the engine with a runner handle and published status]]):**
  - The tray embeds the engine in-process through `RunnerHandle`, with no IPC in 0.3. The CLI keeps working through the existing on-disk locks.
  - If a CLI `run` or another host holds the workspace, the tray refuses to start a second engine, explains why, and offers Retry.
- [x] **Single executable or separate binary.** Should this be a new `dispatch-mesh tray` subcommand in the existing binary, or a separate `dispatch-mesh-tray.exe`? Windows constraint: a tray app should be built with `windows_subsystem = "windows"` so no console window appears, but that subsystem detaches the CLI from its console. One binary therefore needs a console-attach workaround; two binaries avoid it.
  **Answer (Kai, 2026-11-05):** a **separate `dispatch-mesh-tray.exe`**, built with `windows_subsystem = "windows"` from the same Cargo workspace and version as `dispatch-mesh.exe`, sharing the engine through the library. The CLI stays an ordinary console program, which agents rely on; no console-attach workaround. Login startup (0.6) registers the tray program, which also enforces 0.6's "one desktop host per user context". The installer (0.4) ships both.
- [x] **Workspaces per host.** One host per workspace (`--workspace`, like the CLI), or one host for several workspaces from the start? The milestone says "each workspace's … state", which suggests several. Choosing one now affects the icon-state aggregation and the window layout.
  **Answer (Kai, 2026-11-05, from the Claude Design result of [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]], deliverable 3, multi-workspace frame):**
  - One host serves **several workspaces**, each with its own embedded runner (ADR-0009).
  - The tray shows the highest-precedence state of any workspace: Needs attention > Offline > Working > Up to date. Its tooltip names that workspace and the reason, and sums up the rest (for example "Dispatch Mesh — Needs attention: Team Wiki, 2 unresolved conflicts · 2 other workspaces fine"). Left-click opens the window on that workspace.
  - With several workspaces, the window lists them sorted by who needs the user first. With one workspace, it opens straight on it and the list never shows.
  - Adding a workspace from the UI is guided setup (0.5). How the 0.3 host learns its workspace list (for example repeated `--workspace` arguments) is left to the implementation plan.
  - "Already running" is per workspace: a workspace held by a CLI `run` or another host shows the already-running view with **Retry**, and the host's other workspaces run normally.
- [x] **Icon state set.** What exact states does the icon show, and how do they map to the `status` fields? Is "paused" reserved now, although pause lands in a later ticket? Two sub-points found in the code: (a) `status` has three values and merges *saved locally* with *waiting to sync*; do the icon states split them? (b) an offline device currently becomes `needs_attention` via the sync-failure attention message; should offline be its own, non-alarming state?
  *Narrowed (2026-11-05):* [[ADR-0010 — Report workspace status and session stages separately]] defines the state set: `needs_attention`, `offline`, `saved_locally`, `draft_uploaded`, `waiting_for_finalization`, `synced`. That answers sub-points (a) and (b): they are split, and offline is its own state. Still open:
  - which of the six get distinct icons, as opposed to sharing a "working" icon;
  - whether "paused" is reserved. Pause is not in ADR-0010's model, so adding it later means extending that ADR.
  **Answer (Kai, 2026-11-05, from the Claude Design result of [[Claude Design briefing — desktop app]], deliverable 1 in [[Dispatch Mesh desktop app.pdf]]):**
  - Four icons in 0.3: **Up to date** (`synced`; the plain mark, no badge), **Working** (`saved_locally`, `draft_uploaded`, `waiting_for_finalization` share one icon), **Offline** (`offline`; dimmed mark with a slash, calm rather than red) and **Needs attention** (`needs_attention`; exclamation badge).
  - The badge *shape* carries the state, so it reads without color and on a monochrome macOS template image.
  - The tooltip, not the icon, names the exact state, as `Dispatch Mesh — <label>` with the reason for attention and offline, for example "Dispatch Mesh — Shared as draft (Team Wiki)" or "Dispatch Mesh — Needs attention: 2 unresolved conflicts". Screen readers get the tooltip text as the icon name.
  - **Paused** is reserved: a glyph slot exists in the design, but 0.3 ships no paused icon. The pause ticket adds it together with the ADR-0010 extension.
- [x] **Attention reason in the shell.** When the icon shows *needs attention*, does the status window shell show the reason (the `attention` message, capture error, or "N unresolved conflicts"), or only the state name until the later detail ticket? Per-stage detail is out of scope, but a bare "needs attention" with no reason is not actionable.
  *Narrowed (2026-11-05):* [[ADR-0010 — Report workspace status and session stages separately]] supplies a sanitized `reason` whenever the summary is `needs_attention` or `offline`, so showing it costs the shell one label. What remains is only the UI choice of whether the shell displays it.
  **Answer (Kai, 2026-11-05, from the Claude Design result of [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]], deliverable 3):** yes. The window's header card shows the state label and, for `needs_attention` and `offline`, the engine's `reason` in plain words. Offline adds the reassurance "Keep working — your changes upload when the connection returns." Only Needs attention gets color; offline is grey, the working states stay quiet. The primary actions the design puts in that card (Open conflicts, Retry now, Sign in again, Finish session) arrive with their later 0.3 tickets, so the 0.3 shell shows the reason without the action.
- [x] **Quit with pending work.** When Quit is chosen while a session is open or a publication is in flight, should it quit immediately after capture (as Ctrl+C does), or offer to finish or publish first?
  *Narrowed (2026-11-05):* [[ADR-0009 — Embed the engine with a runner handle and published status]] fixes what `RunnerHandle::stop()` does: it captures saved edits and returns without waiting for in-flight network or AI requests, and the captured work survives through the journals ([[ADR-0007 — Persist state transactionally with recovery journals]]). What remains is only whether Quit first *offers* "Finish session", which is a UI prompt layered on top of `stop()`.
  **Answer (Kai, 2026-11-05, from the Claude Design result of [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]], deliverable 2, Variant A — recommended):** Quit never prompts. It calls `stop()` and exits. With a session open, the menu item carries the hint "Your edits stay saved. The session continues next start." The design places **Finish session** directly above Quit so the choice is visible without a dialog; that item lands with the Finish session ticket. Variant B (a "Quit with a session open?" dialog) was rejected because it adds a step to every quit with unfinished work.

## Acceptance criteria

Draft. These are sharpened as the open questions are answered; they are not final.

- With the host running on an initialized workspace, no console window is visible and a Mesh icon is in the notification area.
- `dispatch-mesh.exe` still prints its output to the terminal it was started from, and its output can be piped, as in 0.2.x.
- Editing a file changes the icon from *synced* to a working state, and back to *synced* after the change synchronizes. The tooltip names the current state.
- Creating an unresolved conflict changes the icon to *needs attention*, and the tooltip names the reason (for example "2 unresolved conflicts").
- With the remote unreachable, the icon shows *offline* (not *needs attention*), the tooltip says changes are saved on this device, and saved edits are still captured.
- *Saved locally*, *shared as draft* and *finishing up* show the same working icon; the tooltip names which one applies.
- The icon states are distinguishable without color (badge shape), on light and dark taskbars.
- Left-clicking the icon, or choosing **Open status window**, shows a window with the workspace name, path and current state. In *needs attention* or *offline*, the window also shows the reason.
- With two workspaces, one with a conflict and one up to date, the icon shows *needs attention*, the tooltip names the conflicted workspace, and the window lists it first. With one workspace, the window shows it directly without a list.
- Closing the status window leaves the icon present; a file edited afterwards is still captured and synchronized. The first close tells the user Mesh keeps running.
- Choosing **Quit** removes the icon and ends the process without a prompt, even with a session open. An edit saved just before Quit is captured, and the next `run` or host start synchronizes it.
- Starting the host on a workspace that already has a `dispatch-mesh run` or another host shows a message saying Mesh is already running for that workspace, with **Retry**, and starts no second engine for it. After the other runner stops, Retry starts it. The host's other workspaces run normally meanwhile.
- While the host runs, `dispatch-mesh status`, `sync`, `session finish` and `intent begin/end` succeed against the same workspace.

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

### 2026-11-05 — Governing engine decisions

[[ADR-0009 — Embed the engine with a runner handle and published status]] and [[ADR-0010 — Report workspace status and session stages separately]] record the agreed designs supplied by US00002 and TASK00001. They govern host integration and status presentation but do not settle this ticket's remaining UI questions or complete those dependencies.

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[Dispatch Mesh milestone plan]] — 0.3 tray application; Decisions to close during delivery
- [[Dispatch Mesh requirements and architecture]] — desktop interface, tray and status window
- [[Claude Design briefing — desktop app]] and its results [[Dispatch Mesh desktop app.pdf]] / [[Dispatch Mesh desktop app.html]] — icon set, tray menu, status window, toasts and token sheet
- Code: `src/main.rs` (`run`, `.mesh/runner.lock`, `status`), `src/engine.rs`
- [[Story - US00002 - Local engine control interface for the tray host|US00002]] — decides the host↔engine interface (open question on the local control interface)
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] — status model the icon states derive from
- Discussion: no chat integration is configured; refinement questions and answers are recorded in this ticket.

## Dispatch runs

- 2026-11-05 09:10 — Start refinement (claude, done, 846 min)
    > I created `.claude/settings.local.json` with `"defaultMode": "bypassPermissions"`. New Claude sessions started in `dispatch-mesh` will now skip permission prompts, after you accept the one-time bypass warning. This session's mode stays as it is. Git ignores the file, but through your global ignore file (`~/.config/git/ignore`), not the repo's `.gitignore`. So it stays out of commits on this machin…

- 2026-11-05 09:09 — Start refinement (claude, done, 845 min)

- 2026-11-05 09:09 — Start refinement (claude, done, 845 min)
    > I created `.claude/settings.local.json` with `"defaultMode": "bypassPermissions"`. New Claude sessions started in `dispatch-mesh` will now skip permission prompts, after you accept the one-time bypass warning. This session's mode stays as it is. Git ignores the file, but through your global ignore file (`~/.config/git/ignore`), not the repo's `.gitignore`. So it stays out of commits on this machin…

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
