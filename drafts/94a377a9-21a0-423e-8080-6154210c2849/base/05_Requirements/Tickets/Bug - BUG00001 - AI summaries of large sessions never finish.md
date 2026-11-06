---
id: BUG00001
type: bug
status: In progress
priority: high
rank: 11000
version_target:
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings:
discussion:
updated: 2026-11-06
completed:
frozen:
---
With an AI provider configured, a closed session in a large workspace can stay in **Finishing up** indefinitely. The summary step gets one 0.5-second attempt at the workspace lock before the AI request and one after it. While sync and capture hold the lock for longer, both attempts fail silently: nothing is summarized and nothing tells the user why. When an attempt does get through, the provider's answer, already paid for, is thrown away because the second lock attempt fails. Mesh then asks again about a minute later. Reported by Kai on 2026-11-06 for the `dispatch/wiki` workspace; requested as an explicit `/fix-bug` shortcut.

<!-- ═══════════ CONTRACT ZONE ═══════════
     Freely editable until `frozen:` is stamped; read-only afterwards, with corrections
     appended as annotations rather than rewrites.
     ═══════════════════════════════════════ -->

## Symptom

**Observed:** session `1edabcff` (32 files: design-frame PNGs, HTML sources, ticket edits) in `dispatch/wiki` stayed in "Finishing up" for more than 16 minutes, with `status` showing `waiting_for_finalization`, no `reason`, no `summary_failure` and no `attention`. The tray opened an HTTPS connection about once a minute for 5–10 s, an AI request that was answered, yet no message was ever saved.

**Expected:** the closed session is summarized and added to shared history within one or two sync cycles, as it is in smaller workspaces; a provider answer that arrived is used, not discarded.

## Reproduction

1. A workspace with a provider configured (`ai_with_text`) and a remote, large enough that sync and capture hold the operation lock for more than 0.5 s (here 73 MB with a 6.5 MB PDF and design-frame PNGs).
2. Close a session (idle timeout or **Finish session**).
3. Watch `status`: the session stays `waiting_for_finalization`, and `summary_failure` stays empty.

Reliability: persistent on the live workspace (>16 min, and again under a diagnostic build). Not reproduced on isolated copies with a local mock provider, where the timing happened to let the first attempts through.

**Evidence (diagnostic tray build with logging in `summarize_ready`, 2026-11-06 20:37–20:39):** four of five rounds ended with `open#1 failed after ~503 ms: Another Mesh operation is using this workspace; retry shortly`, and the runner ignored that error. The fifth prepared the input in 11.7 s, the AI answered in 6.3 s ("docs: document desktop design frames and complete US00005"), every guard passed, and the message was saved; the restarted release tray then published it as `b441e87`.

## Environment

Windows 11; `dispatch-mesh-tray` release build of `819d9b0` (US00005 branch, version 0.2.5); workspace `dispatch/wiki` with remote `kaimys/dispatch-mesh-llm-wiki`, provider OpenAI Chat Completions with text sharing. The cause in `src/runner.rs` predates US00005.

## Root cause

`runner::summarize_ready` calls `Workspace::open` twice: before preparing the input and after the AI request. `Workspace::open` takes the operation lock with `LOCK_SETTLE` (0.5 s) and otherwise fails with `WorkspaceBusy`. The runner's `report_failure` ignores `WorkspaceBusy` by design, so a failed attempt leaves no trace, and the next tick starts the whole round again. On a large workspace, sync and capture routinely hold the lock longer than 0.5 s, so the first open usually loses. When it wins, the second open, after the paid AI request, can lose the same way, and the answer is dropped.

## Scope

- `summarize_ready` waits for the operation lock instead of failing after one attempt: both opens retry while the workspace is busy, for a bounded time (30 s, as the tray's **Finish session** does), sleeping asynchronously between attempts so the runner's loop keeps running.
- A provider answer that arrived is saved once the lock is free, never discarded because of a busy lock within that bound.
- Regression tests in `tests/provider.rs`: the summary finalizes while another operation holds the lock before the request, and while it holds the lock when the answer arrives.

## Out of scope

- Moving the summary step's blocking work (input preparation: about 12 s of Git calls on this workspace) off the runner's single async thread. It works, but it stalls the loop while it runs: a structural change for the full workflow.
- Recording a summary that keeps failing on the lock (beyond the 30 s bound) in `status`, so the user sees why finishing is slow. This changes what `status` reports and needs a decision.
- The general cost of locked operations on large workspaces (hashing every blob on each save): storage growth, see [[Story - US00011 - Limit saved copies in the recovery folder|US00011]] and the README *Storage growth* section.

## Open questions

None, because the cause is confirmed by the diagnostic log, the fix keeps every existing behavior except the length of the wait, and the bound copies the tray's existing **Finish session** wait. The target version is not set; it is asked at completion.

## Acceptance criteria

1. A closed session is summarized and its message saved even when another operation holds the workspace lock for longer than 0.5 s when the summary step starts.
2. An AI answer that arrives while another operation holds the lock is saved once the lock is released (within 30 s), without a second AI request.
3. Regression tests assert 1 and 2 and fail without the fix.
4. Gates green: `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.

## Implementation plan

Small-bug shortcut, no separate plan. Approach: in `src/runner.rs`, add an async helper that calls `Workspace::open` and, on `WorkspaceBusy`, sleeps 50 ms with `tokio::time::sleep` and retries until 30 s have passed (then returns the busy error as today). Use it for both opens in `summarize_ready`. Tests use the existing mock provider in `tests/provider.rs`, with a second thread holding the workspace open (and so its lock) for 1.5 s at the critical moment.

<!-- ═══════════ RECORD ZONE ═══════════ -->

## Test plan

Coverage for the small-bug shortcut (no separate `/test-plan`; its review gate is unmet by design).

- **Automated:** both acceptance-criteria cases are asserted by the two regression tests in `tests/provider.rs` (busy lock when the summary starts; busy lock when the answer arrives, with exactly one AI request). The full suite passes.
- **Original reproduction:** the live tray on `dispatch/wiki` with the real provider. Verified by finishing a session in that workspace under a release tray built from `403b165` (result below).

## As-built notes

### 2026-11-06 — fix (branch `BUG00001-summary-lock-wait`, commit `403b165`)

- **Explicit small-bug shortcut** (`/fix-bug`, requested by Kai). Independent code review omitted; `open_findings` left empty.
- `src/runner.rs`: new `open_when_free`, which retries `Workspace::open` on `WorkspaceBusy` with a 50 ms `tokio::time::sleep` between attempts for up to `SUMMARY_LOCK_WAIT` (30 s), then returns the busy error as before. Used for both opens in `summarize_ready`. No other behavior changed.
- `tests/provider.rs`: `a_summary_waits_for_a_workspace_busy_when_it_starts` and `an_answer_that_arrives_while_the_workspace_is_busy_is_kept` (one AI request only), plus a `serve_delayed` mock variant and an optional delay in `respond`. Both tests failed before the fix with "Another Mesh operation is using this workspace; retry shortly" and pass with it.
- Gates on `403b165` (Windows): `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all pass (engine suite 54 tests, provider 23, runner 11, tray host 10, tray model 20, CLI 11, storage/transport 8).

## Follow-ups

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]: found while verifying its release build; the stuck session was US00005's own wiki update.
- [[ADR-0009 — Embed the engine with a runner handle and published status]], [[ADR-0005 — Make AI summaries configurable and opt-in]]
- Code: `src/runner.rs` (`summarize_ready`, `report_failure`), `src/engine.rs` (`Workspace::open`, `LOCK_SETTLE`), `tray/src/host.rs` (`Host::finish`, the 30 s wait this copies).

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
