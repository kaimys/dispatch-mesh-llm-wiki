---
id: TASK00001
type: task
status: Refinement
priority: high
rank: 1024
version_target: v0.2.5
size:
assignee:
owner: "[[Kai]]"
open_questions: 0
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

Refinement findings (2026-11-05):
- **Interrupted publication already has a recovery path; it is only untested.** `Workspace::sync` (`src/engine.rs:580`) calls `reconcile_published` before `publish_ready` on every cycle. `reconcile_published` marks a session published when a commit on `main` ends with its `Mesh-Session: <id>` and `Mesh-Snapshot: <hash>` trailers. So a session pushed to `main` but not yet recorded locally should be recognized on the next sync rather than published twice. The test must prove exactly this.
- **Only one consumer reads the `status` value.** That is `tests/cli.rs:250`, which asserts `"synced"`. The GitHub harness's `status` keys are its own PASS/FAIL results, not Mesh's. No external consumer exists yet; the tray (US00001) will be the first.
- **Failure persistence is decided upstream.** [[Story - US00002 - Local engine control interface for the tray host|US00002]] decides that AI-summary failures are persisted in state, alongside sync failures, and that the runner's capture, sync and summary errors travel in the published status snapshot. This ticket's status model defines how they are presented.

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

- [x] **Split or keep together?** Should these four items stay in one ticket, or split into separate tickets? The status-model item blocks US00001 and is much larger than the others.
  **Answer (Kai, 2026-11-05):** keep together in one ticket. The implementation plan orders the status model first, because US00001 depends on it.
- [x] **Status shape.** One top-level state with a precedence order (which wins when a workspace is both offline and has a draft waiting?), a set of flags, or per-session states plus a workspace summary?
  **Answer (Kai, 2026-11-05):** a workspace summary plus a stage per session.
  - **Session stage**, one of `saved_locally` → `draft_uploaded` → `waiting_for_finalization` → `published`, or `discarded`.
  - **Workspace summary**, one state with a fixed precedence: `needs_attention` > `offline` > `saved_locally` > `draft_uploaded` > `waiting_for_finalization` > `synced`. Here `synced` means every own session is published or discarded and nothing is waiting.
  - **`reason`**, naming the cause whenever the summary is `needs_attention` or `offline`.
  - The tray icon reads only the summary; detail views read the sessions.
- [x] **Offline detection.** What makes Mesh decide it is offline rather than failing? Connection-level transport errors only, or also authentication failures? And after how many consecutive failures?
  **Answer (Kai, 2026-11-05):** connection-level errors only, from the first failure.
  - **Offline** means the last sync failed with a network-level transport error: DNS resolution, connection refused or unreachable, timeout, TLS handshake. It clears on the next successful sync.
  - **Needs attention** remains the state for authentication, permission, rejected-push and history failures, because the user has to act.
  - Local capture continues in both states.
- [x] **"Draft uploaded" bookkeeping.** Should the engine record the last draft revision successfully pushed per session (a new state field and migration), or derive it from the device branch on demand?
  **Answer (Kai, 2026-11-05):** store it. After a successful device-branch push, record each own session's pushed revision; a session is `draft_uploaded` when that equals its current `revision`. The record must load as empty from older state, so no migration step is needed.
  > Constraint found while recording this (2026-11-05): keep the record **local-only**, e.g. `State.uploaded: BTreeMap<session id, revision>`, not a field on `Session`. `push_manifest` serializes own `Session`s into the device-branch manifest. A field set after each push would change that manifest and trigger another push on every cycle, and it would also publish a device-local fact to peers.
- [x] **Compatibility.** Can existing `status` JSON consumers (scripts, the GitHub harness) tolerate a changed `status` value set, or must the old three values stay as a separate field?
  **Answer (Kai, 2026-11-05):** replace and document. `status` carries the new workspace summary values: `synced` and `needs_attention` keep their names, and `waiting_to_sync` is removed. Update `tests/cli.rs` accordingly and document the value set in the README. There is no legacy field, because no external consumer exists in this 0.x preview.
- [x] **Docs location.** Should the documentation go in the README only, or the README plus a recovery page in the wiki?
  **Answer (Kai, 2026-11-05):** README only, next to the existing *Conflicts and recovery* and limits sections, so the documentation versions with the code.

## Acceptance criteria

**Tests**
1. An automated test interrupts publication after a finalized session's commit has reached `main` on the remote, but before local state marks it published. After reopening and syncing, the session is marked published, `main` contains exactly one commit for it, and no captured content is lost.
2. An automated two-client test renames a file (path containing spaces and non-ASCII characters) and deletes another file on one client. The other client converges to the same files, with no leftover copy at the old path, and both changes finalize on `main`.

**Status model**
3. `status` reports a workspace summary `status` with one of `needs_attention`, `offline`, `saved_locally`, `draft_uploaded`, `waiting_for_finalization` or `synced`, chosen by that precedence. `waiting_to_sync` no longer appears.
4. Each listed session carries a `stage` of `saved_locally`, `draft_uploaded`, `waiting_for_finalization`, `published` or `discarded`.
5. A session is `draft_uploaded` only after its current revision was successfully pushed to the device branch. That record is local-only: it never appears in the device-branch manifest, and recording it does not trigger an extra push. Older workspaces load with an empty record.
6. A sync failing with a connection-level error (DNS, connect, timeout, TLS handshake) yields `offline` with a `reason`, and capture continues. Authentication, permission and rejected-push failures yield `needs_attention` with a `reason`. The next successful sync clears either state.
7. When the summary is `needs_attention` or `offline`, `reason` names the cause, drawn from sync, capture or the persisted AI-summary failure ([[Story - US00002 - Local engine control interface for the tray host|US00002]]), or from unresolved conflicts. It never includes credentials or provider response bodies.
8. Automated tests cover each summary value and each session stage, including offline vs. authentication failure. `tests/cli.rs` is updated for the new values.

**AI failures**
9. On HTTP 429, a provider error code/type of quota exhaustion (e.g. `insufficient_quota`) reports "quota or billing exhausted". A rate-limit code, or a `Retry-After` header, reports "temporary rate limit", including the retry delay when given. Without either, it reports the current generic hint.
10. Mock-provider tests cover all three 429 cases and assert that no response body text appears in the message.

**Documentation**
11. README.md documents:
    - the `status` summary values, session stages and their precedence;
    - storage growth: what accumulates in `.mesh`, on device branches and on `main`, and the current limits;
    - a recovery procedure for each `needs_attention` cause, for `offline`, for the pending-capture fallback, and for restoring a file from `.mesh/recovery/<id>/`.

**Gates**
12. `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` pass.

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
