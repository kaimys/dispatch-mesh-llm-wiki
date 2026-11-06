---
id: US00006
type: story
status: "Backlog"
priority: high
rank: 5000
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
List unresolved conflicts in the status window and resolve them there. Each conflict shows its path, the competing authors and versions, timestamps, and which version is currently in the folder. Text conflicts open a comparison of the *current version* and the *other version*, with the common base available; binary conflicts offer whole-file choice only. **Keep current version** / **Use other version** use the same engine operation as `dispatch-mesh conflicts resolve --choose accepted|incoming`, and both versions stay preserved. This is the largest piece of 0.3 and carries the exit criterion "resolve a conflict through the desktop interface".

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a collaborator whose edit collided with someone else's, I want to see both versions side by side and choose one in the status window, so that I can settle the conflict without the CLI and without losing either version.

## Context

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]: work item "List unresolved conflicts. Show the path, competing authors and versions, and the currently displayed version. Support text comparison and version selection; binary resolution offers whole-file selection"; exit criterion "resolve a conflict through the desktop interface".
- [[Dispatch Mesh requirements and architecture]], *Conflict handling recommendation* and *Durable conflict records*; [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]: a resolution is a new traceable change, inputs are retained, and edits made since the conflict was recorded are not overwritten.
- [[Claude Design briefing — desktop app]] §6.3 items 3–4 (conflicts list, conflict detail), §5 vocabulary (*current version* / *other version*, never ours/theirs), §7 (compact window, expandable for comparison), §8 sample conflicts (`policies/retention.md`, `assets/logo.png`); frames in [[Dispatch Mesh desktop app.pdf]].
- CLI today: `conflicts list`, `show`, `export`, `resolve --choose accepted|incoming`. The tray already shows needs attention with "N unresolved conflicts" ([[Story - US00001 - Tray icon and status window host|US00001]]).
- [[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]] (AI-proposed merges, v0.4.0) builds on this review surface; its Q1 asks which manual review capabilities must exist first.
- Suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

## Scope

- Conflicts section per workspace with a count; the attention card's **Open conflicts** action leads there.
- Conflict detail for text: comparison of current and other version, base on demand, author, session, time and requested purpose per side.
- Conflict detail for binary files: metadata per side (size, type, author, time) and whole-file choice, no merge.
- **Keep current version** / **Use other version**, calling the engine's resolve operation in-process, with a clear statement that both versions are kept and later edits are protected.

## Out of scope

- Writing a third, combined version by hand, and AI-proposed merges ([[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]]).
- Toasts for new conflicts ([[Story - US00008 - Desktop notifications for conflicts and attention|US00008]]).
- Two-device verification of propagation ([[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]]).

## Open questions

- [ ] **Layout.** Side-by-side or inline comparison, and does the window widen for it, or does conflict detail open in its own window?
- [ ] **Engine access.** Does the host read conflict detail (sides, authors, sessions, purpose, base) through an existing engine call, or does the runner need a conflict query and resolve API (an ADR-0009 extension)?
- [ ] **Binary previews.** Which types get a preview (images only?), and how does the frontend receive blob content without the webview reading arbitrary paths?
- [ ] **Text limits.** What happens for very large or non-UTF-8 text files: truncated comparison, or metadata only, like binaries?
- [ ] **After resolving.** What does the window show afterwards, and how does it explain edits made since the conflict was recorded?

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
