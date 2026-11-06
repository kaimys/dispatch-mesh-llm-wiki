---
id: TASK00003
type: task
status: "Backlog"
priority: high
rank: 9000
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 3
open_tests:
open_findings:
discussion:
updated: 2026-11-06
completed:
frozen:
---
Prove the 0.3 exit criteria across two devices: presence, conflict notification, propagation of a resolution, recovery after restarting the tray host, and new edits staying safe while an earlier session is summarized or published. Automate what can run against a shared bare remote in the integration suite and the live GitHub harness, and keep a short manual checklist for what needs two real desktops. This is the last 0.3 ticket and depends on US00005–US00008.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As the release owner, I want the 0.3 exit criteria verified with two devices working on one workspace, so that the tray release ships with evidence instead of single-device checks.

## Context

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]] exit criteria: "Two-device tests cover presence, conflict notification, resolution propagation, and recovery after restarting the tray host" and "New edits remain safe while a prior session is being summarized or published".
- [[Story - US00001 - Tray icon and status window host|US00001]]'s test plan left out a real two-device conflict; only the conflict → needs-attention mapping is unit-tested.
- Integration tests already use temporary folders and local bare Git remotes, so two engines can share a remote in-process. `scripts/test_github_workflow.py` runs isolated clients against the designated test remote `https://github.com/kaimys/dispatch-mesh-test.git` (AGENTS.md).
- Depends on [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]], [[Story - US00006 - Conflict list and resolution in the status window|US00006]], [[Story - US00007 - Recent-save presence in the status window|US00007]] and [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]].
- Suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

## Scope

- Integration tests with two hosts on one bare remote: presence appears and expires, a conflict reaches needs attention and a notification decision, a resolution on one device reaches the other, a host restart mid-sync recovers without loss, and edits made during summarization or publication land in a new session.
- Live-harness cases for the same flows where GitHub behaviour matters.
- A manual checklist for the visual and toast parts on two Windows desktops.

## Out of scope

- macOS and Linux desktops (0.7, 0.8).
- Performance or load testing.

## Open questions

- [ ] **Second device.** Two physical PCs, a VM, or two Windows user accounts on one machine for the manual part?
- [ ] **Split.** How much runs in-process against a bare remote, and how much needs the live GitHub harness?
- [ ] **Restart points.** At which moments is the host killed for the recovery check (during draft upload, during publication, while applying incoming content)?

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
