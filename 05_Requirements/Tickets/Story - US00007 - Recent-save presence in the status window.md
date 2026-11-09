---
id: US00007
type: story
status: "Refinement"
priority: medium
rank: 6000
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 7
open_tests:
open_findings:
discussion:
updated: 2026-11-09
completed:
frozen:
---
Show recent-save presence in the status window: per file, who saved it, on which device and how long ago, expiring five minutes after the last save. When a collaborator has recently saved a file that is also in your open session, show a non-blocking notice. The engine already derives presence from open sessions, including drafts received from other devices (`status.presence`, signal `recent_save`); this ticket makes it visible, worded as saved changes and never as "is editing".

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a collaborator, I want to see who recently saved a file I'm working on, so that I can coordinate before our edits collide.

## Context

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]: work item "Show recent-save presence per file, including the collaborator and last observed save. Expire stale signals and distinguish them from actual editor activity."
- [[Dispatch Mesh requirements and architecture]], *Editing presence*: advisory only; per file; expires after five minutes without edits to that file; never generated from applying incoming changes; exchanged independently of permanent commits; accessible from the status window.
- [[Claude Design briefing — desktop app]] §5 ("Alice recently changed this file · 12 s ago"; never "is editing" or "has the file open") and §6.3 item 5.
- Engine: `src/status.rs` builds presence from unsettled open sessions whose file changes are under 300 s old; sessions from other devices arrive with draft sync.
- Suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

## Scope

- Presence per file in the activity list and the workspace view, with author, device and a relative time that updates while the window is open.
- Expiry after five minutes, as the engine reports it.
- A non-blocking notice when another collaborator's presence overlaps a file in your own open session.
- Your own saves are distinguishable from collaborators'.

## Out of scope

- Editor integrations and any "is editing" or "has the file open" signal.
- Toasts for presence (decided in [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]]).
- A separate presence transport, unless the transport question shows the current one misses the requirement.

## Open questions

Recorded here: no tracker or chat is configured. Refinement started 2026-11-09; engine facts verified against `src/status.rs`, `src/engine.rs`, `src/codec.rs`, `src/runner.rs`, `tray/src/model.rs` and frame 3c.

**Engine facts (2026-11-09).**
- `Status.presence` lists one `Presence { path, author, device, last_saved, signal: "recent_save" }` per file change of every session that is **not closed and not settled**, when `now − change.last_edit < 300` (`src/status.rs`). Per-file timing is correct: edits to other files of the session do not refresh a file's entry.
- `change.last_edit` is stamped by the device that captured the save and travels unchanged with draft sync (`src/codec.rs`); the receiving device subtracts it from its **own** clock. `saturating_sub` turns a future timestamp into "0 s ago", so a fast remote clock lengthens presence and a slow one shortens it or hides it entirely.
- A session's `closed` flag also travels with draft sync. **Finish session** (US00005) or the idle boundary closes a session, which removes all its presence entries on every device at the next sync, before the five minutes run out. With the default `idle_seconds: 300` the idle boundary never ends presence early; a lower configured value would.
- The runner captures a save after two equal scans (about 1–2 s) and synchronizes drafts about 3 s after the previous sync ends (`src/runner.rs`), on the draft ref, which is separate from shared history ([[ADR-0001 — Synchronize drafts separately from permanent history|ADR-0001]]).
- Your own sessions are in `presence` too. The status window already calls this device's sessions "You" (US00012 Q5; `tray/src/model.rs`); the same person on another device is shown by name.
- The tray model has a `presence` slot but renders nothing yet.

- [ ] **Q1 Where the notice shows.** In the window only, or also in the tray tooltip or as a toast (decided together with US00008)?
- [ ] **Q2 Own presence.** Show your own recent saves, or collaborators' only? Is "own" this device (the window's "You" rule) or your identity on any device?
- [ ] **Q3 Latency and transport.** Presence now travels with draft sync (every few seconds while online). Does that meet the requirement to exchange presence independently of permanent commits, or does it need its own exchange?
- [ ] **Q4 Clock skew.** `last_saved` comes from the other device's clock. How is "12 s ago" kept honest when clocks differ?
- [ ] **Q5 Finished sessions.** Finishing a session removes its presence everywhere at once, although the requirement says presence expires five minutes after the last save. Keep presence for five minutes after the save regardless of Finish (an engine change), or end it on Finish?
- [ ] **Q6 Notice placement and dismissal.** The window has no per-file view. Where does the "Alice recently changed this file" card appear (top of the workspace view, on the session's file row, both)? Once dismissed, does it return on that collaborator's next save to the file?
- [ ] **Q7 Several entries per file.** Two collaborators, or one person on two devices, can have recent saves to the same file. One row per person and device, or one row per file naming the latest saver ("Alice and Bob recently changed this file")?

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

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]
- [[Dispatch Mesh requirements and architecture]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]]
- [[Story - US00001 - Tray icon and status window host|US00001]]

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
