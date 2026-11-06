---
id: US00005
type: story
status: Done
priority: high
rank: -1024
version_target: v0.3.0
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests: 0
open_findings: 0
discussion:
updated: 2026-11-06
completed: 2026-11-06
frozen: 2026-11-06
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
- Activity list per workspace: every unsettled session, then the 10 settled ones with the latest last save, each group newest first, each with its stage track, files, author, times, requested purpose if recorded, and the summary title once published. Incoming sessions appear as the other author's, never as yours.
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
5. The list shows every unsettled session first, then the 10 settled sessions (published or discarded) with the latest last save; each group is ordered by last save, newest first. Settling records no time, so a long-open session that settles late may fall outside the ten. *(Amended 2026-11-06, Kai, resolving code-review finding F2: the planned order-proxy policy is the contract.)*
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

Manual checks for the workspace detail, Finish session and saved copies, on branch `US00005-workspace-detail` at `819d9b0` (Windows). Only what the suites and the pre-plan run (as-built notes, 2026-11-06) do not already cover: the tray menu, keyboard and the dark theme.

**Setup (once, PowerShell, repository root):** build with `cargo build`, then create two disposable workspaces on a local remote. `A` ends up with two published sessions (Alice's and Bob's) and one saved copy, and neither workspace has an open session:

```powershell
$t = "$env:TEMP\mesh-us5"; $m = ".\target\debug\dispatch-mesh.exe"
New-Item -ItemType Directory "$t\A", "$t\B" | Out-Null; git init -q --bare "$t\remote.git"
& $m --workspace "$t\A" init --name Alice --email alice@example.test --remote "$t\remote.git"
& $m --workspace "$t\B" init --name Bob --email bob@example.test --remote "$t\remote.git"
Set-Content "$t\A\note.md" "hello"; & $m --workspace "$t\A" session finish; & $m --workspace "$t\A" sync; & $m --workspace "$t\A" sync
& $m --workspace "$t\B" sync; Set-Content "$t\B\note.md" "changed by Bob"; & $m --workspace "$t\B" session finish; & $m --workspace "$t\B" sync; & $m --workspace "$t\B" sync; & $m --workspace "$t\A" sync
.\target\debug\dispatch-mesh-tray.exe --workspace "$t\A" --workspace "$t\B"
```

**Cleanup:** Quit the tray from its menu, then `Remove-Item $t -Recurse -Force`.

> 2026-11-06: Setup corrected before testing. Bob's change is now finished and published; the original recipe left B with an open session, which contradicted check 1's starting state.
>
> 2026-11-06: The Narrator check was removed at Kai's request before testing began (`open_tests` 4 → 3). The screen-reader part of AC 13 therefore has no manual verification. It rests on the code: every new control has an `aria-label` or visible text, and the stage-track steps carry hidden state text.

- [x] **Kai:** *Tray menu (AC 8).* With no edits, right-click the tray icon: no "Finish session" item, and **Quit Dispatch Mesh** has no hint line beneath it. Save any change to `A\note.md`; within about 5 s the menu shows one plain **Finish session** item above Quit, and the greyed hint "Your edits stay saved. The session continues next start." Save a change in `B` too; the menu now shows **Finish session — A** and **Finish session — B** instead. Choose **Finish session — B**: in the window, B's session moves on to Finishing up and then Added to shared history, and A's session is still open (A still offers **Finish session**). Choosing **Quit Dispatch Mesh** while A's session is open closes the tray at once, with no prompt.
- [x] **Kai:** *Keyboard (AC 13).* Open the window and select workspace A (save a change in A first, so it has an open session). Using only Tab, Shift+Tab, Enter and Space: **‹ All workspaces**, **Finish session**, the **Saved copies · 1** disclosure and **Show in folder** are each reachable in visual order, each with a visible focus outline. Space on the disclosure opens and closes it. Enter on **Finish session** finishes the session. Wait 30 s with focus on a control: focus stays on that control when the window refreshes its relative times.
- [x] **Kai:** *Dark theme and layout.* Switch Windows to dark mode (Settings → Personalization → Colors → Dark) and reopen the window on A. The header's "Last synced …", the activity cards, the stage-track marks (✓ ● ○), an attention card if one shows, and the saved-copies list are all legible with no element lost against its background. At the window's default size there is no horizontal scrollbar, and the long folder path in the header wraps instead of overflowing. Switch back afterwards.

**Results 2026-11-06 (Kai, guided session, debug build of `819d9b0`, Windows 11):** all three checks passed.
- *Tray menu:* no item with no open session. One plain **Finish session** with the hint line once A had a session; **Finish session — A / — B** with both. Finishing B moved B's session to Added to shared history and left A open. Quit with A's session open closed the test tray at once with no prompt. Afterwards CLI `status` showed A's session still open and its last edit captured (`local_changes_waiting: false`); the wiki tray kept running.
- *Keyboard:* every new control was reachable in order with a visible outline. The disclosure toggled with Space, and **Show in folder** opened Explorer with `content` selected. Focus survived the 30 s refresh, and Enter on **Finish session** finished the session. The transient "Finishing…" label was too brief to read, because the finish completes in a fraction of a second with no other operation holding the workspace. Expected, not a defect.
- *Dark theme and layout:* all new elements legible in dark mode, no horizontal scrollbar, and the folder path wraps.

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

### 2026-11-06 — review blocked at 4133e9c

Independent code review found two blockers: F1 (AC 11: interrupted displacement can list nonexistent saved-copy content) and F2 (AC 5: acceptance wording disagrees with the planned/built activity selection and ordering). Keep `In progress`; do not run `/test-plan` until both are resolved. The fix-round plan is recorded in Code review below. No implementation or contract text changed in this session.

### 2026-11-06 — code-review fix round (F1, F2)

- **F1 (phantom saved copy) fixed.** Every replay of the apply journal now first drops displacement bookkeeping whose content was never moved, from both `apply_backups` and `saved_copies`, whichever path the replay then takes (newer user save, file already written, re-displacement). This replaces the special case that only the re-displacement branch handled. The one-time backfill skips recovery folders without `content`. No recovery file is deleted: the empty folder and its `record.json` stay on disk. Regression tests in `tests/engine.rs`: crash before the move then a newer save, crash before the move with the file already written, crash before the move then a replay, crash after the move (real copy kept), and a legacy record without content. Run against the reviewed engine (`4133e9c`), the newer-save, already-written and legacy cases fail; all pass on the fix.
- **F2 (AC 5 vs. plan) resolved by contract.** Kai chose the planned policy (2026-11-06): unsettled sessions first, then the 10 settled ones with the latest last save, each group newest first. AC 5, the Scope line and README now say this; no code change. Added `activity_keeps_open_sessions_first_and_ranks_settled_ones_by_last_save` for the interleaved open/published and settled-late cases.
- `open_findings` cleared to empty for the review's re-check.

### 2026-11-06 — test-plan preparation: AC 9 defect found and fixed

- **Found while verifying AC 9 in the running tray** (`47f810e`, two disposable workspaces on a local bare remote). With "Edits depend on a received draft …" raised, the window showed "Local capture waiting: Dependent edits preserved; finish the current session …" with no primary **Finish session**. The runner reported capture's refusal as a capture fault, and a capture fault outranks `attention` in the summary. CLI `sync` had the same effect another way: `record_sync_failure` replaced the engine's reason with "Synchronization waiting: …", which would also have shown "Mesh retries automatically." where retrying cannot help. The cause predates US00005 (TASK00001/US00002 failure recording), but it defeats AC 9 and AC 10.
- **Fix:** capture's refusal is now a typed error, `engine::DependentEditsPreserved`. `record_sync_failure` leaves the engine-raised reason in place for it, and the runner does not count it as a capture fault. The recorded reason, and with it the attention card's **Finish session**, now reaches CLI `status` and the tray.
- **Tests:** `a_sync_failure_keeps_the_dependent_edits_reason_and_its_remedy` (`tests/engine.rs`) and `edits_depending_on_a_received_draft_show_their_reason_not_a_capture_fault` (`tests/runner.rs`). Both fail on `47f810e` and pass on the fix.
- **Unchanged, noted:** while such edits are pending, `RunnerHandle::join` after a stop still returns that error from the final capture. The bytes stay in pending capture. Pre-existing behavior; the tray ignores the result on Quit.
- **Verified live on the fixed debug build (Windows):** the card shows the engine's reason and a primary **Finish session**, and the activity section's button is hidden. Clicking it cleared the attention, and the dependent edit entered a new session that was shared as draft. With the bare remote renamed away, the card showed "Synchronization waiting: Git fetch …" then "Mesh retries automatically.", no Retry button, and the header kept "Last synced 1 min ago". Restoring the remote cleared it. After restarting the tray, the header showed the stored sync time, not "Not synced yet". **Show in folder** opened Explorer in `.mesh/recovery/<id>` with `content` selected (read back through the Shell API).

### 2026-11-06 — merged

Rebased onto `51cf60e` (docs-only) and fast-forwarded `origin/main` to `676092d` (`eda8c04`, `579f0df`, `8560753`, `6f28cfe`, `676092d`). Code identical to the tested `819d9b0`; fmt and clippy rerun on the rebased tree.

## Code review

### 2026-11-06 — branch US00005-workspace-detail at 4133e9c

**Verdict: blocked.** Two blocking findings, no additional non-blocking findings. Keep `In progress`; this build cannot proceed to `/test-plan` or leave Review as it stands. This is an independent review: this session authored none of the implementation. Scope: all three commits in `main...4133e9c` (`c9a894c`, `fb38b8c`, `4133e9c`), all 17 changed files, the complete ticket and its linked ADR-0009/ADR-0010, plus the recovery and tray-host decisions. No open pull request was found for this branch. No unrelated commits found.

**Gates:** independently passed `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `git diff --check main...HEAD` on Windows. Full `cargo test` passed: 132 tests, zero failures (engine suite 383.50 s); both packages and doc-test targets completed successfully. No live remote, AI-provider, separate-host or UI/Narrator validation was performed in this review. Disposable crash-state CLI evidence is retained at `target/us00005-review-evidence.json` (not in source commits).

**Findings, in severity order**

1. **[P2, blocking — AC 11] An interrupted displacement can permanently list a saved copy that never existed.** `src/engine.rs:1241–1257` removes an abandoned entry only inside the branch that moves the file again; replay's earlier exits at `src/engine.rs:1208–1224` bypass that cleanup. Trigger: state and `record.json` have been saved before `fs::rename`, the process stops, and the user saves a different edit before restart. Replay correctly preserves that newer edit, but subsequently clears `apply_backups`, leaving the nonexistent copy in `saved_copies` indefinitely. Reproduced with an isolated initialized workspace and an injected durable crash journal, followed by ordinary CLI `session finish` and `status`: `recovery_pending` becomes false, the newer user edit survives, and `saved_copies` still contains an id with no `.mesh/recovery/<id>/content`. The window consequently advertises a version it cannot reveal or restore. The one-time backfill at `src/engine.rs:98–107` also accepts a readable record without checking that its content exists, so older interrupted applies can import the same phantom. **Fix:** reconcile unfinished displacement bookkeeping on every replay path, retaining entries for actual displaced content and removing only bookkeeping for copies never moved; require actual content in the legacy backfill. Preserve all existing recovery files and newer local edits. **Regression cases:** crash before rename followed by a newer user save, crash before rename followed by a file already equal to the desired result, crash after rename (real copy retained), and legacy recovery folders with records but no content. Existing tests cover normal displacement and the old after-rename journal, not this branch's new bookkeeping across these boundaries.

2. **[P2, blocking — AC 5 / contract disagreement] Activity selection and ordering do not meet the acceptance criterion literally.** `tray/src/model.rs:395–406` selects settled sessions by `last_saved` and concatenates every unsettled session ahead of them. AC 5 promises “the 10 most recently settled ones … newest first.” For example, an old session that finally publishes after a conflict or dependency clears is omitted if ten already-published sessions have newer saves; an old open session is displayed before a newer published session. The implementation follows plan step 3 and its explicit order-proxy decision, but the acceptance criterion was never aligned with that plan. The current model test at `tray/src/model.rs:847–890` encodes the grouping rather than resolving the disagreement. **Resolution required:** either Kai confirms the planned policy and `/update-ticket` makes AC 5 and the README describe “all unsettled first, then ten settled by last save; each group newest first,” or retain AC 5 and implement a durable settlement timestamp plus the agreed overall ordering. Neither interpretation is silently chosen in this review. Add the delayed-settlement and interleaved open/published cases against the chosen contract.

**Acceptance-criteria check:** AC 1–4: session detail, live snapshots, local-only track, titles, purposes and original authors are implemented with engine/model coverage. AC 5: blocked by F2. AC 6: persisted/coalesced successful remote sync and never-synced/local-only wording covered. AC 7–8: host uses the CLI's engine operation, ownership refusal is tested, menu cardinality is unit-tested and Quit behavior remains intact; desktop invocation is still manual test-plan work. AC 9–10: primary action and retry-note routing covered in the pure model; desktop presentation remains manual. AC 11: normal copies and id-only reveal implemented, but interrupted-copy correctness fails F1. AC 12: additive CLI JSON and old-key coverage passed in this run. AC 13: native buttons/disclosure, accessible names, keyed focus restoration, textual stages and non-color markers are present; actual keyboard/Narrator verification remains manual, not claimed as exercised here.

**Plan versus built:** the recorded deviations (author name rather than “You,” additive `has_remote`, same-second purpose tie-break, worker-side finish retry, and direct Rust opener call) are documented in the dated development entry. The activity proxy is in the plan, so F2 is an unresolved contract inconsistency rather than an undocumented implementation pivot. The ADR-0010 amendment and source documentation were delivered. The recovery replay implementation does not fully deliver its recorded promise that only copies holding a file count (F1).

**What should stay:** one additive engine `Status` feeds CLI and tray, avoiding a second interpretation of workspace detail; Finish calls `Workspace::finish()` off the UI thread and refuses workspaces this host does not own; Show in folder accepts only a listed UUID and resolves its path in Rust; incoming session authors are preserved; stage tracks use text and distinct marks; no file contents enter the webview as markup. These choices maintain the engine/host boundary and make the behavior directly testable.

**Before this leaves review — fix-round plan:**

1. In a separate `/develop US00005` session, repair F1 with the replay and migration regression cases above; do not delete recovery data.
2. Resolve F2's policy with Kai and use `/update-ticket US00005` to align the contract before implementing any policy change. This review changes no contract text.
3. Run the mechanical gates on the fixed build, record the fix and results in As-built notes, and clear `open_findings` to empty for the re-check of this same review. The independent re-check writes `0` only when its blocking findings are resolved.
4. Only then hand off to `/test-plan US00005`, which owns manual checks, `open_tests` and the freeze.

### 2026-11-06 — branch US00005-workspace-detail at 47f810e

**Verdict: clear — F1 and F2 resolved; zero blocking and zero non-blocking findings remain.** This completes the fix re-check of the same review begun at `4133e9c`, independently performed. Scope: the complete four-file fix commit `47f810e`, its tests, the amended AC 5/Scope, README, and the dated fix-round notes. No implementation was authored in this review session. No open PR found for this branch; source working tree clean.

**F1 resolved:** `recover_apply` now reconciles missing displacement content before any replay branch can skip the file, removes the abandoned id from both bookkeeping collections, and persists that correction while retaining the recovery folder. Existing content remains listed. Legacy backfill requires a content file. All five new recovery tests passed in this run: newer user save, already-written desired file, repeat displacement, actual moved content retained, and legacy empty record skipped. Independently repeated the original CLI crash-journal reproduction in a new disposable workspace: `saved_copies` is now empty after replay, the newer user edit survives, `recovery_pending` clears, and the empty recovery record is retained. Evidence: `target/us00005-recheck-evidence.json`. The real-copy tests verify exact retained bytes, so the fix does not achieve a clean list by dropping real copies.

**F2 resolved:** AC 5 explicitly records Kai's 2026-11-06 decision: every unsettled session first, followed by the ten settled sessions with the latest last save, each group newest by last save. Scope, README and the implementation now agree. The new activity regression covers an older open session ahead of a newer published session and a late-settling session excluded by its old last save. No settlement timestamp or different ordering was introduced.

**Gates:** `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `git diff --check 4133e9c..HEAD` passed independently on Windows. Full `cargo test` passed: 138 tests, zero failures; both packages and doc-test targets completed successfully (engine suite 357.27 s). No live remote, AI-provider, separate-host, UI or Narrator verification performed in this re-check.

**What should stay:** cleanup is confined to abandoned bookkeeping and preserves files; the tests check both phantom removal and real-content retention. The existing shared Status, engine Finish operation, id-only reveal and text-based stage rendering remain intact.

**Next:** `/test-plan US00005` against `47f810e`, covering the outstanding desktop/menu/reveal/keyboard/Narrator checks and stamping the freeze. `open_findings: 0`; status stays `In progress`, and `open_tests` remains unset because it belongs to the test plan. No implementation or contract changes made by this re-check.

## Follow-ups

- 2026-11-06: [[Story - US00011 - Limit saved copies in the recovery folder|US00011]] — saved copies grow with every received edit and are never cleaned up (see the observation in the as-built notes).
- 2026-11-06: [[Story - US00012 - Status window layout follows the design frames|US00012]] — the window layout does not follow the design frames (stage track, session rows, state card).

## References

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]
- [[Dispatch Mesh requirements and architecture]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]]
- [[Story - US00001 - Tray icon and status window host|US00001]]

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
