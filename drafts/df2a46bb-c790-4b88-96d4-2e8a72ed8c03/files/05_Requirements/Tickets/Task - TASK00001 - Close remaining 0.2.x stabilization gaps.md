---
id: TASK00001
type: task
status: In progress
priority: high
rank: 1024
version_target: v0.2.5
size:
assignee: Kai
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

Gap analysis of 2026-11-05 against [[Dispatch Mesh milestone plan]] → *0.2.x — engine stabilization*, from the code, tests and README at 0.2.4 and the live run in [[github-workflow-findings-2026-11-04]].

1. **Regression coverage.** Two-device drafts, restart and crash recovery, offline capture, binary conflicts and edits during incoming changes are covered (`tests/engine.rs`, `tests/cli.rs`). Two cases are missing:
   - **Interrupted publication:** a crash or network loss between pushing a finalized session to `main` and recording it as published. `reconcile_published` in `src/engine.rs` suggests a recovery path, but no test asserts it.
   - **Renames:** exercised only by the live GitHub harness (`scripts/test_github_workflow.py`), not by the automated suite.
2. **Status model.** `show_status` (`src/main.rs:220`) reports `synced`, `waiting_to_sync` or `needs_attention`. The plan requires six distinguishable states: *saved locally*, *draft uploaded*, *waiting for finalization*, *published*, *offline* and *needs attention*. Today "saved locally" and "waiting" are merged, and per-session JSON exposes `closed`/`published` but no "draft uploaded" fact. Exit criterion: "A user can distinguish a draft upload from publication on `main` and can find the cause of a failed operation." Found during US00001 refinement: an offline device currently surfaces as `needs_attention` through the sync-failure attention message.
3. **AI failures.** `http_failure` (`src/ai.rs:138`) gives distinct hints for 400/401/403/404/5xx and for timeouts and connection failures. HTTP 429 covers both a temporary rate limit and exhausted quota, and gets one hint ("check rate limits and available API quota"). The plan asks to distinguish them "where the provider supplies that information". OpenAI-compatible providers send an error `code`/`type` (e.g. `insufficient_quota` vs. `rate_limit_exceeded`) and a `Retry-After` header. Invariant: diagnostics must not echo response bodies or credentials.
4. **Documentation.** The README documents path/size limits, the ignore policy and the recovery folder. It defers storage growth ("retention, and application-level cleanup remain future work") and gives no step-by-step recovery procedure for sync paused for attention, pending-capture fallback, or restoring from `.mesh/recovery/<id>/`.

Refinement findings (2026-11-05):
- **Interrupted publication already has a recovery path; it is only untested.** `Workspace::sync` (`src/engine.rs:580`) calls `reconcile_published` before `publish_ready` on every cycle. `reconcile_published` marks a session published when a commit on `main` ends with its `Mesh-Session: <id>` and `Mesh-Snapshot: <hash>` trailers. So a session pushed to `main` but not yet recorded locally should be recognized on the next sync rather than published twice. The test must prove exactly this.
- **Only one consumer reads the `status` value.** That is `tests/cli.rs:250`, which asserts `"synced"`. The GitHub harness's `status` keys are its own PASS/FAIL results, not Mesh's. No external consumer exists yet; the tray (US00001) will be the first.
- **Git error text is locale-dependent (2026-11-05, decided by Kai).** `src/git.rs` runs the `git` executable and sets only `GIT_TERMINAL_PROMPT=0` and `GCM_INTERACTIVE=never`. Git translates its messages according to the user's locale, so classifying a failed sync as offline vs. authentication from stderr would misfire on a non-English system, such as a German Windows. Every Git invocation must therefore force untranslated output with `LC_ALL=C`. The longer-term move from the Git CLI to a library is [[Task - TASK00002 - Choose the Git backend before bundling Git|TASK00002]].
- **Failure persistence is decided upstream.** [[Story - US00002 - Local engine control interface for the tray host|US00002]] decides that AI-summary failures are persisted in state, alongside sync failures, and that the runner's capture, sync and summary errors travel in the published status snapshot. This ticket's status model defines how they are presented.

Downstream: [[Story - US00001 - Tray icon and status window host|US00001]] derives its icon states from item 2, and [[Story - US00002 - Local engine control interface for the tray host|US00002]] transports that status to the host.

## Scope

- **Tests:**
  - An automated regression for interrupted publication: process stops or the network fails after the push to `main` but before local state records it. After restart, the session is marked published exactly once, with no duplicate commit and no lost captured work.
  - An automated two-client regression for renames, including paths with spaces and non-ASCII characters, combined with a separate deletion.
- **Status model:** extend `status` (and the engine state it reads) so that each of the six states can be told apart, per workspace and per session where meaningful, with offline distinct from needs-attention. Keep the existing fields, or document any field change in the README. Force `LC_ALL=C` on every Git invocation so the offline/attention classification reads untranslated Git messages.
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
6a. Every `git` process Mesh starts runs with `LC_ALL=C`, set centrally in `Git::command()`, so failure classification reads untranslated messages. A test asserts the classification with a non-English locale (e.g. `LANG=de_DE.UTF-8`) in the test process environment.
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

*Drafted 2026-11-05 against `main` at `3c2d513`. Awaiting Kai's sign-off.*

### Ground truth that shapes the plan

- **Git stderr is discarded on purpose.** `Git::execute` (`src/git.rs:46`) bails with a generic "Git operation failed (exit N)" and never reads stderr, because a server or credential helper may put secrets there. No error that reaches `sync()`'s caller says *why* Git failed. Offline/authentication classification must therefore happen **inside `execute`**, while stderr is still in hand, and leave as a typed kind. The stderr text itself still never leaves `execute`.
- **The recovery path covers the interrupted-publication case.** `fetch` (`src/engine.rs:597`) sets `state.main` to the remote `main` before `reconcile_published` runs. `publish_ready` pushes, then sets `published = true` and saves. A crash between the push and the save leaves the commit on the remote while local state still says unpublished, and the next `sync` re-detects it by its trailers. Without that match, `message_base != main` would clear the message and the session would be re-summarized and **published twice**. That is exactly what criterion 1 must rule out.
- **AI failures are swallowed.** `ai::generate` (`src/ai.rs:250`) returns a `Message`, prints `AI summary failed: …` to stderr and falls back. The 429 branch (`src/ai.rs:229`) sees only the status code, never headers or body.
- **Sync failures are recorded only by the runner.** `watch` in `src/main.rs:417` writes `attention = "Synchronization waiting: …"`; the CLI `sync` command just returns the error. `sync()` clears `attention` on success (`src/engine.rs:594`). Engine-raised attentions (`src/engine.rs:433`, `:449`) are separate.
- **State persistence:**
  - `State` goes through `State::map_blobs` (`src/codec.rs:95`) as a struct literal, so a new field fails to compile until it is mapped.
  - No type uses `deny_unknown_fields`, so new `#[serde(default)]` fields load as empty from old state, and an older binary ignores them (downgrade-safe).
  - The state DB runs in WAL mode (`src/storage.rs:14`).

### Steps, in order

1. **Typed Git failures plus `LC_ALL=C`** (`src/git.rs`)
   - `Git::command()` sets `LC_ALL=C` alongside the existing `GIT_TERMINAL_PROMPT`/`GCM_INTERACTIVE`.
   - On a non-zero exit, `execute` reads stderr (bounded, 64 KiB) only to classify it into a new `GitFailure { kind: Offline | Authentication | Rejected | Other, exit }`. The kind is chosen by fixed patterns:
     - offline: `Could not resolve host`, `Failed to connect`, `Connection refused`, `timed out`, `Network is unreachable`, TLS handshake failures;
     - authentication: `Authentication failed`, `could not read Username`, `Permission denied (publickey)`, HTTP 401/403, `Repository not found`;
     - rejected: `[rejected]`, `non-fast-forward`, `failed to push some refs`.

     The 30 s kill becomes `Offline` only for `fetch`/`push`.
   - `Display` stays generic, with a short kind-specific hint and no stderr. `run()` keeps its `Git <op>` context, so callers can `downcast_ref::<GitFailure>()`.
   - *Why first:* the offline state (step 3) cannot be derived without it. It is self-contained and has unit tests on captured `LC_ALL=C` stderr samples.
2. **State fields** (`src/model.rs`, `src/codec.rs`). Add to `State`, each `#[serde(default)]`, all mapped in `map_blobs`:
   - `uploaded: BTreeMap<String, u64>`: own session id → last revision pushed to the device branch. Local-only, never part of `Manifest`.
   - `offline: Option<String>`: the reason, when the last sync failed as `Offline`.
   - `summary_failure: Option<String>`: the last AI-summary failure message (already credential- and body-free).

   *Why before the logic:* the codec mapping must change in lockstep, or state silently fails to persist. A storage round-trip test with an old-format state proves the defaults load.
3. **Engine bookkeeping** (`src/engine.rs`, `src/ai.rs`, `src/main.rs`)
   - `push_manifest`: after a successful push, and also on the early "remote already at this commit" return, set `uploaded[id] = revision` for every own session in the pushed manifest. Drop entries for settled sessions. This touches only `State`, so the manifest, and therefore the next push, is unaffected (criterion 5).
   - New `Workspace::record_sync_failure(&error)`: `Offline` sets `offline` and leaves `attention` alone; everything else sets `attention = "Synchronization waiting: …"` as today. Both the runner (`main.rs:413`) and the CLI `sync` command call it, so `status` after a failed CLI `sync` also shows the cause. On success, `sync()` clears `offline` next to `attention`.
   - `ai::generate` returns `(Message, Option<String>)`, with the failure text instead of an `eprintln`. `summarize_ready` writes `state.summary_failure` and clears it on a successful provider summary. Disabled text sharing is not a failure. This is the persistence half of US00002's criterion 6 (see *Cross-ticket effects*).
4. **Status model in the library** (new `src/status.rs`; `show_status` in `src/main.rs` delegates)
   - `Workspace::status_snapshot() -> Status` (serde). `status` JSON keeps every existing field and adds `reason` at the top level, plus `stage` per session.
   - **Must work on a lock-free `Workspace::inspect`**, as CLI `status` uses today (`src/main.rs:146`). [[ADR-0007 — Persist state transactionally with recovery journals]] relies on status reading the committed SQLite snapshot without the writer operation lock. Therefore `status_snapshot()` takes `&self`, never writes state, and calls nothing that acquires `.mesh/lock` or `network.lock`. Its scan of the live folder stays read-only, as in `show_status`. A test runs `status` while another process holds the operation lock (extending `status_remains_readable_while_an_operation_holds_the_workspace_lock`).
   - **Session stage:**

     | Session | Own, remote configured | Own, local-only workspace | Received from another device |
     |---|---|---|---|
     | published or discarded | `published` / `discarded` | same | same |
     | open | `draft_uploaded` if `uploaded[id] == revision`, else `saved_locally` | `saved_locally` | `draft_uploaded` |
     | closed, not settled | `waiting_for_finalization` if `uploaded[id] == revision`, else `saved_locally` | `waiting_for_finalization` | `waiting_for_finalization` |

     A local-only workspace has no upload step. A received session reached us through the remote, so it is uploaded by definition.
   - **Workspace summary**, decided precedence, first match wins:
     1. `needs_attention`: `attention`, capture error, unresolved conflicts or `summary_failure`;
     2. `offline`: `offline` is set;
     3. `saved_locally`: uncaptured local changes, or any own session at `saved_locally`;
     4. `draft_uploaded`: any own session at that stage;
     5. `waiting_for_finalization`: any own session at that stage;
     6. `synced`.
   - **`reason`** is the cause, picked in the same order: "N unresolved conflicts", the capture error, `attention`, `summary_failure`, then `offline`. It is `null` otherwise.
   - *Why the library and not `main.rs`:* US00002 must move status computation into the crate anyway (its criterion 2). Building it there now avoids writing it twice, and the runner snapshot can reuse it as is.
5. **AI 429 split** (`src/ai.rs`)
   - In the 429 branch, read `Retry-After` (seconds or HTTP date) and up to 64 KiB of body. Parse only `error.code`/`error.type`.
     - `insufficient_quota` / `billing_*` → "Provider quota or billing exhausted: check the provider account".
     - `rate_limit*`, or any `Retry-After` → "Temporary provider rate limit; retry after N s".
     - Otherwise, today's generic hint.
   - The body is never formatted into the message. The existing one-retry loop is unchanged.
   - Independent of steps 1–4, so it can move anywhere.
6. **Regression tests for interrupted publication and renames** (`tests/engine.rs`). These are pure test additions, after step 4 so they can also assert `stage`.
   - **Interrupted publication:**
     1. Finish a session.
     2. Drop the workspace and copy `.mesh/state.sqlite*` (all three WAL files).
     3. `sync` until it publishes.
     4. Drop the workspace, restore the copied files, reopen.

     Now the commit is on the remote but local state says unpublished: the exact crash window. Then `sync` again and assert: `published`, exactly one commit with that `Mesh-Session` trailer on `main`, and file content intact.
   - **Rename:** Alice renames `Notizen für März.md` → `Archiv/Notizen für März.md` and deletes `old.md`. Bob converges: new path present, old path and `old.md` absent, and both finalize on `main`.
7. **Status, offline and locale tests**
   - The offline, authentication and locale tests run in **`tests/cli.rs` through the binary**, because they need per-process environment variables. Setting `LANG` or Git config in the shared test process would race with parallel tests.
   - **Offline:** remote `https://127.0.0.1:9/x.git` (connection refused). After `sync` fails, `status` shows `offline` with a reason, and capture still records a new save.
   - **Authentication:** a mock HTTP server (pattern from `tests/provider.rs`) answers the Git request with 401. Credential helpers are disabled for that child (`GIT_CONFIG_NOSYSTEM=1`, empty `GIT_CONFIG_GLOBAL`). Expect `needs_attention`.
   - **Locale:** the same two cases with `LANG=de_DE.UTF-8`/`LC_MESSAGES=de_DE.UTF-8`.
   - **Plumbing:** a unit test asserts `Git::command()` carries `LC_ALL=C`. That check is deterministic even on hosts without German Git translations, where the locale test would pass trivially.
   - **Engine:**
     - each session stage and summary value (criteria 3, 4 and 8);
     - criterion 5: a second `sync` with no edits leaves the device-branch commit unchanged, and the manifest decoded from the device branch contains no `uploaded` data;
     - an old-format state loads with an empty `uploaded`.
   - **Update** `tests/cli.rs:250` for the new value set.
8. **README** (criterion 11), last, so it documents the behavior as built:
   - extend the `status` paragraph (line 36) with summary values, precedence, stages and `reason`;
   - a *Storage growth* subsection: `.mesh/state.sqlite` and its blob table, `.mesh/objects.git`, `.mesh/recovery/`, device branches and `main`; nothing is pruned in this preview;
   - a *Recovery procedures* subsection with one procedure per `needs_attention` cause and for `offline`, the pending-capture fallback, and restoring from `.mesh/recovery/<id>/`.
9. **Gates:** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test`. Engine tests take several minutes on Windows.

### Review points (decided in this plan; flag them if you disagree)

- **An AI-summary failure counts as `needs_attention`.** It does not block publication, because the factual fallback is used, but it is actionable (fix key, quota or endpoint). Without this, criterion 7's "reason from the persisted AI-summary failure" would never surface. The alternative is to show it as `reason` without raising the summary state.
- **A rejected push stays `needs_attention`**, per criterion 6, although a non-fast-forward publication race is normally resolved by the next cycle. It can therefore flash for one cycle, which is unchanged from today.
- **An unknown Git failure falls back to `needs_attention`.** Misclassifying an unknown failure as harmless `offline` would hide something the user must act on.

### External changes and migration

- No change to the remote format: device-branch manifests and `main` trailers are untouched, and clients on 0.2.4 and this version interoperate.
- Local state gains three defaulted fields. There is no migration step, and downgrading works because unknown fields are ignored.
- The `status` value set changes (`waiting_to_sync` is removed), as decided. `tests/cli.rs` is the only consumer.

### Cross-ticket effects

- **[[Story - US00002 - Local engine control interface for the tray host|US00002]]:** this plan delivers the persistence half of its criterion 6 (`summary_failure`) and the library-side status computation its criterion 2 needs. US00002 then only moves `watch` and `summarize_ready` and publishes `status_snapshot()`. Record this in US00002 via `/update-ticket` once this ticket's code lands.
- **[[Story - US00001 - Tray icon and status window host|US00001]]:** its icon-state question can map 1:1 onto the summary values above.

### Wiki pages this makes wrong

- [[Dispatch Mesh requirements and architecture]] line 106 lists four desktop states ("saved locally, waiting to sync, synced, and needs attention"). Kai should annotate it with the six-state model when this ticket is done.
- [[Dispatch Mesh milestone plan]]: no change, since its six states are what this implements.

### Risks and rollback

- **Brittle classification patterns.** Messages differ between Git versions and TLS backends (schannel vs. OpenSSL). Mitigations: `LC_ALL=C`, unit samples for both backends, and the safe fallback to `needs_attention`. TASK00002 (a library backend) eventually replaces this with typed errors.
- **The WAL copy in the interruption test.** If restoring `state.sqlite*` proves flaky on Windows because of file handles, the fallback is a test-only fault hook behind a Cargo feature. Prefer the copy, since it needs no production code.
- **Rollback:** revert the commit. Older binaries read the new state, and the remote format is unchanged.

### Size

Roughly 3–5 focused days: steps 1–4 about 60%, tests about 30%, docs 10%. This is an order-of-magnitude estimate, not a commitment.

### ADRs

- **Status model:** recorded as [[ADR-0010 — Report workspace status and session stages separately]] (proposed). On 2026-11-05 it was amended with this plan's three review-point rules (AI-summary failure, rejected push and unknown Git failure all go to `needs_attention`) and the offline definition. When this ticket reaches `Done`, set it to `accepted`, amending it first if the built model deviates.
- **Governing ADRs for this plan:**
  - [[ADR-0001 — Synchronize drafts separately from permanent history]]: recovery from interrupted publication is a stated consequence, which step 6 tests.
  - [[ADR-0005 — Make AI summaries configurable and opt-in]]: bounded retries, and no alternate provider; step 5 keeps both.
  - [[ADR-0007 — Persist state transactionally with recovery journals]]: lock-free status (step 4).
  - [[ADR-0009 — Embed the engine with a runner handle and published status]]: the snapshot consumer.
- **Still a candidate, not yet extracted — Git failure classification:** stderr is classified inside `execute` and never echoed, with `LC_ALL=C` on every Git process, and unknown failures go to `needs_attention`. Durable until TASK00002 replaces the CLI backend, which would supersede it.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

<!-- GUIDE: Written by /test-plan — `- [ ]` manual checks only, excluding what the
     automated suites already assert. Keep `open_tests:` in sync as items are ticked. -->

## As-built notes

<!-- GUIDE: Where the implementation deviated from the plan, and why. Dated entries. -->

## Follow-ups

### 2026-11-05 — Status ADR recorded

Kai requested ADR extraction. [[ADR-0010 — Report workspace status and session stages separately]] records the agreed summary precedence, session stages, reason field and local-only upload record. It is `proposed` under the ADR template until built. [[ADR-0009 — Embed the engine with a runner handle and published status]] covers its host interface. The implementation plan, review gates and tests remain outstanding; this entry is not plan sign-off. The Git backend ADR remains separate under TASK00002; the Git error-classification ADR candidate in this plan has not been extracted here.

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[Dispatch Mesh milestone plan]] — 0.2.x engine stabilization (work items and exit criteria)
- [[github-workflow-findings-2026-11-04]] — live verification of 0.2.4
- [[Story - US00001 - Tray icon and status window host|US00001]], [[Story - US00002 - Local engine control interface for the tray host|US00002]]
- [[Task - TASK00002 - Choose the Git backend before bundling Git|TASK00002]] — longer-term Git library backend; this ticket carries the interim `LC_ALL=C` fix
- Code: `src/main.rs` (`show_status`), `src/engine.rs` (`reconcile_published`), `src/ai.rs` (`http_failure`), `README.md` (*Conflicts and recovery*, limits)

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
