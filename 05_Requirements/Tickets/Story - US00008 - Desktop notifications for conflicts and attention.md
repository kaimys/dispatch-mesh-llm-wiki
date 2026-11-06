---
id: US00008
type: story
status: "Backlog"
priority: medium
rank: 7000
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 5
open_tests:
open_findings:
discussion:
updated: 2026-11-06
completed:
frozen:
---
Send native notifications for meaningful changes only: a new conflict, an attention state that starts, and an attention state that clears. Never one per sync cycle, and never a repeat of an unchanged warning. The rules live in the Tauri-free view model so they are unit-testable, and US00001's first-close toast moves under them. This closes the milestone's open decision on notification behavior.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As someone who keeps the status window closed, I want to be told when Mesh needs me and when the problem is gone, without being nagged on every sync cycle.

## Context

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]: work item "Notify on meaningful failures or conflicts without repeating unchanged warnings on every synchronization cycle"; *Decisions to close during delivery* lists notification behavior as open.
- [[Claude Design briefing — desktop app]] §6.4 (new conflict, attention starts, attention clears; supply wording and a frequency rule); wording in [[Dispatch Mesh desktop app.pdf]], deliverable 4.
- [[Story - US00001 - Tray icon and status window host|US00001]] uses `tauri-plugin-notification` for its single first-close toast and says the notifications ticket owns the toast rules. In US00001's manual test the toast appeared on the unpackaged build.
- Attention reasons and the calm offline state come from [[ADR-0010 — Report workspace status and session stages separately]].
- Suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

## Scope

- Notification rules in `tray/src/model.rs`: which transitions notify, the deduplication key and a frequency limit, per workspace.
- Toasts for a new conflict, attention starting and attention clearing, with the design's wording; clicking one opens the window on that workspace.
- The first-close toast follows the same rules and settings.
- Unit tests for the rules, including unchanged warnings across many cycles.

## Out of scope

- Presence toasts, unless the presence question decides otherwise ([[Story - US00007 - Recent-save presence in the status window|US00007]]).
- Login startup and a general preferences window (0.6).

## Open questions

- [ ] **Frequency rule.** What identifies "the same warning" (reason text or reason category), and is there a cool-down before a cleared-then-returning problem notifies again?
- [ ] **Offline.** Does going offline notify, given the design treats offline as calm?
- [ ] **Presence.** Does a collaborator's recent save in your file notify (US00007)?
- [ ] **Muting.** Is there a setting to turn notifications off in 0.3, or does that wait for preferences in 0.6?
- [ ] **Click activation.** Does clicking a toast open the window on the unpackaged build, or does that need the packaged app identity from 0.4?

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
