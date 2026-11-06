---
id: US00011
type: story
status: "Backlog"
priority: medium
rank: 10000
version_target:
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
Saved copies in `.mesh/recovery/` grow without bound. Each change Mesh receives for a file that already exists moves the old version into `.mesh/recovery/<id>/` before writing the new one, and nothing ever removes these copies. Since [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]], `status` JSON and the tray window also list every one of them, so a busy workspace accumulates disk use, a growing status payload and a list too long to use. This ticket decides which copies are worth keeping and how long, and puts that rule into the engine.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a person working in a busy Mesh workspace, I want Mesh to keep only the saved copies that can still help me recover something, so that the recovery folder, the status output and the window's list stay small enough to be useful.

## Context

- Found during development of [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] (as-built notes, 2026-11-06). In a two-workspace demo, a single received edit to `guide.md` already produced a saved copy.
- **Engine facts checked 2026-11-06 (`src/engine.rs`):**
  - `recover_apply` moves *every* existing file it is about to overwrite or delete into `.mesh/recovery/<uuid>/content` with a `record.json` (`path`, `created_at`), not only files a racing save changed.
  - `install` refuses to write while the folder differs from the last installed view (`state.observed`); it captures the edits and retries. After the move, a copy whose bytes differ from that view aborts the apply ("A save raced incoming changes …"). So a copy kept by a successful apply holds the bytes of the last installed view, which `.mesh/state.sqlite` already stores as a captured version (README *Storage growth*: "every captured version"). Copies from a race are the exception and may hold bytes found nowhere else. Refinement should confirm this before relying on it.
  - Since US00005, displaced files are recorded in `State.saved_copies` and reported in `status.saved_copies`; the tray lists them newest first with **Show in folder**.
- README *Storage growth* documents today's behavior: "Files displaced while applying incoming content. Each is kept indefinitely." [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] documented growth and explicitly left retention out of scope.
- [[Dispatch Mesh milestone plan]] 0.4: "Keep managed documents and recovery data safe across upgrades and uninstall operations. Define any explicit data-removal action separately." [[Dispatch Mesh SWOT analysis]]: storage growth needs measurement before retention or backend changes.
- Invariant (`AGENTS.md`): never discard uncaptured edits or original conflict inputs. Removing a copy that holds the only instance of some bytes would break it.

## Scope

- A rule, in the engine, for which displaced files become saved copies and which existing copies are removed. It applies the same way for the CLI and the tray.
- `State.saved_copies`, `status.saved_copies` and `.mesh/recovery/` stay consistent with each other after any cleanup.
- README *Storage growth* and *Recovery procedures* updated to the new rule; ADR if the rule qualifies.

## Out of scope

- Pruning `.mesh/state.sqlite` blobs, `.mesh/objects.git`, device branches or `main` (wider storage growth).
- Restoring a saved copy from the window (kept out in US00005).
- Uninstall-time data removal (0.4).

## Open questions

- [ ] **Redundant copies.** If a copy kept by a successful apply always holds bytes Mesh already stores (see the engine facts), should Mesh stop creating it and keep copies only from a race? Or is a plain file in the folder worth keeping, because users can reach it without Mesh?
- [ ] **Retention.** For the copies that are kept, what limit applies: age (e.g. 30 days), count per workspace or per path, total size, or none? Must copies that may hold the only instance of some bytes be exempt?
- [ ] **Who removes them.** Automatically during sync, or only through an explicit user action (CLI command, tray button)? The milestone plan asks for data-removal actions to be defined separately.
- [ ] **Existing copies.** What happens to copies already accumulated by 0.2/0.3 workspaces: same rule on upgrade, or left alone?
- [ ] **Display.** Independent of retention, should `status` and the window cap or summarize the list (e.g. newest N plus a count)?

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

- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] — as-built notes, observation on saved-copy growth
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] — storage growth documentation
- [[Dispatch Mesh milestone plan]], [[Dispatch Mesh SWOT analysis]]
- [[ADR-0007 — Persist state transactionally with recovery journals]], [[ADR-0010 — Report workspace status and session stages separately]]
- README: *Storage growth*, *Recovery procedures*

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
