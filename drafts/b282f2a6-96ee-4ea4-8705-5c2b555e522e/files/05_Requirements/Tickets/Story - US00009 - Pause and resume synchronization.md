---
id: US00009
type: story
status: "Backlog"
priority: medium
rank: 8000
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 6
open_tests:
open_findings:
discussion:
updated: 2026-11-06
completed:
frozen:
---
Let the user pause and resume Mesh from the tray, after deciding exactly what pause suspends. The decision extends ADR-0010's status model with a paused state, distinguishes a user's pause from the engine's own "sync paused" attention states, and fills the paused glyph slot US00001 reserved. The milestone requires the semantics to be defined and labeled before implementation.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a user on a metered connection or giving a presentation, I want to pause Mesh and resume it later, knowing exactly what keeps happening while it is paused, so that I stay in control without risking my edits.

## Context

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]: "Add pause/resume … Define and label exactly what pause suspends before implementing it"; *Decisions to close during delivery* lists pause semantics as open.
- [[Dispatch Mesh requirements and architecture]] asks for pause, resume, quit and recovery actions.
- [[Story - US00001 - Tray icon and status window host|US00001]], open question *Icon state set*: **Paused** is reserved; the pause ticket adds the icon together with the ADR-0010 extension. [[Claude Design briefing — desktop app]] §6.2 puts **Pause / Resume** in the tray menu.
- The engine already pauses synchronization on its own in some attention states ("Shared history was rewritten outside Mesh"); README lists "an engine pause" under `needs_attention`. A user's pause must read differently.
- Engine control goes through `RunnerHandle` ([[ADR-0009 — Embed the engine with a runner handle and published status]]).
- Suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

## Scope

- The pause decision, recorded as an ADR or an amendment to ADR-0010.
- Engine support for pause and resume through the runner handle, with the paused state in `Status`.
- Tray menu **Pause** / **Resume**, the paused icon and label, and the window's paused state.

## Out of scope

- Excluding individual files or folders ([[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]]).
- Scheduled or metered-connection-aware automatic pause.

## Open questions

- [ ] **What pause suspends.** Network only (fetch, draft upload, publication) while capture continues, or capture too? Recommendation: keep capturing, because capture is local and protects edits.
- [ ] **Finalization and AI.** Do sessions still close and get summarized while paused? A summary may send text to a provider.
- [ ] **Duration.** Indefinite, or timed options (1 hour, until tomorrow)?
- [ ] **Restart.** Does a pause survive Quit and the next start?
- [ ] **Precedence and wording.** Where does paused rank against needs attention and offline, and how is it worded so it is never confused with the engine's own "sync paused" attention states?
- [ ] **Scope.** Per workspace, or for all workspaces in the host?

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
