---
id: US00005
type: story
status: "In progress"
priority: high
rank: 4000
version_target: v0.3.0
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings:
discussion:
updated: 2026-11-06
completed:
frozen:
---
Fill the status window's workspace view with what the engine already reports, and let the user finish a session there. Each recent session shows its stage track (saved on this device → shared as draft → finishing up → added to shared history), the files it touched, its author and any requested purpose; the header adds the last successful sync. **Finish session** becomes an action in the window and the tray menu, using the same engine operation as `dispatch-mesh session finish`. Recovery information shows what Mesh kept aside (saved copies, pending capture) and where. First of the remaining [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]] tickets after [[Story - US00001 - Tray icon and status window host|US00001]].

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a person working in a Mesh workspace, I want to see where each of my recent changes is on its way to shared history and finish my session from the tray, so that I know when my work is visible to others without using the CLI.

## Context

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]: work items "Show each workspace's capture, draft-sync, finalization, publication, and attention state" and "Add … optional **Finish session**, and access to recovery information"; first exit criterion "A user can inspect a workspace, finish a session, … through the desktop interface".
- Builds on [[Story - US00001 - Tray icon and status window host|US00001]]: the host (`tray/src/host.rs`), the Tauri-free view model (`tray/src/model.rs`) and the static frontend (`tray/ui/`). US00001 left per-stage detail, Finish session and recovery actions to later 0.3 tickets and shows only a hint line under Quit while a session is open.
- [[ADR-0010 — Report workspace status and session stages separately]] supplies the summary, `reason` and a `stage` per session; [[ADR-0009 — Embed the engine with a runner handle and published status]] publishes it through `RunnerSnapshot`.
- [[Claude Design briefing — desktop app]] §6.3 items 1–2 (header with last successful sync, activity with stage track) and §4 (the attention reason "Edits depend on a received draft …" has the action **Finish session**); frames in [[Dispatch Mesh desktop app.pdf]]. Vocabulary §5: "Shared as draft", "Added to shared history", "Saved copy".
- Engine: `session finish` queues finalization, which a running engine summarizes and publishes (README). Displaced files live in `.mesh/recovery/<id>/` with `record.json` (`path`, `created_at`).
- **Engine facts checked 2026-11-06 (refinement):**
  - `Status.sessions[]` (`src/status.rs`) carries `id`, `author` (name, email), `stage`, `closed`, `published`, `discarded`, `files`, `dependencies`. It has no times, no summary title and no requested purpose, although the engine stores them (`Session.created_at`, `last_edit`, `message.title`, `intents[].purpose` in `src/model.rs`).
  - No time of the last successful sync is recorded anywhere.
  - Authors are an `Identity` (name, email); nothing marks an agent. An agent is visible only through a recorded intent's purpose.
  - Saved copies are not in `Status`; they exist only as `.mesh/recovery/<id>/record.json`. `recovery_pending` means an apply journal is in progress, not that saved copies exist.
  - `Status.sessions` lists every session ever recorded; nothing prunes published ones.
  - `RunnerHandle` offers `status`, `stop` and `join` only. CLI `session finish` calls `Workspace::finish()` next to a running engine, so the host can do the same without a new handle method.
- Suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

## Scope

- **Engine (`Status`, amends ADR-0010):** `SessionStatus` gains the session's start and last-save times, the published summary title and the requested purposes; `Status` gains the time of the last successful sync (persisted on each successful sync) and the saved copies (original path, time, location). CLI `status` shows the same fields.
- Activity list per workspace: every unsettled session plus the 10 most recently settled ones, newest first, each with its stage track, files, author, times, requested purpose if recorded, and the summary title once published. Incoming sessions appear as the other author's, never as yours.
- Header: the last successful sync time next to the state.
- **Finish session** in the window while the workspace has an open session, and in the tray menu as one item per workspace with an open session; it calls `Workspace::finish()` in-process, like CLI `session finish`.
- The attention card's primary action **Finish session** for "Edits depend on a received draft …", and "Mesh retries automatically." under sync-failure reasons.
- Saved copies: original path and time, with **Show in folder**.

## Out of scope

- Conflict list and resolution ([[Story - US00006 - Conflict list and resolution in the status window|US00006]]), presence ([[Story - US00007 - Recent-save presence in the status window|US00007]]), notifications ([[Story - US00008 - Desktop notifications for conflicts and attention|US00008]]), pause/resume ([[Story - US00009 - Pause and resume synchronization|US00009]]).
- **Sign in again** (account sign-in arrives with guided setup in 0.5).
- Deleting, cleaning up or restoring saved copies from the window.
- **Retry now** for sync failures, a Finish-first prompt on Quit, and any "agent" marker on authors (answers above).

## Open questions

- [x] **Engine data.** Do `Status` and `RunnerSnapshot` already carry what the activity list needs: last successful sync time, published summary title, requested purpose, and whether the author is an agent? If not, which fields are added, and does that extend ADR-0010?
  **Answer (Kai, 2026-11-06):** extend `Status`, so CLI and tray read the same state. `SessionStatus` gains the session times, the published summary title and the requested purposes; `Status` gains the time of the last successful sync (newly persisted on each successful sync) and the list of saved copies. This amends [[ADR-0010 — Report workspace status and session stages separately|ADR-0010]]. No agent marker: the window shows the requested purpose and the author's identity, never "agent" (see the engine facts under Context).
- [x] **Activity length.** How many sessions does the list show, and how long do published ones stay visible?
  **Answer (Kai, 2026-11-06):** every session not yet settled (yours and collaborators'), plus the 10 most recently settled ones (published or discarded), newest first. Nothing is removed from engine state; the limit is the window's.
- [x] **Saved copies.** Show them only, or also offer **Restore** (copy back to the original path as an ordinary edit) or **Show in folder**?
  **Answer (Kai, 2026-11-06):** list each saved copy with its original path and time, and offer **Show in folder**, which opens Explorer on that copy. No **Restore** in this ticket: the window adds no new write path into the workspace, and restoring stays a manual copy (README).
- [x] **Finish session with several workspaces.** One tray-menu item per workspace with an open session, or one item that finishes all of them? Does Quit with an open session now offer Finish session first (the design's second Quit variant), or keep US00001's hint line?
  **Answer (Kai, 2026-11-06):** one tray-menu item per workspace with an open session, "Finish session — <workspace name>"; plain "Finish session" when only one workspace has an open session; no item when none has. Quit stays immediate with US00001's greyed hint line ("Your edits stay saved. The session continues next start."); no Finish-first prompt.
- [x] **Retry now.** Does the attention card's **Retry now** for "Remote rejected the update" and other sync failures belong here, or in a later ticket?
  **Answer (Kai, 2026-11-06):** no **Retry now** button. The engine retries a failed sync every 3 s without backoff (`src/runner.rs`), so the button would save at most 3 s. The attention card says "Mesh retries automatically." under sync-failure reasons. US00001's **Retry** for already-running and stopped workspaces stays. Revisit if sync gets a backoff or a paused state ([[Story - US00009 - Pause and resume synchronization|US00009]]).

## Acceptance criteria

1. With a session open in a workspace, the window's activity list shows it as "Saved on this device" with its files, the author's name, its start time and its last save.
2. As that session moves on, its stage track updates without reopening the window: Saved on this device → Shared as draft → Finishing up → Added to shared history. A workspace without a remote shows no "Shared as draft" step.
3. A published session shows its summary title. A session with a recorded intent shows its requested purpose.
4. A session received from another device shows that device's author, never the local user.
5. The list shows every unsettled session plus the 10 most recently settled ones (published or discarded), newest first.
6. The header shows the time of the last successful sync. It survives a restart of the host. A workspace that has never synced says so instead of showing a time.
7. **Finish session** appears in the window only while the workspace has an open session. Choosing it closes the session exactly as CLI `session finish` does, and the session moves on to Finishing up.
8. The tray menu shows "Finish session — <workspace name>" for each workspace with an open session, plain "Finish session" when only one has one, and no such item when none has. Quit is unchanged: no prompt, and US00001's hint line while a session is open.
9. With the attention reason "Edits depend on a received draft …", the attention card's primary action is **Finish session**.
10. Under a sync-failure reason, the attention card says "Mesh retries automatically." and offers no Retry now button.
11. Saved copies are listed with original path and time. **Show in folder** opens Explorer with that copy selected. The window offers no restore.
12. CLI `status` JSON contains the new session times, summary title, requested purposes, last successful sync and saved copies; every existing field keeps its name and meaning.
13. Every new control is reachable by keyboard and has a screen-reader name; stages and states are never shown by color alone.

## Implementation plan

### Ground truth that shapes the plan

Checked against the code on `main` at `29d3d58`:

- `Status` is built in one place, `Workspace::status_from` (`src/status.rs`), for both CLI `status` and the runner's `RunnerSnapshot` (ADR-0009, ADR-0010). Adding fields there reaches both, which is the point of Q1's answer.
- The engine already stores what the activity list needs except two things: `Session` has `created_at`, `last_edit`, `message.title` and `intents[].purpose` (`src/model.rs`), but **no time of a successful sync** and **no record of saved copies**. Displaced files are written to `.mesh/recovery/<id>/` (`src/engine.rs`, apply path); the in-state `apply_backups` map is cleared after each apply, so state forgets them.
- `Workspace::sync()` ends by clearing `attention` and `offline` and saving: that is the one success point. The runner calls it every 3 s.
- `State` is one JSON value in SQLite (`src/storage.rs`) with `#[serde(default)]` fields and no `deny_unknown_fields`; `src/codec.rs` maps every `State` field when converting blobs, so a new field must be added there too.
- `Workspace::finish()` is what CLI `session finish` calls; `Workspace::open` waits for the operation lock, so it works next to a running engine. `RunnerHandle` needs nothing new.
- Tray side: wording and rules live in `tray/src/model.rs` (pure, unit-tested); `tray/src/host.rs` owns workspaces; `tray/src/tray.rs` rebuilds the menu only when its content changes (today: the Quit hint); `tray/ui/app.js` renders the view model as text only.
- `published` and `discarded` carry no time, so "most recently settled" has no exact timestamp.

### Steps, in order

1. **Engine: new `Status` fields** (`src/status.rs`, `src/model.rs`, `src/codec.rs`, `src/engine.rs`).
   - `SessionStatus` gains `started_at` (`created_at`), `last_saved` (`last_edit`), `title` (`message.title` once published, else `null`) and `purposes` (the sessions' intent purposes, deduplicated, in recorded order). Existing fields keep name and meaning.
   - `State` gains `last_sync: Option<u64>`, set at the end of a successful `sync()` of a workspace with a remote (a workspace without one never sets it). **Coalesced:** it is only rewritten when the stored value is at least 60 s old, so an idle workspace does not write SQLite every 3 s. The window shows it at minute precision ("Last synced 2 min ago"), so nothing is lost. `Status` exposes it as `last_sync`.
   - `State` gains `saved_copies: Option<Vec<SavedCopy>>` (`id`, `path`, `created_at`), appended in the same transaction that records a displaced file. `None` means "never recorded": the first `Workspace::open` that sees `None` fills it once from `.mesh/recovery/*/record.json` and saves, so workspaces from earlier versions show their existing copies. `Status` exposes `saved_copies` (`id`, `path`, `created_at`) and never a file-system location; hosts resolve the location from the workspace root and the id.
   - Why first: the tray depends on these fields, and they are additive, so the CLI JSON stays compatible and older binaries ignore the new state fields (no `deny_unknown_fields`).
2. **ADR-0010 amendment** (wiki): a dated amendment listing the new `Status` fields, the 60 s coalescing of `last_sync`, and that hosts read workspace detail only through `Status`. Written with step 1 so the ADR never describes a different model than the code.
3. **Tray model** (`tray/src/model.rs`), all pure and unit-tested:
   - `WorkspaceView` gains `sessions` (the activity list), `last_sync`, `has_remote`, `open_session`, `saved_copies` and `actions` (the attention card's primary action, `finish_session` or none, and the "Mesh retries automatically." note).
   - Activity rule (Q2): every unsettled session, then the 10 most recently settled ones. Settled sessions are ordered by `last_saved`, newest first, because no settle time exists; this is the closest honest proxy and needs no new engine field.
   - Stage labels: "Saved on this device", "Shared as draft", "Finishing up", "Added to shared history", and "No changes kept" for discarded. A workspace without a remote never shows "Shared as draft" (the engine already skips that stage).
   - Author: the session author's name; "You" when the session's device is this device. Never "agent" (Q1).
   - Primary action: `finish_session` when the reason starts with "Edits depend on a received draft"; the retry note for sync-failure reasons (needs attention from a failed sync, not conflicts, capture or AI-summary reasons).
   - Menu items (Q4): one `("Finish session — <name>", workspace id)` per workspace with an open session, or the plain label when exactly one has one.
4. **Host** (`tray/src/host.rs`):
   - `finish(id)`: `Workspace::open(path)` then `finish()` on a worker thread, like the CLI; refused when the workspace is not running here (already running elsewhere or stopped), because then this host does not own it. The next snapshot shows the result, so no extra callback is needed.
   - `saved_copy_location(id, copy_id)`: returns `<root>/.mesh/recovery/<copy_id>/content` only if `copy_id` is listed in that workspace's latest `saved_copies` and is a plain UUID. The webview never sends a path.
5. **Tauri wiring** (`tray/src/main.rs`, `tray/src/tray.rs`, `tray/Cargo.toml`):
   - Commands `finish_session(id)` and `show_saved_copy(id, copy)`; the latter calls `tauri-plugin-opener`'s `reveal_item_in_dir` from Rust (no JS permission for the opener).
   - The tray menu gains the Finish-session items above Quit; it is rebuilt only when the list of items or the hint changes, as today.
6. **Window** (`tray/ui/app.js`, `app.css`): header line with the last sync ("Last synced 2 min ago", "Not synced yet", or "This device only" without a remote); activity list with a stage track per session, files, author, times, purposes and title; **Finish session** button; attention-card action and retry note; a "Saved copies" section with **Show in folder**. Relative times are recomputed every 30 s while the window is visible. All text via `textContent`; every control gets a `data-key` and an accessible name; stages carry a text label, not only a color.
7. **Docs**: README (status JSON fields; the tray's "Not yet" list loses session detail and Finish session), ADR-0010 amendment (step 2), and the ticket's as-built notes.

### Tests

- **Engine** (`tests/engine.rs`, `tests/cli.rs`): `status` JSON has the new session fields with correct values for an open, a published (title) and an intent-scoped (purpose) session; `last_sync` is `null` before the first sync, set after it, survives reopening, and is not rewritten within 60 s; a displaced file appears in `saved_copies` with its original path; a state without `saved_copies` is backfilled from `.mesh/recovery` on open; existing JSON fields unchanged (an assertion over the old key set).
- **Tray model** (`tray/src/model.rs` unit tests): the activity rule with 0, 5 and 15 settled sessions; stage labels for all five stages and for a workspace without a remote; "You" vs. a collaborator's name; primary action and retry note per reason; menu items for zero, one and several open sessions.
- **Host** (`tray/tests/host.rs`): `finish` on a running workspace closes its session and a later snapshot shows it finishing up; `finish` on an already-running workspace is refused; `saved_copy_location` rejects an unknown id and a path-like id.
- **Newly assertable:** the last-sync time and saved copies become testable engine state instead of folder scans.
- **Manual** (for `/test-plan`): rendering of the activity list and stage track, Finish session from window and menu, Show in folder opening Explorer with the copy selected, Narrator on the new controls.

### External changes and migration

- `status` JSON: additive fields only (`started_at`, `last_saved`, `title`, `purposes` per session; `last_sync`, `saved_copies`). Scripts parsing it keep working.
- Local state: two new optional fields; old states load with defaults, `saved_copies` is backfilled once. No format version change. A downgrade to 0.2.5 ignores the fields; a later upgrade backfills again (harmless, ids are stable).
- New dependency: `tauri-plugin-opener` (official Tauri plugin).

### Wiki pages this makes wrong

- [[ADR-0010 — Report workspace status and session stages separately]]: amended in step 2.
- README sections on `status` and on the tray preview: updated in step 7.
- [[Story - US00001 - Tray icon and status window host|US00001]]'s menu description stays true for its build; it is not edited (frozen).

### Risks and rollback

- **Write churn from `last_sync`.** Mitigated by the 60 s coalescing; a test pins it.
- **Order proxy.** Settled sessions are ordered by last save, not settle time. If that confuses users, a `settled_at` field is a small follow-up.
- **Finish while the engine is mid-sync** waits for the operation lock; it runs off the UI thread, so the window never freezes.
- **Path safety.** The webview passes only ids; the host resolves and checks them, so a compromised page cannot reveal arbitrary paths.
- **Rollback:** revert the commits. The new state fields are optional and ignored by older builds, so no data migration is needed either way.

### Decisions for review

1. `last_sync` is stored at most once a minute and shown at minute precision.
2. A workspace without a remote shows "This device only" instead of a sync time; its local finalization cycles do not count as syncing.
3. Settled sessions are ordered by last save.
4. Saved copies are recorded in state (with a one-time backfill) rather than read from the folder on every status.
5. "Show in folder" goes through `tauri-plugin-opener` from Rust, with ids only.

### Size

Order of magnitude: about three to four focused days, half engine and tests, half tray and window. Not a commitment.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

<!-- GUIDE: Written by /test-plan — `- [ ]` manual checks only, excluding what the
     automated suites already assert. Keep `open_tests:` in sync as items are ticked. -->

## As-built notes

### 2026-11-06 — development (branch `US00005-workspace-detail`)

Built as planned in three commits (engine `Status`, tray host and model, window and docs). Deviations and details the plan left open:

- **`has_remote` added to `Status`.** The plan gives `WorkspaceView` a `has_remote` but `Status` had no such field, and hosts read detail only through `Status`. Added as one more additive field; listed in the ADR-0010 amendment and README.
- **No "You" for own sessions.** The plan said "You" when the session's device is this device, but `SessionStatus` carries no device and AC 1 asks for the author's name. The window always shows the author's name, which also satisfies AC 4. No device field was added.
- **Reason matching uses engine constants.** `engine::SYNC_FAILURE` ("Synchronization waiting: ") and `engine::RECEIVED_DRAFT_DEPENDENCY` are now public, so the tray recognizes the two reasons without copying text. The retry note shows only for a `needs_attention` sync failure; offline keeps its own note.
- **Saved-copy bookkeeping on replay.** If a crash leaves an empty recovery folder before the move and the replay displaces the file again under a new id, the empty one is dropped from `saved_copies`. The one-time backfill skips folders whose name is not a UUID or whose `record.json` is unreadable. A read-only `inspect` of a workspace that has not recorded copies yet reads the folder, without saving.
- **`purposes` order within one second.** Intents carry whole seconds, so purposes recorded in the same second cannot keep their exact order; they are ordered by text then, not by their random ids (found as a flaky test).
- **New workspaces start with an empty `saved_copies`**, so the first open of a fresh workspace has nothing to backfill and leaves the state unchanged (caught by the legacy-migration test).
- **Finish session waits for the engine.** `Host::finish` retries `Workspace::open` for up to 30 s while the engine holds the operation lock (`open` itself gives up after 0.5 s). A failure from the window is shown under the state card; from the tray menu it is silent (the session simply stays open).
- **Show in folder** calls `tauri_plugin_opener::reveal_item_in_dir` directly: no plugin registration and no JS permission.
- **Window layout.** Saved copies sit in a collapsed "Saved copies · N" disclosure, newest first. The activity card's heading is the summary title once published, otherwise the files. The stage track marks steps ✓ / ● / ○ with hidden "(done / current step / not yet)" text. With the attention action showing, the activity section's own **Finish session** button is hidden so there is only one.
- **Observation, not changed:** every incoming change to an existing file displaces the old file into `.mesh/recovery`, so `saved_copies` grows with each received edit and is never cleaned up (README). Status JSON and the window list grow with it. A cleanup or retention ticket may be warranted.

**Verification (Windows, 2026-11-06):** `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` are green. The debug tray ran against two disposable workspaces with a local bare remote. The window showed the last sync, a received draft under its author (Alice), an own session with its requested purpose, a published session with its title, and a saved copy. **Finish session** in the window moved the open session to "Added to shared history" and removed the button. **Show in folder** opened Explorer in the copy's recovery folder. Not exercised live: the tray-menu items, the attention card's action, Narrator.

## Follow-ups

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]
- [[Dispatch Mesh requirements and architecture]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]]
- [[Story - US00001 - Tray icon and status window host|US00001]]

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
