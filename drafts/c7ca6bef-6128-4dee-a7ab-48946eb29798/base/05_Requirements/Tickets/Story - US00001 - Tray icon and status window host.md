---
id: US00001
type: story
status: In progress
priority: high
rank: 1024
version_target: v0.3.0
size:
assignee: Kai
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
- *Code drift, 2026-11-05:* 0.2.5 shipped both decisions below. `src/runner.rs` (`acquire`, `run`, `Runner::start`, `RunnerHandle`, `AlreadyRunning`, `RunnerSnapshot`) and `src/status.rs` (`Summary`, `Stage`, `Status` with `reason`, `active_session`, `presence`) are now library modules; CLI `run` uses `runner::run`. The 0.2.4 bullets above are kept for history.
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

Final as of 2026-11-05 (refinement ended by Kai).

- With the host running on an initialized workspace, no console window is visible and a Mesh icon is in the notification area.
- `dispatch-mesh.exe` still prints its output to the terminal it was started from, and its output can be piped, as in 0.2.x.
- Editing a file changes the icon from *synced* to a working state, and back to *synced* after the change synchronizes. The tooltip names the current state, in the form `Dispatch Mesh — <state>`.
- Creating an unresolved conflict changes the icon to *needs attention*, and the tooltip names the reason (for example "2 unresolved conflicts").
- With the remote unreachable, the icon shows *offline* (not *needs attention*), the tooltip says changes are saved on this device, and saved edits are still captured.
- *Saved locally*, *shared as draft* and *finishing up* show the same working icon; the tooltip names which one applies.
- The icon states are distinguishable without color (badge shape), on light and dark taskbars.
- Narrator reads the tooltip text as the icon's name, and reads the window's workspace name, state and reason.
- Left-clicking the icon, or choosing **Open status window**, shows a window with the workspace name, path and current state. In *needs attention* or *offline*, the window also shows the reason.
- With two workspaces, one with a conflict and one up to date, the icon shows *needs attention*, the tooltip names the conflicted workspace, and the window lists it first. With one workspace, the window shows it directly without a list.
- Closing the status window leaves the icon present; a file edited afterwards is still captured and synchronized. The first close tells the user Mesh keeps running.
- The tray menu offers **Open status window** and **Quit**. With a session open, Quit carries the hint that edits stay saved and the session continues next start.
- Choosing **Quit** removes the icon and ends the process without a prompt, even with a session open. An edit saved just before Quit is captured, and the next `run` or host start synchronizes it.
- Starting the host on a workspace that already has a `dispatch-mesh run` or another host shows a message saying Mesh is already running for that workspace, with **Retry**, and starts no second engine for it. After the other runner stops, Retry starts it. The host's other workspaces run normally meanwhile.
- While the host runs, `dispatch-mesh status`, `sync`, `session finish` and `intent begin/end` succeed against the same workspace.

## Implementation plan

*Drafted 2026-11-05 against `main` at `7e13efd` (0.2.5). Signed off by Kai on 2026-11-05 by invoking `/develop US00001`; development on branch `us00001-tray-host` off `7e13efd`.*

### Ground truth that shapes the plan

- **The engine side is done.** `dispatch_mesh::runner` already gives a host everything this ticket needs: `Runner::start(root)` fails synchronously with `AlreadyRunning`, `status()` is a `watch::Receiver<Option<RunnerSnapshot>>` replaced only on change, `stop()` is idempotent, and dropping a handle stops and joins it (ADR-0009). `RunnerSnapshot.status` is ADR-0010's `Status`, with `status` (six-value `Summary`), a sanitized `reason` for `needs_attention`/`offline`, and `active_session`. **No library change is planned.** If one turns up, it goes into the library with tests, not into the tray crate.
- One `RunnerHandle` per workspace, several per process, works today: the runner lock is a per-file-handle `fs2` lock, and `a_held_runner_lock_is_refused_until_it_is_released` in `tests/runner.rs` covers refusal and retry.
- Tauri 2's async runtime is Tokio, so the host can `await rx.changed()` per workspace: no polling and no `on_change` callback (ADR-0009 leaves that callback optional).
- `report_failure` and the "resumed" messages in `src/runner.rs` still `println!`/`eprintln!`. In a GUI-subsystem process, Rust's std on Windows discards writes to a missing std handle instead of failing. The spike confirms this; logging is not in scope.
- The project is a single package today (`Cargo.toml`, no `[workspace]`). CI runs the gates on `windows-latest` and `ubuntu-latest`.
- Dev machine: Rust 1.98, Node 24 (not needed by this plan), WebView2 runtime present, no `tauri-cli`.
- Design source: [[Dispatch Mesh desktop app.pdf]] deliverables 1–3 and 6 (icons, menu Variant A, status window frames, token sheet). The HTML version is a self-unpacking bundle; icon SVGs and tokens are extracted from it once (step 4).

### Steps, in order

1. **Spike, on a throwaway branch (gate for everything below).** A minimal Tauri 2 app that embeds `Runner::start` on one workspace and shows: a tray icon whose image and tooltip change with the snapshot; left-click opens a window and right-click a menu; the window hides on close while capture continues; **Quit** stops, joins and exits; one toast; Narrator reads the tray tooltip and an `aria-live` state label in the window; `println!` from the runner does no harm in a `windows_subsystem = "windows"` build. Also time a clean `cargo build` and the Linux CI build with the Tauri system packages.
   > ⚙️ Plan update 2026-11-05 (development): an agent can't operate Narrator or judge how the icon and toast look, and the Linux build runs only in CI. The spike therefore checks the lifecycle automatically; the visual, Narrator and toast checks move to the app run before `/code-review` (step 8), and the Linux build to the first CI run of step 2. Either failing still reopens the toolkit question. Results under As-built notes.
   *Why first:* the toolkit answer made this test a condition. **If any must-have fails, stop** and reopen the toolkit question in this ticket (runner-up: egui/eframe + `tray-icon`) rather than working around it. The spike code is not merged.
2. **Cargo workspace and an empty tray crate.**
   - Root `Cargo.toml` gains `[workspace] members = [".", "tray"]`, `default-members = [".", "tray"]`, and `[workspace.package] version`; both packages use `version.workspace = true`, so one bump in `/release` versions both programs. `Cargo.lock` stays shared.
   - New `tray/` package `dispatch-mesh-tray` (binary `dispatch-mesh-tray.exe`): `Cargo.toml` (depends on `dispatch-mesh` by path, `tauri` 2, `serde`), `build.rs` (`tauri_build::build()`), `tauri.conf.json` (no `version`, so it follows Cargo; no bundle config, which is 0.4; strict CSP, no remote URLs), `capabilities/default.json` (main window: core events only), `src/main.rs` with `#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]`.
   - The frontend is plain static files in `tray/ui/` (HTML, CSS, a small vanilla JS file) served from `frontendDist` with `withGlobalTauri`. **No npm, no bundler, no `tauri-cli`** in 0.3; `cargo build` embeds the assets. This removes most of the "JS toolchain" cost accepted with Tauri.
   - CI: on Ubuntu, install the Tauri 2 Linux prerequisites (WebKitGTK 4.1, AppIndicator, librsvg, xdo; exact list from the spike) before the gates.
   *Why second, and why `default-members`:* the unchanged gate commands then cover both packages on both CI platforms. The Linux build keeps 0.8 honest from the first commit, and the gates in `dispatch/invariants.md` and `AGENTS.md` need no rewording. The cost is that building anything on Linux now needs the GTK packages.
3. **Pure presentation model** (`tray/src/model.rs`, no Tauri types).
   - `WorkspaceState`: `Running(RunnerSnapshot)`, `Starting`, `AlreadyRunning`, `Stopped(reason)` (the loop ended with an error, or the folder is not a Mesh workspace).
   - `label(Summary)`: user labels from the briefing (Up to date, Saved on this device, Shared as draft, Finishing up, Offline — changes saved on this device, Needs attention).
   - `icon(Summary)`: `synced → UpToDate`, `saved_locally | draft_uploaded | waiting_for_finalization → Working`, `offline → Offline`, `needs_attention → Attention`.
   - `aggregate(&[Workspace])`: highest precedence wins, Needs attention > Offline > Working > Up to date. `Stopped` counts as needs attention. `AlreadyRunning` does not count, because another runner is syncing that folder; if *no* workspace is running, the icon shows needs attention ("Mesh isn't syncing any workspace").
   - `tooltip(...)`: `Dispatch Mesh — <label>`, with `: <reason>` for attention and offline; with several workspaces, it names the deciding workspace and sums up the rest ("· 2 other workspaces fine"). Windows caps tooltips at 127 characters, so it truncates the summary first, then the reason, never the state.
   - `order(...)`: the window list, most urgent first, then by name.
   - The view model sent to the window is built here, so the tray and window share every word.
   *Why before the host:* it is all the logic the AC tests depend on, and it is unit-testable without a window.
4. **Host core** (`tray/src/host.rs`, no Tauri types; the Tauri side passes callbacks in).
   - Workspaces come from repeated `--workspace <path>` arguments; display name = folder name. Started with none, the window shows a "No workspace yet" state with the `dispatch-mesh init` command and how to start the tray with `--workspace`. Adding workspaces from the UI is 0.5, and no workspace list is persisted in 0.3, so there is nothing to migrate later.
   - `start(path)` → `Runner::start`; `AlreadyRunning` → `WorkspaceState::AlreadyRunning`; any other error → `Stopped(sanitized message)`. `retry(id)` repeats it.
   - Per running workspace, one task awaits `changed()` and pushes the new `WorkspaceState` to the host callback. A closed channel means the loop ended; the task joins the handle and reports `Stopped` with the error, which offers Retry.
   - `quit()`: hide the tray icon at once, `stop()` all handles (so they shut down in parallel), join them on a background thread, then exit. Worst case is about 10 s per ADR-0009's shutdown lock wait, with the icon already gone.
   *Why separate from Tauri:* the Retry, AlreadyRunning and Quit behavior is then integration-testable with real workspaces (see Tests).
5. **Tray** (`tray/src/tray.rs`).
   - Icons: four states × light/dark taskbar as multi-size `.ico` (16, 20, 24, 32 px for 100–200 % scaling) in `tray/icons/`, rasterized once from the design's SVGs. Paused is not shipped. The design asks for hand hinting at 16 px, which is a design task for Kai and not a blocker for development; the rasterized set is used until it lands.
   - Taskbar theme: read `HKCU\…\Themes\Personalize\SystemUsesLightTheme` (the taskbar's setting, not the apps' setting) at start, on each status change and on a 5 s timer, and swap the icon set on change. Off Windows, use the light set for now (0.7/0.8 add template images).
   - Left-click opens and focuses the window; the right-click menu is **Open status window**, **Quit Dispatch Mesh**, and, only while any workspace has an `active_session`, a disabled hint line under Quit: "Your edits stay saved. The session continues next start." No Finish session or Pause items in 0.3.
   - The tooltip string from step 3 is also the icon's accessible name.
6. **Status window** (`tray/ui/`, `tray/src/main.rs` commands).
   - Compact fixed-width utility window, about 440 px, built from the token sheet (CSS variables for light and dark via `prefers-color-scheme`). One workspace: opens straight on it. Several: the list, most urgent first, with a detail view per workspace.
   - Header card per workspace: name, folder path, state label, and for attention and offline the `reason` (offline adds "Keep working — your changes upload when the connection returns."). Only attention gets color, offline is grey. No action buttons except Retry, which belongs to already-running and stopped states. No activity list, conflicts, presence or last-synced time; those are later 0.3 tickets.
   - Already-running view per the design ("Mesh is already running for this workspace … Retry").
   - Commands: `workspaces()` for the initial render, `retry(id)`; the host emits a `workspaces` event on every change. The JS only renders the view model.
   - Accessibility: the state label is an `aria-live="polite"` region; all controls are reachable by keyboard; state is never shown by color alone (glyph + text).
   - Close: `CloseRequested` → `prevent_close()` + hide. On the **first** close ever, one toast: "Dispatch Mesh is still running. It keeps syncing from the tray." The "seen" flag is stored in `tray-settings.json` in the app config directory. This is the only toast in 0.3; the notification rules are the notifications ticket's.
   - `RunEvent::ExitRequested` without an exit code is prevented, so hiding the last window never ends the process.
7. **Docs.**
   - `README.md`: a "Tray (preview)" section with how to start `dispatch-mesh-tray --workspace <folder>`, what it shows, and what it doesn't do yet.
   - `AGENTS.md`: the product summary says tray controls are future work; update to "a tray host with status (0.3, in progress)", keeping the not-yet-built parts marked.
   - The ADRs and wiki updates listed below.
8. **Gates:** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test`, now covering both packages through `default-members`, green on Windows and Linux CI.

### Tests

- **Unit, `tray/src/model.rs`:** label and icon for all six summaries; aggregate precedence for every pair, including `Stopped`, `AlreadyRunning` and "none running"; tooltip format for one and several workspaces, with the 127-character truncation keeping the state; list ordering. These pin AC 3–7 and the multi-workspace AC at the level that decides them.
- **Integration, `tray/tests/host.rs`** (real temp workspaces via `Workspace::init` without a remote, no window): two workspaces start and both publish; a workspace whose runner lock is held by `runner::acquire` reports `AlreadyRunning` while the other runs, and `retry` succeeds after the lease drops; an edit saved right before `quit()` is captured when `quit()` returns; a runner whose loop ends with an error is reported as `Stopped` with Retry.
- **Already covered, not duplicated:** CLI commands against a workspace with a running handle, final capture on stop, snapshot publication and error clearing (`tests/runner.rs`, `tests/cli.rs`).
- **Newly assertable:** host-level multi-workspace behavior and the shared wording, which had no code before.
- **Manual (for `/test-plan`):** no console window; icon and tooltip on light and dark taskbars at 100 % and 150 % scaling; left/right click; hide on close and the first-close toast; Quit with a session open (hint shown, no prompt); offline by unplugging the network; Narrator; CLI `dispatch-mesh.exe` output and piping unchanged while the tray runs.

### External changes and migration

- None to storage, transport or the on-disk format. The tray only reads published snapshots and uses existing locks.
- New per-user file `tray-settings.json` (first-close flag only). Losing it repeats one toast; no migration needed.
- The Tauri `identifier` fixes the app config and data directories, and later the installer and login-startup registration. It must be chosen once (see Decisions for review).
- CI gains the Linux system packages and a longer build.

### Cross-ticket effects

- **Later 0.3 tickets** (per-stage detail, Finish session/pause, conflicts, presence, notifications) build on `host.rs`, the view model and `tray/ui/`. The notifications ticket owns the toast rules; this ticket's single first-close toast should move under them.
- **0.4 packaging** adds the Tauri bundler (and `tauri-cli` or its build API), the WebView2 bootstrapper, and ships both executables.
- **0.5 guided setup** replaces `--workspace` arguments with a persisted workspace list.
- **0.6 login startup** registers `dispatch-mesh-tray.exe` and adds the one-host-per-user rule. 0.3 deliberately has no single-instance check, because the AC expects a second host to report "already running" per workspace.

### Wiki pages this makes wrong

| Page | Change | Who |
|---|---|---|
| [[Dispatch Mesh milestone plan]] → *Decisions to close during delivery* | Mark "Tray UI toolkit" closed by the toolkit ADR | this workflow, at ADR extraction |
| ADR index in [[index]] | Add the new ADRs | this workflow, at ADR extraction |
| [[Claude Design briefing — desktop app]] §7 "Toolkit not chosen yet" | Note the decision; the brief is a draft source, so add a dated note instead of rewriting | `/develop`, docs step |
| `AGENTS.md`, `README.md` | Step 7 | `/develop` |
| Release note for 0.3 | Mentions the second executable | `/release` |

### Risks and rollback

- **Spike failure:** handled by the gate in step 1; the plan stops and the toolkit question reopens.
- **Taskbar theme detection** is registry-based and polled; a missed change shows a low-contrast icon until the next 5 s tick. Acceptable for 0.3.
- **Tooltip length:** multi-workspace tooltips get truncated; tested.
- **WebView2 missing** on some Windows 10 machines: Windows 11 ships it; the 0.4 installer bootstraps it. 0.3 shows Windows' own missing-runtime error.
- **CI time and Linux dependencies grow.** If the Linux Tauri build becomes a burden, the fallback is a Linux-only `cargo check -p dispatch-mesh-tray` instead of the full build, not dropping Linux.
- **Process kill** (Task Manager, crash) skips the final capture; captured work survives through ADR-0007 journals, as ADR-0009 already accepts. Logoff and shutdown handling is 0.6.
- **Rollback:** the tray is additive. Reverting the tray commits and the workspace change restores the single-package CLI; no data migration in either direction.

### Decisions for review

Approved by Kai on 2026-11-05 with `/develop US00001`; the identifier was chosen explicitly.

1. **Tauri identifier: `de.eightnine.mesh`** (Kai, 2026-11-05). Durable: it names the app directories and later registrations. Chosen short so that it survives a rename of "Dispatch".
2. **Already-running workspaces don't count** toward the tray icon; if nothing runs at all, the icon shows needs attention.
3. **Workspaces from `--workspace` arguments only**, nothing persisted until 0.5.
4. **First-close notice as a single toast**, not an in-window banner.
5. **`default-members` covers both packages**, so Linux builds need the GTK packages.

### Size

Roughly 7–9 focused days: spike 1, workspace and CI 0.5, model 1, host 1.5, tray and icons 1.5, window 2, docs and gates 0.5. Order-of-magnitude estimate, not a commitment; the spike may move it either way.

### ADRs

Extracted on 2026-11-05 at sign-off, both `proposed`: [[ADR-0011 — Build the desktop host with Tauri 2]] and [[ADR-0012 — Ship the desktop host as a separate executable serving several workspaces]]. Applying ADR-0009 and ADR-0010 needs no new ADR.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

<!-- GUIDE: Written by /test-plan — `- [ ]` manual checks only, excluding what the
     automated suites already assert. Keep `open_tests:` in sync as items are ticked. -->

## As-built notes

### 2026-11-05 — Spike (step 1)

Throwaway Tauri 2.12 app in a scratch folder (not in the repo), embedding `Runner::start` on a local workspace, driven by an autopilot and a log file. Results on Windows 11, WebView2 156:

- **Passed:** tray icon and tooltip updates from the status channel (`set_icon`/`set_tooltip` OK; Synced → SavedLocally within about 1 s of an edit); the window loads the static frontend under a strict CSP with `withGlobalTauri`, receives `status` events and calls commands; `CloseRequested` → `prevent_close` + hide keeps the process and capture running (edit captured with the window hidden); `ExitRequested` without a code is prevented; Quit hides the icon, stops and joins the runner (57 ms with nothing in flight) and exits 0; an edit saved 300 ms before Quit was captured; `println!`/`eprintln!` in a GUI-subsystem build (PE subsystem 2) do not panic; the notification plugin's `show()` returns Ok.
- **Measured:** clean release build of the spike 3 min 50 s.
- **Not checkable by an agent, moved:** Narrator reading the tooltip and the window's live state, the icon's look on light and dark taskbars, whether the first-close toast is actually displayed for an unpackaged app. These go to the app run before `/code-review`. The Linux build goes to the first CI run.
- WebView2 logs `Failed to unregister class Chrome_WidgetWin_0` to stderr at exit; harmless and invisible in a GUI build.

No must-have failed, so Tauri stays.

### 2026-11-05 — Build (steps 2–8)

Commits on `us00001-tray-host`: `bf7a68f` (tray package, workspace, CI), `091180e` (README, AGENTS.md). Built as planned, with these differences:

- **Window shown at start.** The window opens when the tray starts, so a user who launches the program sees something besides a possibly hidden tray icon. Login startup (0.6) can add a start-hidden option.
- **Debug builds keep a console.** `windows_subsystem = "windows"` applies to release builds only (`cfg_attr(not(debug_assertions))`), so `cargo run` still shows panics during development.
- **Not-a-workspace message.** A folder without `.mesh/config.json` is checked before starting and reported as "This folder is not a Mesh workspace yet. Set it up with dispatch-mesh init, then retry." instead of the raw file error. The first draft repeated the path and made the window scroll sideways; fixed before commit.
- **Loop ending on its own: not tested.** `host.rs` reports an engine whose loop ends without Quit as *Not syncing* with Retry, but the runner offers no way to make its loop fail on demand (capture and sync errors are reported, not fatal), so `tray/tests/host.rs` covers start failures instead. No library change was made to force it.
- **Same folder twice** in `--workspace` is one workspace, not a refused second engine (tested).
- **Icons** are supersampled renders from `scripts/tray_icons.py`, not yet hand-hinted at 16 px as the design asks. Rendered at 16, 20, 24 and 32 px and chosen by the primary monitor's scale factor.
- **Fonts:** the window uses Segoe UI and Cascadia Mono instead of the design's IBM Plex, because the CSP allows no remote fonts and bundling Plex was not planned.
- **Gates:** `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` green on Windows for both packages (CLI/engine 79, tray 11 unit + 6 host tests). Linux CI not yet run: the branch has not been pushed.

**App run, 2026-11-05 (release build, Windows 11 at 150 %):** checked by the agent with screenshots and UI Automation:

- three workspaces, one held by a CLI `run`: list ordered held → saved on this device → up to date, with the design's glyphs;
- the held workspace's detail shows the already-running text and Retry; after the CLI `run` stopped, Retry started it (*Up to date*);
- UI Automation exposes each row as a named button and the live region as "Held: Mesh is already running for this workspace. Research notes: Saved on this device. Team Wiki: Up to date", which is what Narrator reads;
- closing the window hid it, the process kept running, and `tray-settings.json` recorded the first close;
- an edit made with the window hidden was captured, and `status`, `intent begin/end`, `session finish` and `sync` all succeeded against the tray-owned workspace, printing to the console as before;
- a plain folder shows the attention card with its reason and Retry.

**Still to check by a person** (the agent can't see the notification area or toasts): the icon and tooltip on light and dark taskbars, left and right click, the Quit hint with an open session, Quit from the menu, whether the first-close toast is displayed for an unpackaged build, and Narrator itself. These belong in `/test-plan`.

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
- [[ADR-0011 — Build the desktop host with Tauri 2]] and [[ADR-0012 — Ship the desktop host as a separate executable serving several workspaces]] — decisions extracted from this ticket
- Discussion: no chat integration is configured; refinement questions and answers are recorded in this ticket.

## Dispatch runs

- 2026-11-05 09:10 — Start refinement (claude, done, 846 min)
    > I created `.claude/settings.local.json` with `"defaultMode": "bypassPermissions"`. New Claude sessions started in `dispatch-mesh` will now skip permission prompts, after you accept the one-time bypass warning. This session's mode stays as it is. Git ignores the file, but through your global ignore file (`~/.config/git/ignore`), not the repo's `.gitignore`. So it stays out of commits on this machin…

- 2026-11-05 09:09 — Start refinement (claude, done, 845 min)

- 2026-11-05 09:09 — Start refinement (claude, done, 845 min)
    > I created `.claude/settings.local.json` with `"defaultMode": "bypassPermissions"`. New Claude sessions started in `dispatch-mesh` will now skip permission prompts, after you accept the one-time bypass warning. This session's mode stays as it is. Git ignores the file, but through your global ignore file (`~/.config/git/ignore`), not the repo's `.gitignore`. So it stays out of commits on this machin…

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
