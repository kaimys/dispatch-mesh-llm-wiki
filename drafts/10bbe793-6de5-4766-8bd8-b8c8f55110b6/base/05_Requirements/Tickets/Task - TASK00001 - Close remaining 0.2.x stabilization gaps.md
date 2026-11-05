---
id: TASK00001
type: task
status: "Backlog"
priority: high
rank: 2000
version_target: v0.2.5
size:
assignee:
owner: "[[Kai]]"
open_questions: 6
open_tests:
open_findings:
discussion:
updated: 2026-11-05
completed:
frozen:
---
Finish the four 0.2.x engine-stabilization items that are only partly done in 0.2.4, so the milestone's exit criteria hold before the tray work builds on them:
1. regression tests for interrupted publication and renames;
2. a status model that separates the six states the plan names;
3. AI failure messages that tell a rate limit apart from exhausted quota;
4. documentation of storage growth and recovery procedures.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a Mesh user, I want to see exactly where my changes are (saved, uploaded as a draft, waiting, published, offline or stuck) and why an operation failed. I also want to trust that a publication interrupted halfway or a rename never loses work, so that I can rely on Mesh before it gets a desktop interface.

## Context

Gap analysis of 2026-11-05 against [[Dispatch Mesh milestone plan]] → *0.2.x — engine stabilization*, from the code, tests and README at 0.2.4 and the live run in `reports/github-workflow-findings-2026-11-04.md`.

1. **Regression coverage.** Two-device drafts, restart and crash recovery, offline capture, binary conflicts and edits during incoming changes are covered (`tests/engine.rs`, `tests/cli.rs`). Two cases are missing:
   - **Interrupted publication:** a crash or network loss between pushing a finalized session to `main` and recording it as published. `reconcile_published` in `src/engine.rs` suggests a recovery path, but no test asserts it.
   - **Renames:** exercised only by the live GitHub harness (`scripts/test_github_workflow.py`), not by the automated suite.
2. **Status model.** `show_status` (`src/main.rs:220`) reports `synced`, `waiting_to_sync` or `needs_attention`. The plan requires six distinguishable states: *saved locally*, *draft uploaded*, *waiting for finalization*, *published*, *offline* and *needs attention*. Today "saved locally" and "waiting" are merged, and per-session JSON exposes `closed`/`published` but no "draft uploaded" fact. Exit criterion: "A user can distinguish a draft upload from publication on `main` and can find the cause of a failed operation." Found during US00001 refinement: an offline device currently surfaces as `needs_attention` through the sync-failure attention message.
3. **AI failures.** `http_failure` (`src/ai.rs:138`) gives distinct hints for 400/401/403/404/5xx and for timeouts and connection failures. HTTP 429 covers both a temporary rate limit and exhausted quota, and gets one hint ("check rate limits and available API quota"). The plan asks to distinguish them "where the provider supplies that information". OpenAI-compatible providers send an error `code`/`type` (e.g. `insufficient_quota` vs. `rate_limit_exceeded`) and a `Retry-After` header. Invariant: diagnostics must not echo response bodies or credentials.
4. **Documentation.** The README documents path/size limits, the ignore policy and the recovery folder. It defers storage growth ("retention, and application-level cleanup remain future work") and gives no step-by-step recovery procedure for sync paused for attention, pending-capture fallback, or restoring from `.mesh/recovery/<id>/`.

Downstream: [[Story - US00001 - Tray icon and status window host|US00001]] derives its icon states from item 2, and [[Story - US00002 - Local engine control interface for the tray host|US00002]] transports that status to the host.

## Scope

- **Tests:**
  - An automated regression for interrupted publication: process stops or the network fails after the push to `main` but before local state records it. After restart, the session is marked published exactly once, with no duplicate commit and no lost captured work.
  - An automated two-client regression for renames, including paths with spaces and non-ASCII characters, combined with a separate deletion.
- **Status model:** extend `status` (and the engine state it reads) so that each of the six states can be told apart, per workspace and per session where meaningful, with offline distinct from needs-attention. Keep the existing fields, or document any field change in the README.
- **AI failures:** read the provider's error code/type and `Retry-After` from a 429 without logging the body. Report "temporary rate limit (retry after N s)" separately from "quota or billing exhausted". Keep the current generic hint when the provider supplies neither. Add mock-provider tests for each case.
- **Docs:** a README section on storage growth (what accumulates in `.mesh`, the device branches and `main`, and current limits), plus recovery procedures for each attention state and for `.mesh/recovery`.

## Out of scope

- Implementing retention or cleanup of stored history (a documented limitation, not a fix here).
- The host control channel ([[Story - US00002 - Local engine control interface for the tray host|US00002]]).
- Tray UI presentation of the states ([[Story - US00001 - Tray icon and status window host|US00001]]).
- Provider-specific adapters beyond OpenAI-compatible Chat Completions.

## Open questions

- [ ] **Split or keep together?** Should these four items stay in one ticket, or split into separate tickets? The status-model item blocks US00001 and is much larger than the others.
- [ ] **Status shape.** One top-level state with a precedence order (which wins when a workspace is both offline and has a draft waiting?), a set of flags, or per-session states plus a workspace summary?
- [ ] **Offline detection.** What makes Mesh decide it is offline rather than failing? Connection-level transport errors only, or also authentication failures? And after how many consecutive failures?
- [ ] **"Draft uploaded" bookkeeping.** Should the engine record the last draft revision successfully pushed per session (a new state field and migration), or derive it from the device branch on demand?
- [ ] **Compatibility.** Can existing `status` JSON consumers (scripts, the GitHub harness) tolerate a changed `status` value set, or must the old three values stay as a separate field?
- [ ] **Docs location.** Should the documentation go in the README only, or the README plus a recovery page in the wiki?

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

- [[Dispatch Mesh milestone plan]] — 0.2.x engine stabilization (work items and exit criteria)
- `reports/github-workflow-findings-2026-11-04.md` — live verification of 0.2.4
- [[Story - US00001 - Tray icon and status window host|US00001]], [[Story - US00002 - Local engine control interface for the tray host|US00002]]
- Code: `src/main.rs` (`show_status`), `src/engine.rs` (`reconcile_published`), `src/ai.rs` (`http_failure`), `README.md` (*Conflicts and recovery*, limits)

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
