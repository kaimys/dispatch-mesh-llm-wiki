---
id: BUG00002
type: bug
status: "Done"
priority: high
rank: 12000
version_target: v0.3.0
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests: 0
open_findings:
discussion:
updated: 2026-11-07
completed: 2026-11-07
frozen: 2026-11-07
---
With text sharing on, a session that contains one large text file gets no AI summary at all. The summary input carries the full before and after text of every changed file, and `ChatCompletions::summarize` refuses the whole request when that input exceeds 512 KiB. One oversized file therefore costs the summary for every small edit beside it: the commit gets the factual local message, and the tray shows *Needs attention* until a later summary succeeds. Reported by Kai on 2026-11-07 for the `dispatch/wiki` workspace; requested as an explicit `/fix-bug` shortcut.

<!-- ═══════════ CONTRACT ZONE ═══════════
     Freely editable until `frozen:` is stamped; read-only afterwards, with corrections
     appended as annotations rather than rewrites.
     ═══════════════════════════════════════ -->

## Symptom

**Observed:** the tray shows *Needs attention* with "AI summary failed: Summary input exceeds preview limit; using a factual local message". The session's commit `d49570f` on `kaimys/dispatch-mesh-llm-wiki` has the title "Update 7 workspace files" and the body "Saved editing session. No AI-generated explanation is available.", followed by the file list.

**Expected:** the session is summarized from the files whose text fits. A file too large to send is described by its operation alone, as a binary or excluded file already is, and the tray does not need attention.

## Reproduction

1. A workspace with a provider configured with text sharing on (`ai_with_text`).
2. In one session, edit a small text file and add, change or delete a text file larger than 512 KiB (here session `c65c0d1e`: the deleted `06_UX/Dispatch Mesh desktop app.html`, 1,356,759 bytes of valid UTF-8, next to two small Markdown edits and three small HTML edits; the 6.8 MB PDF is binary and already sent without text).
3. Let the session close. `status` reports `summary_failure` with the message above, and the commit uses the local fallback.

Reliability: always, for any session whose combined text exceeds 512 KiB.

## Environment

Windows 11; release tray of `main` at `403b165` (version 0.2.5); workspace `dispatch/wiki` with remote `kaimys/dispatch-mesh-llm-wiki`, provider OpenAI Chat Completions with text sharing. Platform-independent.

## Root cause

`src/ai.rs`, `ChatCompletions::summarize`: every text file that may be shared gets its full `before` and `after` text in the payload, and then a single `ensure!` rejects the payload as a whole if it serializes to more than 512 KiB. The cap is right, since it bounds what is sent to the provider, but it is all-or-nothing: there is no step that leaves out the text of the files that don't fit.

## Scope

- The 512 KiB cap becomes a budget for file text. Files whose text may be shared are given their text smallest-first while the payload stays within the cap. A file that doesn't fit is sent like a binary file: operation, existence and byte sizes, with `content_available: false` and no `before`/`after`.
- The local fallback stays for the case where even the payload without any file text exceeds the cap (thousands of files), with the same message.
- No other change to what is sent: binary, excluded and metadata-only files are treated as before, recorded intent is unchanged, and nothing beyond today's limit is ever sent.
- Regression test in `tests/provider.rs`.

## Out of scope

- Sending a diff or an excerpt of a large file instead of omitting its text. A better summary for large files, but a change to the provider input format for the full workflow.
- Classifying generated bundles (such as the compressed design export) as binary. Text/binary classification is an open question in the requirements (*How should text/binary classification, file-size limits, and large-binary storage be configured?*).
- Regenerating the message of `d49570f`: it is published, and shared history is not rewritten.

## Open questions

None, because the cause is confirmed from the live commit and the file sizes in it, the cap and the `content_available: false` treatment already exist, and the fix only ever sends less than before. The target version is not set; it is asked at completion.

## Acceptance criteria

1. With text sharing on, a session with one text file larger than the cap and one small text file is summarized by the provider: the small file is sent with its text, the large one with `content_available: false` and no `before`/`after`, and no summary failure is recorded.
2. The serialized summary input never exceeds 512 KiB.
3. Sessions whose text fits are sent exactly as before (existing provider tests unchanged and passing).
4. A regression test asserts 1 and 2 and fails without the fix.
5. Gates green: `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.

## Implementation plan

Small-bug shortcut, no separate plan. Approach: in `ChatCompletions::summarize`, build every change item first without text, recording for each shareable file its text, and the serialized size of that text (`{"before":…,"after":…}`). Measure the text-less payload; then add text to items in ascending size order while the running size stays within 512 KiB, setting `content_available: true` only for those. Keep the final `ensure!` as the guard for the text-less overflow. Test with the existing mock provider: a small text file plus two deleted text files of 300 KiB and 400 KiB, each under the cap alone but not together, so that the test also shows text is left out file by file.

<!-- ═══════════ RECORD ZONE ═══════════ -->

## Test plan

Coverage for the small-bug shortcut (no separate `/test-plan`; its review gate is unmet by design).

- **Automated:** criteria 1, 2 and 4 are asserted by `text_too_large_for_the_input_cap_is_left_out_file_by_file` in `tests/provider.rs`. Criterion 3: the 23 existing provider tests pass unchanged. Criterion 5: gates below.
- **Original reproduction (offline, 2026-11-07):** the 7 files of session `c65c0d1e`, before and after taken from `d49570f~1` and `d49570f`, run through `ai::generate_with_failure` with text sharing on against a local mock provider that records the request. That way no wiki text was sent to the real provider.
  - On `403b165` (before the fix): no request is sent; "Update 7 workspace files" with failure "AI summary failed: Summary input exceeds preview limit; using a factual local message", exactly as reported.
  - On `01da7b1` (the fix): one request with 140,232 bytes of input; the two Markdown files and three `.dc.html` frame sources are sent with their text, the deleted `Dispatch Mesh desktop app.html` (1,356,759 bytes) and the binary PDF with `content_available: false` and no text; the provider's message is used and no failure is recorded.
- **Not checked live:** the real provider was not called. Its side of the request is unchanged (same fields, same prompt), and the first live session over the cap will show the result in the tray. Nothing requires a human check: `open_tests: 0`.

## As-built notes

### 2026-11-07 — fix (branch `BUG00002-summary-text-budget`, commit `01da7b1`)

- **Explicit small-bug shortcut** (`/fix-bug`, requested by Kai). Independent code review omitted; `open_findings` left empty.
- `src/ai.rs`: new `SUMMARY_INPUT_LIMIT` (512 KiB, the existing value). `ChatCompletions::summarize` builds every change item without text and with `content_available: false`, sizes each shareable file's `{"before","after"}` text, then adds text smallest-first while the payload stays within the limit. The size added is computed exactly (the object's braces stand in for the two commas; `true` is one byte shorter than `false`). The final `ensure!` stays as the guard for a payload too large even without text.
- `tests/provider.rs`: `text_too_large_for_the_input_cap_is_left_out_file_by_file`. It failed before the fix (title "Update 3 workspace files", the local fallback) and passes with it.
- Gates on `01da7b1` (Windows): `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all pass (engine suite 54 tests, provider 24, runner 11, tray host 10, tray model 20, CLI 11, storage/transport 8, lib 5).
- Completed 2026-11-07 under the shortcut, with `version_target: v0.3.0` set by Kai. Not merged: the fix is on branch `BUG00002-summary-text-budget`; release, tag and merge are outside `/fix-bug`.
- Merged 2026-11-07: `main` fast-forwarded to `01da7b1` and pushed, and `target/release` rebuilt from it, at Kai's request after completion.
- Noticed on the way: the stderr log line repeats the fallback phrase ("…using a factual local message; using a factual local message…") because the error text and `generate_with_failure` both say it. Wording only, not changed here.

## Follow-ups

- Sending a diff or an excerpt of a file too large for the cap, instead of leaving its text out (see *Out of scope*): not yet ticketed.

## References

- Commit `d49570f` on `kaimys/dispatch-mesh-llm-wiki` (session `c65c0d1e`), the reported fallback.
- [[ADR-0005 — Make AI summaries configurable and opt-in]]; [[Bug - BUG00001 - AI summaries of large sessions never finish|BUG00001]] (same workspace, different cause).
- Code: `src/ai.rs` (`ChatCompletions::summarize`, `generate_with_failure`), `src/model.rs` (`binary`), `src/runner.rs` (`summary_failure` persistence).

## Dispatch runs
