---
id: TASK00001
type: task
status: Released
priority: high
rank: 1024
version_target: v0.2.5
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests: 0
open_findings: 0
discussion:
updated: 2026-11-05
completed: 2026-11-05
frozen: 2026-11-05
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

Manual checks for the six-state status, offline/attention classification, the HTTP 429 split and the new README sections. They apply to branch `task00001-stabilization-gaps` at `c079e9d`, which passed a clean code review. *(2026-11-05: now `41a543b`, after two CI fixes and the CI locale step on top; see As-built notes.)* The automated suites already assert everything else (see the report below this list). Live checks use a **disposable folder** and the designated test repository `https://github.com/kaimys/dispatch-mesh-test.git` (per `AGENTS.md`), never the wiki remote.

- [x] **Kai:** Push the branch to `origin` (`git push -u origin task00001-stabilization-gaps`). In the repository's **Actions** tab, open the *Engine checks* run for `c079e9d`. **Expected:** both the `windows-latest` and `ubuntu-latest` jobs are green. Only Windows has been verified so far; Linux has never run this code.
- [x] **Kai:** Real network loss against GitHub.
  1. In PowerShell: `dispatch-mesh --workspace C:\MeshTask1Net init --name Kai --email kai@eightnine.de --remote https://github.com/kaimys/dispatch-mesh-test.git`, then `dispatch-mesh --workspace C:\MeshTask1Net run` in its own window.
  2. Turn Wi-Fi/Ethernet off, save `C:\MeshTask1Net\net.md` with any text, wait 15 s, then run `dispatch-mesh --workspace C:\MeshTask1Net status` in a second window.

  **Expected:** `"status": "offline"`; `reason` starts with `Git fetch: Remote unreachable`; `attention` is `null`; the session listing `net.md` is at stage `saved_locally`. Turn the network back on and wait 15 s. **Expected:** `status` is `draft_uploaded`, `offline` and `reason` are `null`. Stop `run` with Ctrl+C.
  - *Result 2026-11-05 (Kai, build `07903c4`): **passed.***
    - **While offline:** `status: "offline"`, `reason` "Git fetch: Remote unreachable (exit 128) …", `attention: null`, `net.md` session at `saved_locally`.
    - **Waiting more than 5 minutes after saving** let the session close itself (`closed: true`, still `saved_locally`, as the model specifies). So on reconnect Mesh uploaded and published straight away: `status: "synced"`, stage `published`, and `offline`/`reason`/`attention` all `null`.
    - **On GitHub:** `main` on `kaimys/dispatch-mesh-test` contains `net.md`, with trailer `Mesh-Session: 7cc213cb-3056-4cf8-a986-51022f94f597`.
    - **Not observed:** the expected `draft_uploaded` only appears when checked within 5 minutes of saving. The plan should have said so. That stage is asserted by `status_follows_a_session_from_local_save_to_publication`.
- [x] **Kai:** Real refused login through Git Credential Manager.
  1. Open *Windows Credential Manager → Windows Credentials* and note the `git:https://github.com` entry, which you will need to restore.
  2. Remove that entry.
  3. In the `C:\MeshTask1Net` workspace from the previous item, save another file and run `dispatch-mesh --workspace C:\MeshTask1Net sync`.

  **Expected:** the command fails without a sign-in window, because Mesh sets `GCM_INTERACTIVE=never`. `status` then shows `"status": "needs_attention"`, a `reason` starting with `Synchronization waiting: Git fetch: Remote refused access`, and `"offline": null`. Neither field contains a token, a URL with credentials, or Git's raw output. Sign in again (`git ls-remote https://github.com/kaimys/dispatch-mesh-test.git` triggers GCM), then run `sync` again. **Expected:** `attention` and `reason` are `null`.
  - *Result 2026-11-05 (Kai, build `07903c4`): **passed.***
    - **Credential removed:** `git:https://github.com` (via `cmdkey /delete`). `sync` failed and **no sign-in window or browser opened** (confirmed by Kai).
    - **Status:** `status: "needs_attention"`, `reason` "Synchronization waiting: Git push: Remote refused access (exit 128) …", `offline: null`, and no token, URL credentials or raw Git output anywhere. It reads *push*, not fetch, because `kaimys/dispatch-mesh-test` is public, so the fetch needs no credentials.
    - **Re-sign-in, corrected procedure:** `git ls-remote` on a public repository never prompts. A `git push --dry-run` to the test repository did, and restored the credential without creating a branch (`mesh-login-check` absent on GitHub).
    - **Recovery:** the next `sync` succeeded with `attention`/`reason`/`offline` all `null`. `main` = `7c032b3`, adding `auth.md` with trailer `Mesh-Session: 07173d39-699b-4993-b648-abe7f5433270`, the session that was active during the refusal.
- [x] **Kai:** Locale independence on a host whose Git actually translates. This Windows machine's Git 2.35.1 ships no message catalogs, so the automated locale test cannot fail here. Use a Linux machine or WSL distribution with Git, its German messages (e.g. Debian/Ubuntu `git` plus `locales`, with `de_DE.UTF-8` generated) and Rust.
  1. Confirm the precondition: `LANG=de_DE.UTF-8 LC_ALL=de_DE.UTF-8 git -C /tmp status` prints a **German** message (for example `fatal: Kein Git-Repository …`). If it prints English, this check is not meaningful on that host.
  2. In a checkout of `c079e9d`, run `cargo test --test cli failure_classification_does_not_depend_on_the_user_locale`.

  **Expected:** the test passes.
  - *Result 2026-11-05 (CI, commit `41a543b`; method chosen by Kai): **passed.***
    - **No local Linux:** this machine has no WSL distribution, so the check runs as a new Ubuntu CI step instead.
    - **Precondition proven:** in GitHub Actions run 42033691572, the step generated `de_DE.UTF-8` and showed that Git really translates: `Git under de_DE.UTF-8: Schwerwiegend: Kein Git-Repository (oder irgendeines der Elternverzeichnisse): .git`.
    - **Test:** `failure_classification_does_not_depend_on_the_user_locale ... ok`.
    - **Ongoing:** the step now runs on every push, and fails if the German catalog is ever missing, so it cannot pass trivially.
- [x] **Kai:** Read the new README sections on the pushed branch on GitHub: the `status` table under *Build and run*, and *Storage growth* and *Recovery procedures* under *Conflicts and recovery*. **Expected:** both tables render as tables, and every recovery step is one you would give a user as written: correct commands, correct order, and no step that could discard `.mesh` data. Note any wording to change as a dated entry under *As-built notes*.

**Not in this plan, and why.**
- **Covered by the automated suites (69 tests, green on Windows; clean review at `c079e9d`):**
  - every summary value and session stage, including `discarded` and reopening;
  - precedence and `reason`;
  - the local-only upload record with no extra push;
  - interrupted publication recovered exactly once (mutation-checked);
  - two-client rename and deletion with a non-ASCII, spaced path;
  - offline vs. refused login via the binary, with an isolated Git config;
  - failure recording under lock contention (checked against the pre-fix code);
  - `LC_ALL=C` on every Git process;
  - the 429 quota, rate-limit and generic cases;
  - all three HTTP-date forms, the RFC 850 50-year cutoff and impossible dates;
  - AI-failure persistence through the real summary path;
  - lock-free `status`.
- **Verified by the plan's author on 2026-11-05 with the built binary:**
  - the background `run` loop, which has no automated offline test, recorded `offline` with reason `Git fetch: Remote unreachable …` and `attention: null`, and kept capturing (`a.md` at `saved_locally`) against an unreachable HTTPS remote;
  - the status lifecycle `synced` → `saved_locally` → `draft_uploaded` → `synced`/`published` (recorded in the first development pass).
- **Deliberately omitted:** a real provider in a quota-exhausted state, which cannot be produced on demand; the quota, rate-limit and body-redaction paths are asserted against mock responses. A real crash in the publication window was also left out: its outcome is asserted by restoring pre-publication state, and timing a real kill into that window is not reproducible.

## As-built notes

### 2026-11-05 — First development pass (branch `task00001-stabilization-gaps`)

The plan was followed in its step order. These are the deviations and things a reviewer should know:

- **Step 3, failure recording.** Instead of calling `record_sync_failure` from two places, a shared `Workspace::sync_and_record(root)` opens, syncs and, on failure, reopens and records. Both the runner (`src/main.rs`) and CLI `sync` use it, which removed the runner's inline copy. An `offline` result replaces a previous *sync-failure* attention (text prefixed `Synchronization waiting: `) but keeps engine-raised attention (pending capture). A non-offline failure clears `offline`.
- **Step 3, AI failures.** `ai::generate` keeps its signature as a wrapper. The new `ai::generate_with_failure` returns `(Message, Option<String>)`. This avoided changing the three `tests/provider.rs` call sites. `summarize_ready` saves `summary_failure` only when it changed.
- **Step 3, upload record.** `push_manifest` captures the own sessions' revisions before encoding and records them only after the push returns. `network_call` reloads state from disk, so recording earlier would be lost. The record keeps only unsettled sessions.
- **Step 4, status JSON.** As well as `reason` and per-session `stage`, the JSON now carries the raw `offline` and `summary_failure` fields next to `attention`. `Identity` gained `PartialEq, Eq` so `Status` can be compared, which US00002 needs for "publish only when changed".
- **Step 5, `Retry-After`.** Only the delta-seconds form becomes "retry after N s". An HTTP-date value yields "temporary rate limit; retry later". Parsing HTTP dates would need a new dependency for little gain.
- **Step 7, `tests/cli.rs:250`.** No change was needed: an empty workspace still reports `synced` under the new model. The existing lock-held status test (`status_remains_readable_while_an_operation_holds_the_workspace_lock`) already covers the lock-free requirement, now exercising `status_snapshot()`.
- **Step 7, offline and authentication CLI tests.** These use the test-only loopback HTTP remote override that the slow-remote test already uses, because users may only configure HTTPS, SSH or local remotes. Each child gets an isolated Git config (`GIT_CONFIG_NOSYSTEM=1`, empty `GIT_CONFIG_GLOBAL`), so this machine's Git Credential Manager cannot answer the 401.
- **Step 7, locale test: limited evidence on this host.** Git for Windows 2.35.1 here ships no message catalogs: even `not a git repository` stays English under `LANG=de_DE.UTF-8`. The German-locale CLI test therefore passes trivially on this machine. The guarantee rests on the unit test `every_git_process_runs_with_untranslated_messages`, which asserts `LC_ALL=C` and a removed `LANGUAGE` on every `Git::command()`. The CLI test becomes a real check on a host with German Git translations (e.g. Linux with the `de` catalog and locale installed).
- **Step 6, interrupted publication.** The test restores a pre-publication copy of `.mesh/state.sqlite*`, as planned; no production fault hook was needed. It was **mutation-checked**: with `reconcile_published()` disabled in `sync()`, the test fails.
- **Wiki:** [[Dispatch Mesh requirements and architecture]] line 110 annotated with the six-state model.
- **Gates (2026-11-05, Windows 11, Git 2.35.1):** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` green. 63 tests: 5 lib unit, 9 CLI, 26 engine, 15 provider, 8 storage/transport, of which 19 are new. The Ubuntu CI leg has not been run yet; it runs on push.
- **Manual check with the built binary:** a scratch workspace with a local bare remote showed `synced` → `saved_locally` (uncaptured edit) → `draft_uploaded` (after `sync`) → `synced` with the session `published` (after `session finish` + `sync`). With no provider configured, CLI `sync` summarizes and publishes in one invocation, so `waiting_for_finalization` is not observable there; the engine test `status_follows_a_session_from_local_save_to_publication` asserts it.
- **Code review:** not yet run. Next step is `/code-review TASK00001` in a fresh session.

### 2026-11-05 — Code-review hand-off blocked at `08c0b02`

Independent review found three blocking findings: **B1**, failed-sync status can be lost during operation-lock contention (criteria 6 and 7); **B2**, an HTTP-date `Retry-After` loses its retry delay (criterion 9 and plan step 5); **B3**, the `discarded` status stage has no automated assertion (criterion 8). The card remains `In progress`, unfrozen. `/test-plan` must wait for repairs and a fresh review. See the review and repair steps below; implementation code was not changed in this session.

### 2026-11-05 — Fix round for the review of `08c0b02`

All four findings of the 2026-11-05 review are addressed. `open_findings` is cleared to empty because the code under that count changed; only the next review may write a number.

- **B1 (failure recorded under lock contention).**
  - `sync_and_record` now records the failure on the workspace that ran the sync, while it still owns the operation lock. It no longer drops the workspace and makes a single reopen attempt.
  - If a network step lost the lock to a concurrent operation, the new `persist_sync_failure` waits up to 30 s to re-acquire it instead of skipping the record. It then reloads durable state and records against that, discarding anything the failed sync left half-applied in memory, exactly as the old reopen did.
  - Root cause found while fixing: `network_call` replaced the original Git error with `WorkspaceBusy` when re-acquiring the lock timed out. Since `WorkspaceBusy` is deliberately not recorded, the cause vanished. It now returns the original failure in that case. The lock helper is shared as `acquire_lock`.
  - If even recording fails, the returned error carries both causes.
  - Regression: `a_failed_sync_records_its_cause_even_when_another_operation_holds_the_lock`. A loopback remote signals when the network step starts; a competing open takes the lock and holds it for 7 s, past the 5 s re-acquire window. The test asserts a non-busy error, a durable `needs_attention` reason after reopening, and clearing on the next successful sync. **Checked against `08c0b02`'s `engine.rs`:** there the test fails with "Another Mesh operation is using this workspace", which is the reviewed defect.
- **B2 (HTTP-date `Retry-After`).**
  - `ai::retry_after_seconds` accepts delta-seconds and IMF-fixdate HTTP dates, the form RFC 9110 requires senders to use. A past date gives 0 s, never a negative delay.
  - The obsolete RFC 850 and asctime forms, and malformed values, keep the generic "retry later". No new dependency was added: `httpdate` is not in the tree, so a ~35-line parser was written.
  - Tests:
    - a deterministic unit test, with reference times computed independently in Python, covering the RFC example, a leap day, an elapsed date and malformed forms;
    - two mock-provider cases, a future and an elapsed date, both asserting that the body is not echoed.

  This supersedes the earlier as-built note on `Retry-After`.
- **B3 (`discarded` stage).** `identical_concurrent_changes_do_not_add_an_empty_session_commit` now asserts `Stage::Discarded`, serialized `"stage": "discarded"` and a `synced` summary, before and after reopening. The existing snapshot and no-empty-commit assertions are kept, and `main` is unchanged after the restart.
- **N1 (AI-failure persistence through the real path).** `a_failed_summary_is_persisted_for_status_and_cleared_by_the_next_success` drives the binary (`session finish`, `sync`, `status`) against a mock provider. On a 401 the session still publishes, and `status` in a new process shows `needs_attention` with a sanitized `AI summary failed: … HTTP 401` reason that contains no response body. A later successful summary returns the workspace to `synced`.
- **Gates (2026-11-05, Windows 11, Git 2.35.1):** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` green on commit `7ac5316`. 68 tests: 5 lib unit, 9 CLI, 27 engine, 19 provider, 8 storage/transport. The Ubuntu CI leg has not been run (not pushed).
- **Next:** a fresh `/code-review TASK00001` against `7ac5316`. Only a clean review hands off to `/test-plan`.

### 2026-11-05 — Re-review hand-off blocked at `7ac5316`

The re-review closes B1, B3 and N1 from `08c0b02`. B2 is partially repaired: IMF-fixdate works, but two other valid HTTP-date forms still lose the retry delay, so criterion 9 remains unmet. One new non-blocking issue concerns accepting impossible calendar dates. All 68 tests and the other mechanical gates passed independently on Windows. Keep the card `In progress` and unfrozen; the next `/develop` session repairs the remaining date parsing and tests before another review. See the dated review below.

### 2026-11-05 — Fix round for the re-review of `7ac5316`

`open_findings` is cleared to empty because the code under that count changed; only the next review may write a number.

- **B2 continued (all HTTP-date forms).** `ai::retry_after_seconds` now accepts all three forms RFC 9110 §5.6.7 requires recipients to accept: IMF-fixdate, RFC 850 and asctime. For RFC 850's two-digit year it applies the §5.6.7 rule: a year more than 50 years ahead of the current year means the most recent past year with the same last two digits. The earlier as-built restriction to IMF-fixdate is withdrawn.
  - The deterministic test now requires the three forms of the RFC example instant to yield the same future (60 s) and elapsed (0 s) delays. It also checks the two-digit-year boundary against an independently computed reference: `-76` stays 2076 and `-80` becomes 1980 when the current year is 2026.
  - A new mock-provider test sends RFC 850 and asctime headers and asserts the numeric delay, with no body echo.
- **N2 (impossible calendar dates).** Day-of-month is validated against month length with Gregorian leap-year rules before conversion, so 31 Feb, 30 Feb 2028, 29 Feb 2027, 31 Apr and day 00 all fall back to "retry later" in every form. Valid leap days (IMF-fixdate and asctime) still parse.
- **Malformed-form coverage tightened:** wrong digit counts per form, an unknown weekday, a non-GMT zone and a trailing zone on asctime are rejected.
- **Gates (2026-11-05, Windows 11, Git 2.35.1):** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` green on commit `291c263`. 69 tests: 5 lib unit, 9 CLI, 27 engine, 20 provider, 8 storage/transport. The Ubuntu CI leg has not been run (not pushed).
- **Next:** a fresh `/code-review TASK00001` against `291c263`.

### 2026-11-05 — Re-review hand-off blocked at `291c263`

B2's format support and N2's impossible-date handling are now repaired. Independent review found one new edge case in the added RFC 850 year handling: **B4**, the 50-year cutoff compares only the year, not the complete timestamp. This can report a roughly 50-year retry delay for a timestamp the RFC requires interpreting in the past. Criterion 9 remains unmet for that boundary input. All 69 tests and the other mechanical gates passed independently on Windows. The ticket stays `In progress`, unfrozen, pending the B4 repair and another review.

### 2026-11-05 — Fix round for the re-review of `291c263`

`open_findings` is cleared to empty because the code under that count changed; only the next review may write a number.

- **B4 (RFC 850 cutoff on the complete timestamp).** The two-digit year is now resolved only after month, day and time are parsed. `rfc850_year` compares the complete candidate timestamp in the current century against the instant exactly 50 calendar years after now. Exactly at the cutoff stays in the future; any later instant means the previous century. The final interpreted date is then validated, so impossible dates are still rejected. The year-only `full_year` helper is removed; a small `Year` enum marks the RFC 850 case, and `civil_from_days` (inverse of `days_from_civil`) provides now's calendar date.
  - **Boundary chosen for leap days:** when now is 29 Feb, the 50th anniversary does not exist, and the cutoff becomes the following instant, 1 Mar 00:00:00. This is covered by a test.
  - **New deterministic cases** (reference values computed independently in Python), with now = 2026-11-05T13:00:00Z: one second before the cutoff, exactly at it, one second after (→ 1976, 0 s; the reviewer's reproduction), and 31-Dec-76 (→ 1976, 0 s), plus the leap-day cutoff pair. The existing early-2076 and 2080 cases are kept.
  - **Checked against `291c263`'s `ai.rs`:** there the test fails at "one second past the cutoff means 1976", the reviewed defect.
- **Gates (2026-11-05, Windows 11, Git 2.35.1):** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` green on commit `c079e9d`. 69 tests (unchanged count; the boundary cases extend `retry_after_accepts_delta_seconds_and_http_dates`). The Ubuntu CI leg has not been run (not pushed).
- **Next:** a fresh `/code-review TASK00001` against `c079e9d`.

### 2026-11-05 — CI failure on Linux, fixed in `af1d3a0` (test only)

The first push ran GitHub Actions *Engine checks* (run 42018003838) on `c079e9d`. **`ubuntu-latest` failed** in `a_failed_sync_records_its_cause_even_when_another_operation_holds_the_lock`, with `left: Offline, right: NeedsAttention`. **`windows-latest` did not fail**: it was cancelled by the matrix's default fail-fast when the Ubuntu job failed.

- **Cause: the test, not the product.** The test's fake remote stalled and then closed the connection. Curl reports that as "Empty reply from server" on Windows (unclassified, so `needs_attention`) but as "Connection reset by peer" on Linux, which ADR-0010 correctly classifies as `offline`. The assertion therefore depended on the platform's curl wording.
- **Fix:** after the stall, the fake remote now answers `HTTP/1.1 500`. Git reports that as an unclassified failure on every platform, so `needs_attention` deterministically. The test's purpose is unchanged: the cause must be durable under lock contention.
- **Still guards B1:** run against `08c0b02`'s `engine.rs`, it fails. No production code changed.
- **Because the build under the clean review changed,** `open_findings` is cleared to empty, per the invariants. `af1d3a0` needs a short re-review of this test-only diff before the card can leave Review. The manual test-plan items now apply to `af1d3a0`.

### 2026-11-05 — Second Linux CI failure: spurious `WorkspaceBusy`, fixed in `07903c4`

CI run 42018932901 on `af1d3a0` failed again on `ubuntu-latest` (attempt 1: three engine tests; a rerun of the same commit: two **different** tests, including `journal_recovers_a_crash_after_displacing_the_old_file` and `offline_sync_keeps_durable_local_capture`, which predate this branch). Every failure was `WorkspaceBusy` ("Another Mesh operation is using this workspace") on a workspace that only that test uses, at a non-retrying lock acquisition: `Workspace::open` right after a drop, or the network lock at the start of `sync`. `main` was green in all six earlier CI runs.

- **Cause (inferred from the evidence, not reproduced locally; no Linux environment is available on this machine):** the workspace locks are `flock` locks, which belong to the open file description. When any thread spawns a child (Git), the child holds a copy of every descriptor until it execs, so a lock just released can stay held for a moment. An immediate re-lock then fails. This branch adds many tests that drop and reopen while other test threads spawn Git, which raises the odds. The product has the same exposure: the runner spawns Git on one thread while capturing on another. Windows does not let child processes inherit these handles, which is why it never appeared locally.
- **Fix (production code):** `operation_lock` retries for up to 500 ms (`LOCK_SETTLE`) before reporting `WorkspaceBusy`. Real contention from another operation lasts far longer and is still reported; no test depends on an immediate busy. Local gates green on Windows (69 tests).
- **CI:** `fail-fast: false` in `.github/workflows/ci.yml`. Until now every Ubuntu failure cancelled the Windows job, so CI never reported a Windows result for this branch.
- **Review impact:** `open_findings` stays empty. The re-review must cover `af1d3a0` (test) and `07903c4` (lock retry and CI config).
- **CI result:** run 42020325879 on `07903c4` was green on **both** `ubuntu-latest` and `windows-latest` in **three of three attempts** (the original run plus two reruns of the same commit). This is evidence against the race, not proof: it was random before.

### 2026-11-05 — CI locale step added in `41a543b`

- **New step:** test-plan item 4 is automated in `.github/workflows/ci.yml` as an Ubuntu-only step, *Locale independence with a translating Git*. It runs `locale-gen de_DE.UTF-8`, checks that Git prints German (`Kein Git-Repository`), failing with an error annotation if not, and then runs `failure_classification_does_not_depend_on_the_user_locale`.
- **Result:** run 42033691572 on `41a543b` was green on `ubuntu-latest` (locale step passed) and `windows-latest` (step skipped). It also completed all four manual test-plan items: `open_tests: 0`.
- **Review impact:** `open_findings` stays empty. The pending re-review covers `af1d3a0` (test), `07903c4` (lock retry, `fail-fast: false`) and `41a543b` (locale step).

### 2026-11-05 — No re-review of the post-review commits (Kai's decision)

Kai decided that one code review is enough: the clean review of `c079e9d` stands, and `af1d3a0`, `07903c4` and `41a543b` are not re-reviewed. Recorded so the history is explicit:
- `07903c4` changes production code (`operation_lock` retries up to 500 ms) after the last review. It is covered by green CI on both platforms in three attempts and by the full local suite, not by an independent review.
- `open_findings` stays empty, because only `/code-review` may write a count. The gate out of Review is `open_tests: 0`, which is met.

### 2026-11-05 — Merged into `main`

`task00001-stabilization-gaps` was fast-forwarded into `main` (`3c2d513..41a543b`) and pushed on Kai's request. CI run 42035665583 on `main` at `41a543b` was green on `ubuntu-latest` (including the locale step) and on `windows-latest`.

## Code review

### 2026-11-05 — branch task00001-stabilization-gaps at 08c0b02

**Scope and independence.** Fresh reviewer session; this session authored none of the implementation. Reviewed the complete `main...HEAD` diff (one ticket-scoped commit; 12 files), the whole ticket including revisions and as-built notes, README, workflow invariants and governing ADR-0001/0005/0007/0009/0010. Working tree was clean. No open PR was found for this branch, so there were no PR reviews to fold in.

**Gates.** Independently ran `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`: all passed on this Windows host. All 63 tests passed: 5 library, 9 CLI, 26 engine, 15 provider, 8 storage/transport. Engine tests took 173.56 seconds. `git diff --check main...HEAD` also passed. No live GitHub/provider tests or Linux/macOS verification were performed. As already recorded by the author, this host cannot prove behavior against installed German Git translations; the command-environment assertion is deterministic.

**Verdict: changes required.** **3 blocking findings, 1 non-blocking finding.** The ticket cannot leave review as built and cannot proceed to `/test-plan`. Keep `status: In progress`, `frozen:` unset and `open_tests:` unchanged.

#### Findings, in severity order

1. **B1 — [P2, blocking] Persist the sync outcome before releasing its operation lock.** `src/engine.rs:768–774`, especially the conditional reopen at line 771. After `sync()` returns an error, `sync_and_record()` drops the workspace and then tries to reopen it once. If a concurrent capture/finish operation obtains `.mesh/lock` in that interval, `Workspace::open` returns `WorkspaceBusy`, the `if let Ok` silently skips recording, and the original Git error is returned without any durable cause. A loopback connection-refused probe reproduced this: the returned error was `Git fetch: Remote unreachable (exit 128)…`, while the subsequent snapshot was `Synced` with `reason: None`. This violates criteria **6 and 7**, and ADR-0010's last-sync-failure rule. **Fix:** record and save the error on the existing workspace while it still owns the operation lock; if a failure path has already lost that lock, retain the outcome and retry acquisition rather than dropping it. Preserve the `WorkspaceBusy` exclusion. Add a regression that overlaps failure completion with capture contention and verifies the persisted offline/attention reason after reopening, plus clearing on a successful sync.

   **Evidence:** ignored local review harness `target/review-task00001/probe.rs`; retained outputs `target/review-task00001/probe-offline-output.txt` and `target/review-task00001/probe-output.txt`. It uses disposable workspaces and a competing lock holder; it does not modify the code under review. Both connection-refused and unclassified Git failures were observed returning with neither durable failure field set.

2. **B2 — [P2, blocking] Report the retry delay for HTTP-date `Retry-After`.** `src/ai.rs:171–176`. Only unsigned delta-seconds are parsed. A valid header such as `Retry-After: Thu, 05 Nov 2026 13:00:00 GMT` produces only `temporary rate limit; retry later`, discarding the supplied retry time. The as-built note acknowledges this deviation, but criterion **9** requires the retry delay when given and plan step **5** explicitly includes HTTP dates; documenting the deviation does not satisfy that contract. **Fix:** parse both forms, calculate a non-negative delay for a valid HTTP date, and retain a safe generic fallback for malformed dates. Add mock-provider cases for a future date and an elapsed date, without echoing response bodies. No product decision or `/update-ticket` is needed to implement the agreed behavior; weakening it would require a requester decision and an explicit contract update.

3. **B3 — [P2, blocking] Assert the `discarded` session stage in automation.** `tests/engine.rs:635–683` covers the local-save-to-publication progression, but no test anywhere asserts `Stage::Discarded` or a serialized `stage: "discarded"`. The existing same-change regression at `tests/engine.rs:487` checks only the domain `discarded` flag. Returning `published` or `saved_locally` for a discarded session would therefore leave the suite green. Criterion **8** explicitly requires every session stage to be covered. **Fix:** extend a real discard scenario, such as `identical_concurrent_changes_do_not_add_an_empty_session_commit`, to assert the status stage and a `synced` summary, including after reopen. Keep the assertions that original snapshots survive and no empty commit is added. The code's discarded branch appears correct; the blocking failure here is the required regression coverage.

4. **N1 — [P3, non-blocking] Exercise AI-failure persistence through the actual summary path.** `tests/engine.rs:791–799` directly assigns `summary_failure`, so it proves presentation but never executes the new persistence wiring at `src/main.rs:247–252`. Add a CLI/mock-provider regression where a failed summary still publishes the factual fallback, `status` after restart retains the sanitized failure, and a later successful or metadata-only summary clears it. This protects the new seam between summarization, durable state and the status consumer; no current failure of that wiring was demonstrated.

#### Acceptance criteria and plan assessment

| Criterion | Assessment at this commit |
|---|---|
| 1 — interrupted publication | Pass: restored pre-publication SQLite state is recognized after reopening; exactly one session trailer and intact content are asserted. The author's mutation check is recorded as author evidence, not rerun here. |
| 2 — rename plus separate deletion | Pass: two clients converge on the non-ASCII, spaced destination and remote `main` contains only the destination file. |
| 3 — summary values and precedence | Implementation follows the agreed order; individual summary values are covered. B1 can leave the durable inputs stale after a failed sync. |
| 4 — session stages | Implementation provides all five stages, including distinct local-only and received-session behavior. |
| 5 — local-only upload bookkeeping | Pass: exact pushed revisions are captured before network reload and recorded afterward; manifest/unchanged-ref regression passes; defaulted fields preserve old-state deserialization. |
| 6 — offline vs. attention and recovery | **Fail B1:** a failed sync can leave no recorded cause under contention. Uncontended CLI offline/authentication cases pass. Success clears both fields in code. |
| 6a — untranslated Git processes | Pass: all production Git construction uses `Git::command`, setting `LC_ALL=C` and removing `LANGUAGE`; environment and locale tests pass, subject to the host limitation above. |
| 7 — actionable, sanitized reason | **Fail B1:** the reason can be absent after a failed sync. Known Git/provider diagnostics otherwise use fixed text rather than stderr or response bodies. |
| 8 — automated summary/stage coverage | **Fail B3:** no assertion for the discarded status stage. |
| 9 — HTTP 429 classification and delay | **Fail B2:** quota/rate/generic branches work, but HTTP-date retry delay is omitted. |
| 10 — three mock-provider cases, no body leak | Pass for the three existing cases; B2 adds the missing date variant. |
| 11 — status, storage and recovery documentation | Pass for the specified sections and principal recovery cases; the README's retry-delay claim will become accurate for HTTP dates once B2 is fixed. |
| 12 — mechanical gates | Pass independently on Windows; 63 tests green. |

The principal implementation deviations are dated in the as-built notes: shared failure-recording helper, wrapper API for AI generation, post-network upload bookkeeping, raw status fields, unchanged existing `synced` CLI assertion and locale limitations. The HTTP-date deviation is recorded but still fails the contract. No unrelated branch changes were found. The proposed scan-free status core requested by US00002 is downstream planning work, not an acceptance criterion of this ticket.

#### What is good and should stay

- **Local-only upload acknowledgments:** recording the revisions actually encoded before the network call avoids crediting concurrent newer edits; keeping them out of manifests prevents acknowledgment-induced push loops. The unchanged-ref regression tests the observable consequence.
- **Sanitized typed Git failures:** classification stays beside stderr, while callers receive fixed messages and an inspectable error kind. Central locale configuration and removing `LANGUAGE` keep classification independent of the caller's language settings.
- **Read-only status computation:** typed summary/stage values preserve the prior JSON fields and compute through `inspect` without taking the writer lock. The existing lock-held CLI test continues to pass.
- **Publication recovery regression:** restoring only disposable state files recreates the important remote-commit/local-record discrepancy without a production fault hook, and checks both commit count and content.

#### Repair plan — before this leaves review

1. In a fresh `/develop TASK00001` session, fix **B1** and add a contention/recovery regression that verifies the durable status after reopening.
2. Implement HTTP-date retry delays and mock-provider coverage for **B2**; keep diagnostics free of body text.
3. Extend a genuine discard regression with status assertions for **B3**. Add **N1**'s summary-persistence integration case before merge if practical.
4. Run all mechanical gates, append dated as-built results, and clear the now-stale `open_findings` count as required by the invariants. Request a fresh `/code-review TASK00001` against the new commit.
5. Only a clean review may hand off to `/test-plan TASK00001`, which stamps the freeze. The agreed contract can be repaired without `/update-ticket`; ask Kai only if proposing to change the HTTP-date requirement instead.

### 2026-11-05 — branch task00001-stabilization-gaps at 7ac5316

**Scope and independence.** Re-review by the original reviewer, who authored none of the implementation or repairs. Reviewed all four changed files in `08c0b02..7ac5316` (`src/engine.rs`, `src/ai.rs`, `tests/engine.rs`, `tests/provider.rs`), their surrounding code, and the ticket's new as-built notes against the unchanged contract and the earlier complete `main...HEAD` review. The cumulative branch contains two ticket-scoped commits. Working tree was clean; no open PR was found. This session changes only the review record and ignored probe evidence.

**Gates.** Independently reran `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`: all passed on Windows. All **68 tests** passed: 5 library, 9 CLI, 27 engine, 19 provider and 8 storage/transport. Engine tests took 171.37 seconds. `git diff --check main...HEAD` passed. No live GitHub/provider tests or Linux/macOS checks were run; the previously recorded locale-catalog limitation remains.

**Verdict: changes still required — 1 blocking finding, 1 non-blocking finding.** `open_findings: 1` describes this commit, replacing the prior build's count. Keep `In progress`, leave `open_tests` unchanged and do not stamp the freeze or run `/test-plan` yet.

#### Earlier findings checked

| Finding | Result at this commit |
|---|---|
| B1 — lost sync-failure status under lock contention | **Resolved.** The sync workspace retains its lock while persisting the outcome. When network completion cannot regain the lock, the original Git error survives; persistence waits for the lock and reloads durable state before recording. The new long-held-lock regression passed, including reopening and clearing on a later successful sync. The original review's connection-refused stress probe, recompiled against this commit, observed **5 Git failures and 0 missing durable causes**. |
| B2 — HTTP-date retry delay omitted | **Partly resolved; remains blocking below.** Delta-seconds and preferred IMF-fixdate dates work, including elapsed dates and leap days. The implementation deliberately rejects the other two HTTP-date formats. |
| B3 — discarded status stage untested | **Resolved.** A genuine duplicate-change discard now asserts typed and serialized status before and after reopening, while retaining original snapshots and checking that `main` does not advance. |
| N1 — AI-failure persistence tested only by assigning a field | **Resolved.** The new binary/mock-provider regression exercises the real summary path, fallback publication, sanitized persisted status in a new process and clearing after a later successful summary. It passed. |

#### Findings, in severity order

1. **B2 continued — [P2, blocking] Accept all HTTP-date forms when reading `Retry-After`.** `src/ai.rs:194–201`; the corresponding tests at `tests/provider.rs:559–560` incorrectly group the RFC 850 and asctime forms with malformed input. With a fixed reference time of `784111717`, the preferred `Sun, 06 Nov 1994 08:49:37 GMT` returns `Some(60)`, but the equivalent `Sunday, 06-Nov-94 08:49:37 GMT` and `Sun Nov  6 08:49:37 1994` return `None`. The actual diagnostic for those elapsed dates is consequently `temporary rate limit; retry later` rather than `retry after 0 s`; a future value likewise loses its supplied delay.

   This is the remaining part of criterion **9** and plan step **5**, rather than a request for a new feature. [RFC 9110 §5.6.7](https://www.rfc-editor.org/rfc/rfc9110.html#section-5.6.7) defines HTTP-date to include the obsolete forms and requires recipients to accept all three. Its requirement for senders to generate IMF-fixdate does not waive the recipient requirement. [§10.2.3](https://www.rfc-editor.org/rfc/rfc9110.html#section-10.2.3) uses that HTTP-date definition for `Retry-After`. The as-built note documents the restriction, but no agreed contract change narrows date handling to IMF-fixdate.

   **Fix:** use a complete HTTP-date parser, or implement the remaining forms and their date/year rules. Change those two tests to assert the same delay as the preferred form, including future and elapsed values; keep genuine malformed values on the safe fallback and add a mock-provider assertion for an obsolete date. No contract update is needed to implement the existing requirement. A narrower supported date set would need a requester decision recorded through `/update-ticket` first.

2. **N2 — [P3, non-blocking] Reject impossible calendar dates instead of normalizing them.** `src/ai.rs:227–238` validates only `1..=31` before converting to Unix time. The probe `retry_after_seconds("Thu, 31 Feb 2028 00:00:00 GMT", 0)` returned `Some(1835568000)`, silently treating the nonexistent date as March 2. A malformed provider header can therefore produce a made-up numeric wait, contrary to the helper's documented malformed-input fallback. **Fix:** validate day-of-month with leap-year rules before conversion, or use the complete date parser from B2. Add February 30/31 and April 31 rejection cases alongside the valid leap-day test. This is invalid-header robustness; no additional acceptance criterion is marked failed for it.

**Evidence.** Retained locally in ignored `target/review-task00001/`: `fix-review-probe.rs`, `fix-review-probe-output.txt`, `fix-review-dates.rs` and `fix-review-dates-output.txt`. The probes link the current compiled library, use disposable workspaces for contention, and make no live provider calls. Original review evidence is retained separately.

#### Acceptance criteria and plan assessment

The earlier review's assessment of criteria 1–5, 6a, 10 and 11 still holds; their implementations are unchanged by the repair commit. Criteria **6 and 7** now pass the reviewed contention/recovery paths; criterion **8** now includes the previously missing discarded-stage assertion. Criterion **9** remains failed only for B2's valid date variants. Criterion **12** passes independently with 68 tests. The new AI persistence test closes the earlier optional seam gap. Both repair commits remain within this ticket, and their implementation decisions are recorded in dated as-built notes. No other new defect or undocumented plan departure was found.

#### What is good and should stay

- Recording against the existing locked workspace removes the original race. Reloading durable state before recording also avoids saving a failed sync's partially updated in-memory view over concurrent capture.
- Preserving the actual network failure through a lock timeout gives callers an actionable cause; the regression deliberately exceeds the earlier five-second window and then verifies recovery.
- The discard and AI-persistence tests exercise real domain/CLI transitions and reopening, so they protect behavior rather than merely assigning status fields.
- Deterministic reference timestamps and mock-provider tests usefully separate date arithmetic from HTTP-response handling; retain those cases while correcting the rejected formats.

#### Repair plan — before this leaves review

1. In a fresh `/develop TASK00001` session, finish **B2** by parsing all HTTP-date forms and correcting the tests that currently require valid formats to be rejected. Address **N2** while repairing the parser.
2. Run the mechanical gates, record the as-built result and clear the stale `open_findings` count when code changes.
3. Request another `/code-review TASK00001` against the new commit. A clean review can then hand off to `/test-plan`, which stamps the freeze; this review does not move the card or write a test plan.

### 2026-11-05 — branch task00001-stabilization-gaps at 291c263

**Scope and independence.** The same independent reviewer checked the complete `7ac5316..291c263` diff and surrounding code in `src/ai.rs` and `tests/provider.rs`, together with the new as-built notes and the unchanged ticket contract. Earlier full-branch and repair reviews remain inputs; the cumulative branch has three ticket-scoped commits. This session authored none of that code. Working tree was clean and no open PR was found. Restored the missing `## Code review` heading before the existing review entries; their content and the contract zone are unchanged.

**Gates.** Independently reran `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`: all passed on Windows. All **69 tests** passed: 5 library, 9 CLI, 27 engine, 20 provider and 8 storage/transport. Engine tests took 171.26 seconds. `git diff --check main...HEAD` passed. No live provider/GitHub checks or Linux/macOS verification were run; the previously recorded locale limitation remains.

**Verdict: changes required — 1 blocking finding, 0 non-blocking findings.** The previous B2 and N2 cases are repaired, but the newly introduced year helper has the boundary defect below. Set `open_findings: 1` for this commit; keep `In progress`, `open_tests` unchanged and the ticket unfrozen. `/test-plan` remains blocked by criterion 9.

#### Earlier findings checked

- **B2, all HTTP-date forms — resolved for the previously reported cases.** The three equivalent RFC example dates now return 60 seconds before the timestamp and 0 after it. The new mock-provider test exercises RFC 850 and asctime without echoing body text. This replaces the earlier deliberate rejection of those formats.
- **N2, impossible dates — resolved.** Month lengths and Gregorian leap years are validated before conversion; invalid dates in each form take the safe fallback. Independent comparisons include the non-leap century 2100 and leap century 2400.
- **B1, B3 and N1 remain resolved.** Their implementation and regression tests are unchanged by this commit, and the complete suite passed again.

#### Finding

1. **B4 — [P2, blocking] Apply RFC 850's 50-year cutoff to the complete timestamp.** `src/ai.rs:276–280` in `full_year`, called before parsing the month/day/time at `src/ai.rs:238`. It compares `candidate > current + 50`, so every date in year 2076 remains 2076 when the reference year is 2026, even after the actual 50-year anniversary. With `now_unix = 1793883600` (`2026-11-05T13:00:00Z`), `retry_after_seconds("Friday, 05-Nov-76 13:00:01 GMT", now_unix)` returns **`Some(1577923201)`**. The candidate is one second beyond 50 calendar years in the future, so the required interpretation is **1976**, already elapsed, and the result should be **`Some(0)`**. `Friday, 31-Dec-76 23:59:59 GMT` likewise incorrectly returns `Some(1582801199)`.

   [RFC 9110 §5.6.7](https://www.rfc-editor.org/rfc/rfc9110.html#section-5.6.7) applies this rule to the **timestamp**, rather than just its numeric year. This is a narrow boundary case in the new obsolete-format support; ordinary near-term retry dates pass. It affects the displayed retry delay, not captured data or the existing retry cadence. It nevertheless fails criterion **9**'s supplied-delay requirement.

   **Fix:** parse the candidate month/day/time as well as the year before deciding its century, compare against the complete 50-calendar-year threshold, and apply the past-century interpretation when beyond it. Preserve the distinction between exactly 50 years and more than 50 years. Validate the final interpreted calendar date. Add deterministic cases immediately before, exactly at, and one second after the threshold, plus a later month/day in the boundary year. Keep the existing early-2076 and 2080 tests, which do not expose this error. An established HTTP-date parser is also acceptable if its interpretation is checked against these cases. The contract does not need `/update-ticket` to make this repair.

#### Independent evidence and what should stay

- The reviewer generated **509 independent date/calendar cases** using Python's calendar and datetime calculations and compared them with the compiled Rust helper. All passed. Six checks reproducing the original three-format failure before and after the RFC example timestamp also passed. This confirms the useful format and invalid-date repairs without claiming those cases cover the newly found cutoff.
- The added month-length validation avoids silently normalizing impossible dates, including invalid leap days. Keep it alongside coverage for valid leap days.
- Future/elapsed mock-provider cases retain sanitized diagnostics and validate the actual HTTP response path. The direct date tests keep arithmetic reproducible with a fixed clock.
- All probe source, inputs and output are retained in ignored `target/review-task00001/`: `third-review-cases.py`, `third-review-cases.json`, `third-review-dates.rs`, `third-review-dates-output.txt`, `third-review-year-boundary.rs` and `third-review-year-boundary-output.txt`. No implementation code was modified.

**Acceptance criteria and plan.** All prior passing criteria remain supported by unchanged code and the rerun suite. Criteria 8 and 10 retain the repaired stage/provider coverage; criterion 12 passes with 69 tests. Criterion 9 is the sole failed criterion, for B4. The format-support and invalid-date decisions are documented in the new dated as-built entry. No unrelated branch changes or other undocumented plan departures were found.

#### Repair plan — before this leaves review

1. A fresh `/develop TASK00001` session repairs B4 using a full-timestamp cutoff and adds the boundary cases above.
2. Rerun the mechanical gates, append the result to the as-built notes and clear the stale findings count when code changes.
3. Request a fresh `/code-review TASK00001`. A clean result can hand off to `/test-plan`, which stamps the freeze. This review changes no ticket status or manual-test counter.

### 2026-11-05 — branch task00001-stabilization-gaps at c079e9d

**Scope and independence.** Re-reviewed the complete B4 repair in `291c263..c079e9d`: `src/ai.rs`, `tests/provider.rs` and their surrounding code, against the recorded failure, the ticket contract and the latest as-built notes. Earlier full-branch and repair reviews remain inputs. The reviewer authored none of the implementation. Review began while these two files were uncommitted; the author committed them during verification. Their SHA-256 hashes matched before and after that commit, so the code inspected and tested is the code in `c079e9d`. The working tree was clean at the final check. No open PR was found. No source code changed in this review session.

**Gates.** Independently ran `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`: all passed on Windows. All **69 tests** passed: 5 library, 9 CLI, 27 engine, 20 provider and 8 storage/transport. Engine tests took 182.13 seconds. `git diff --check main...HEAD` passed. These are local automated checks; no live provider/GitHub tests or Linux/macOS verification were performed. The previously recorded limitation concerning installed German Git translations remains.

**Verdict: clean review — 0 blocking findings, 0 non-blocking findings.** B4 is resolved, and no new actionable finding was identified in this repair. Set `open_findings: 0` for `c079e9d`. Leave `status: In progress`, `open_tests` and `frozen` unchanged: `/test-plan` is the next workflow and stamps the freeze against the reviewed commit.

#### Findings checked and acceptance criteria

- **B4 — resolved.** RFC 850 century selection now happens after parsing month/day/time and compares the entire candidate timestamp against the 50-calendar-year cutoff. Exactly at the cutoff stays in the future; a later instant is interpreted in the previous century. Both original reproductions (`Friday, 05-Nov-76 13:00:01 GMT` and `Friday, 31-Dec-76 23:59:59 GMT`, with the recorded reference clock) return `Some(0)`. Final calendar validation remains after century selection.
- **Independent boundary verification:** 360 Python-derived cases passed across reference dates in 2026–2030, month boundaries, differing times of day, and the instants immediately before, at and after the cutoff. These include leap-day reference dates and the recorded March 1 normalization policy. Both original B4 reproductions and all 509 earlier independent calendar cases passed against the current compiled library.
- **Earlier B1, B2, B3, N1 and N2 remain resolved.** The full suite passed again, including lock-contention persistence/recovery, all HTTP-date forms, invalid dates, discarded-stage/reopen behavior and the actual CLI AI-failure persistence path. This commit changes only the RFC 850 cutoff implementation and its regression assertions.
- Criterion **9**, previously blocked by B4, now passes the reviewed cases. All other acceptance criteria retain the passing assessments from the preceding reviews and the rerun suite; criterion **12** is independently green. No contract rewrite is required. The implementation and leap-day choice are recorded in the dated as-built notes. No unrelated changes or undocumented plan departures were found.

#### What is good and should stay

- Deferring two-digit-year interpretation until the complete timestamp is available directly addresses the failure instead of adding a special case for the reported date.
- Calendar validation runs on the final interpreted year, preserving the earlier invalid-date fix.
- The regression distinguishes before, exactly at and after the cutoff and preserves the early-2076/2080 cases; it would detect reintroduction of the year-only comparison.
- The inverse calendar helper uses the same civil-date model as the existing conversion, and independent timestamp comparisons cover the new calculation across month and leap-day boundaries.

**Evidence.** Ignored `target/review-task00001/` retains `fourth-review-boundary-cases.py`, `fourth-review-boundary-cases.json`, `fourth-review-boundary.rs` and `fourth-review-boundary-output.txt`. Earlier evidence is retained separately. Probes use fixed clocks and the compiled library; they contact no live provider and change no user workspace.

#### Before this leaves review

1. Run `/test-plan TASK00001` against `c079e9d`, writing only the required manual checks and stamping the ticket freeze through that workflow.
2. Keep subsequent review/test records tied to the commit that will ship. Any later implementation change invalidates this findings count and needs the applicable verification again.

### 2026-11-05 — Released in v0.2.5

Shipped in [[v0.2.5]] (tag `v0.2.5` on `85bfac3`). `completed` is unchanged.

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
