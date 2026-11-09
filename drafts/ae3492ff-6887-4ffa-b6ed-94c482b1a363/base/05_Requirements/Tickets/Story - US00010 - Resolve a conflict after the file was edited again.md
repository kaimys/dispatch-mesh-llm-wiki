---
id: US00010
type: story
status: Refinement
priority: high
rank: 4500
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 5
open_tests:
open_findings:
discussion:
updated: 2026-11-09
completed:
frozen:
---
Today, choosing a version for a conflict fails with "File has newer edits; review them before resolving this conflict" when the file was saved again after the conflict was recorded. Nothing offers a way out except restoring the current version by hand. With this change, resolving is allowed. The later edit counts as a change on top of the current version and is three-way merged onto the chosen version: a clean merge is applied, and overlapping edits become a new conflict. Neither the later edit nor either conflict input is lost. This applies to CLI `conflicts resolve` and to the status window ([[Story - US00006 - Conflict list and resolution in the status window|US00006]]), which call the same engine operation.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a collaborator who kept working on a file after Mesh recorded a conflict in it, I want to choose a version without first undoing my newer edit, so that my newer work is kept and combined with the version I chose.

## Context

- Origin: [[Story - US00006 - Conflict list and resolution in the status window|US00006]] Q4, answered by Kai on 2026-11-06: "the engine reconciles … built as a separate engine story that US00006 depends on". US00006 is complete without this story; until this story lands, its window shows the engine's refusal (US00006 acceptance criterion 10).
- The [[Claude Design briefing — desktop app]] §6.3 item 4 and the conflict-detail frame in [[Dispatch Mesh desktop app.pdf]] promise this: "You saved this file again at 11:20, after the conflict was recorded. That edit is protected and reconciled separately once you choose."
- [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]: a resolution selects one of the recorded inputs against their contributing revisions, and later distinct edits are reconciled separately. AGENTS.md states the same invariant. This story makes that hold for a *local* later edit at resolve time, not only for publication order.
- **Engine today (verified 2026-11-06):**
  - `Workspace::resolve(id, incoming)` (`src/engine.rs`, around line 1414) calls `capture_at(now())`. Its guard `state.observed[path] == conflict.accepted` then refuses when the folder differs from the current version. Otherwise it inserts a `Resolution { selected, author, created_at }` and calls `render()`.
  - `capture_at` captures saved edits into the local session as a `Change { base, content }`. The base of a later edit is the folder view the user edited, which is the current version, and the change merges with an existing open change via `git.merge` (line ~492).
  - `merge_change` / `matching_resolution` (line ~984 onward) let a recorded resolution govern the original inputs whatever order they are published in. A conflict is created through `state.conflicts.entry(..).or_insert_with(..)` with `accepted_sessions` holding the contributing revisions.
  - No test covers the refusal path. Resolve tests: `tests/engine.rs` lines ~149, ~370, ~437.
- **Engine re-read during refinement (2026-11-09):**
  - `resolve` is now at `src/engine.rs` ~1536 and the guard at ~1549. The guard ignores the choice: "keep current" is refused as well as "choose incoming".
  - `render()` rebuilds the folder from `main` plus every unsettled session. A later edit captured as its *own* change (base = current version) is therefore already three-way merged onto the selected version by `merge_change` once a resolution exists. On overlap, `merge_change` records a new conflict with the selected version on the current side and keeps that current side in the folder, as ADR-0003 requires ("keep the accepted view visible").
  - If the user's open session already holds their conflict input for the path, `capture` folds the later edit into that same change (`git.merge`, ~540). The change's fingerprint then no longer matches `accepted_sessions`, so `matching_resolution` stops recognising it as the recorded input. Nothing prunes `state.conflicts`. This is the core of Q4.
  - While the conflict is open, `publish_ready` skips every session that contains the path, so both contributors' sessions (including their other files) stay unpublished.
- CLI: `dispatch-mesh conflicts resolve <id> --choose accepted|incoming` (`src/main.rs`).

## Scope

- `Workspace::resolve` no longer refuses when the file changed after the conflict was recorded. The later local edit is reconciled onto the selected version: a three-way merge with the current version as base, the later edit as one side and the selected version as the other.
- A clean merge is written to the folder and stays the local user's change, in their session with its own attribution and purpose. It never becomes part of the resolution record.
- Overlapping edits keep the later edit in the folder and record a new conflict between it and the selected version. The original conflict is still marked resolved with the user's choice.
- The resolution record keeps selecting exactly one recorded input (ADR-0003); synchronization and publication order behave as today.
- CLI wording: `conflicts resolve` reports whether a later edit was combined or produced a new conflict.
- Regression tests with real workspaces for each outcome, for both choices, and across two devices with either publication order.

## Out of scope

- Conflict UI: the list, detail view and wording in the status window belong to [[Story - US00006 - Conflict list and resolution in the status window|US00006]].
- Writing a combined third version by hand, and AI-proposed merges ([[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]]).
- Changing how conflicts are detected or recorded in the first place.

## Open questions

Recorded here: no tracker or chat is configured.

- [x] **Q1 Keep current version with a later edit.** Choosing the current version is a no-op on the folder, since the later edit is already based on it. Is that the whole behavior: the later edit simply stays as the user's next change?
  - **Answer (Kai, 2026-11-09):** Yes. The folder stays unchanged, the later edit remains an ordinary change of the local user (own attribution and purpose), and the conflict is marked resolved with the current version selected. `conflicts resolve` reports that the later edit was kept, alongside "combined" and "new conflict" for the other choice. Rejected alternatives: staying silent, because the two choices would then report differently and the protected edit would go unconfirmed; folding the later edit into the resolution, because the resolution would select a version that is not a recorded input (ADR-0003) and new work would be mixed into a record that holds another author's input. Precondition: the later edit must stay a change separate from the recorded conflict input (see Q4). Otherwise the next `render()` can merge the other input against the later edit and record a fresh conflict right after "keep current".
- [ ] **Q2 Overlapping later edit.** When the merge overlaps, which version stays in the folder (proposal: the later edit, never overwritten), and how are the new conflict's sides attributed? Proposal: the current side is the local user's later-edit session, and the other side is the selected version with its original author.
- [ ] **Q3 Binary files and deletions.** A later edit to a binary file cannot be merged, and either side may be a deletion. Does choosing the other version then always produce a new whole-file conflict, or does the later edit win?
- [ ] **Q4 Session state of the later edit.** The later edit may sit in an open session, a closed one waiting for finalization, or one already published, since finalization waits only for sessions containing an unresolved path. How is its change rebased onto the selected version in each case, so that publication on `main` is exactly "selected version → later edit" and no empty or duplicate commit appears?
- [ ] **Q5 Received changes after the conflict.** The guard compares against the folder, which also holds content received from other devices after the conflict. Is a received later change reconciled the same way, or is only a local edit in scope?
- [ ] **Q6 Compatibility.** Does this need any change to the synchronized conflict or resolution format? If it does, how do clients that don't have the change yet behave, given that they still refuse? (Expectation: no format change, because the resolution stays a choice between inputs and the reconciliation is an ordinary session change.)

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

- [[Story - US00006 - Conflict list and resolution in the status window|US00006]] (origin, Q4)
- [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]] (conflict detail frame)
- Code: `src/engine.rs` (`resolve`, `capture_at`, `merge_change`, `matching_resolution`), `src/main.rs` (`conflicts resolve`), `tests/engine.rs`

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
