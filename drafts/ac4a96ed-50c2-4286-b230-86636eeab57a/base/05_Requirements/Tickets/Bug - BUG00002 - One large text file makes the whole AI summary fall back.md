---
id: BUG00002
type: bug
status: "In progress"
priority: high
rank: 12000
version_target:
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings:
discussion:
updated: 2026-11-07
completed:
frozen:
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

Small-bug shortcut, no separate plan. Approach: in `ChatCompletions::summarize`, build every change item first without text, recording for each shareable file its text, and the serialized size of that text (`{"before":…,"after":…}`). Measure the text-less payload; then add text to items in ascending size order while the running size stays within 512 KiB, setting `content_available: true` only for those. Keep the final `ensure!` as the guard for the text-less overflow. Test with the existing mock provider: a 600 KiB text file plus a small one.

<!-- ═══════════ RECORD ZONE ═══════════ -->

## Test plan

## As-built notes

## Follow-ups

## References

- Commit `d49570f` on `kaimys/dispatch-mesh-llm-wiki` (session `c65c0d1e`), the reported fallback.
- [[ADR-0005 — Make AI summaries configurable and opt-in]]; [[Bug - BUG00001 - AI summaries of large sessions never finish|BUG00001]] (same workspace, different cause).
- Code: `src/ai.rs` (`ChatCompletions::summarize`, `generate_with_failure`), `src/model.rs` (`binary`), `src/runner.rs` (`summary_failure` persistence).

## Dispatch runs
