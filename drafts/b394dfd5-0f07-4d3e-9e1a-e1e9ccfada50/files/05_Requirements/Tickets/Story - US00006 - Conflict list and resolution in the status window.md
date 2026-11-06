---
id: US00006
type: story
status: "Refinement"
priority: high
rank: 5000
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings:
discussion:
updated: 2026-11-06
completed:
frozen:
---
List unresolved conflicts in the status window and resolve them there. Each conflict shows its path, the competing authors and versions, timestamps, and which version is currently in the folder. Text conflicts open a comparison of the *current version* and the *other version*, with the common base available; binary conflicts offer whole-file choice only. **Keep current version** / **Use other version** use the same engine operation as `dispatch-mesh conflicts resolve --choose accepted|incoming`, and both versions stay preserved. This is the largest piece of 0.3 and carries the exit criterion "resolve a conflict through the desktop interface".

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a collaborator whose edit collided with someone else's, I want to see both versions side by side and choose one in the status window, so that I can settle the conflict without the CLI and without losing either version.

## Context

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]: work item "List unresolved conflicts. Show the path, competing authors and versions, and the currently displayed version. Support text comparison and version selection; binary resolution offers whole-file selection"; exit criterion "resolve a conflict through the desktop interface".
- [[Dispatch Mesh requirements and architecture]], *Conflict handling recommendation* and *Durable conflict records*; [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]: a resolution is a new traceable change, inputs are retained, and edits made since the conflict was recorded are not overwritten.
- [[Claude Design briefing — desktop app]] §6.3 items 3–4 (conflicts list, conflict detail), §5 vocabulary (*current version* / *other version*, never ours/theirs), §7 (compact window, expandable for comparison), §8 sample conflicts (`policies/retention.md`, `assets/logo.png`); frames in [[Dispatch Mesh desktop app.pdf]].
- CLI today: `conflicts list`, `show`, `export`, `resolve --choose accepted|incoming`. The tray already shows needs attention with "N unresolved conflicts" ([[Story - US00001 - Tray icon and status window host|US00001]]).
- [[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]] (AI-proposed merges, v0.4.0) builds on this review surface; its Q1 asks which manual review capabilities must exist first.
- Suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

## Scope

- Conflicts section per workspace with a count; the attention card's **Open conflicts** action leads there.
- Conflict detail for text: comparison of current and other version, base on demand, author, session, time and requested purpose per side.
- Conflict detail for binary files: metadata per side (size, type, author, time) and whole-file choice, no merge.
- **Keep current version** / **Use other version**, calling the engine's resolve operation in-process, with a clear statement that both versions are kept and later edits are protected.

## Out of scope

- Writing a third, combined version by hand, and AI-proposed merges ([[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]]).
- Toasts for new conflicts ([[Story - US00008 - Desktop notifications for conflicts and attention|US00008]]).
- Two-device verification of propagation ([[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]]).

## Open questions

Recorded here: no tracker or chat is configured. Refinement started 2026-11-06; engine facts verified against `src/engine.rs`, `src/status.rs`, `src/main.rs` and the design frames (PDF pp. 6–8).

**Engine facts (2026-11-06).**
- `Status.unresolved_conflicts` carries only `id`, `path`, `binary` (`src/status.rs`). Conflict detail exists only as `Workspace.state.conflicts` (`model::Conflict`: base/accepted/incoming bytes, incoming `session` + `author`, `binary`, `created_at`, `accepted_sessions`).
- The conflict record names the author and session of the **other version only**. The current version's contributors are the session ids in `accepted_sessions`; it is empty when the current version came from shared history (`main`), which records no author or time for it here. The base has no recorded date.
- `Workspace::resolve(id, incoming)` (what `conflicts resolve` calls) first captures, then **refuses** with "File has newer edits; review them before resolving this conflict" when the file on disk differs from the current version. The design frame instead says a later edit "is protected and reconciled separately once you choose". Today a user who edited the file after the conflict cannot resolve it at all, from CLI or window, without restoring the current version by hand.
- Either side may be a deletion (`None`). `binary` is set for either side containing binary bytes; non-UTF-8 text that is not flagged binary is possible.
- No diff library is in the dependency tree; text merging is done by Git.
- Precedent from [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]: the host calls `Workspace::finish()` in-process next to the runner (no `RunnerHandle` change), and workspace detail is read through `Status` (ADR-0010 amendment).

- [x] **Q1 Layout.** The design's detail frames are ~1040 px wide, side by side with changed lines marked; the window is 420–480 px. Does the same window widen for detail, does detail open in its own window, or is the comparison inline/stacked in the compact window? Are changed lines highlighted (needs a line diff), or are the two versions shown plain?
  **Answer (Kai, 2026-11-06):** the same status window widens to about 1040 px while conflict detail is open and returns to its compact width on Back. Versions side by side, changed lines marked; the line diff is computed in Rust (a small diff crate such as `similar`, chosen in the plan).
- [x] **Q2 Engine access.** How does the host get list metadata (authors, devices, times per side) and on-demand detail (bytes, base, purposes)? Proposal: extend `ConflictStatus` in `Status` with list metadata (amends ADR-0010), add an engine function for one conflict's detail read on demand, and resolve via `Workspace::resolve` in-process like US00005's finish — no ADR-0009 change.
  **Answer (Kai, 2026-11-06):** as proposed. `ConflictStatus` in `Status` gains list metadata (per side: author, device, time, size, deletion; amends [[ADR-0010 — Report workspace status and session stages separately|ADR-0010]], CLI `status` shows it too). One engine function returns a single conflict's detail on demand. The host resolves through `Workspace::resolve` in-process, like US00005's finish. No `RunnerHandle` or ADR-0009 change.
- [x] **Q3 Current-version attribution.** The record does not store who made the current version or when. Derive it from `accepted_sessions` (author, device, last save, purpose of those sessions) and, when it came from shared history, show "Shared history" with the commit's author/time — or store it in the conflict record going forward (a format change)?
  **Answer (Kai, 2026-11-06):** derive, no format change. Current version: the author, device, last save and requested purpose of the sessions in `accepted_sessions` (several contributors are all listed). When it is empty, the current version comes from shared history: show "Shared history" with the author and time of the last commit on `main` that touched the path. The base is labelled "Before both changes" with the same lookup.
- [x] **Q4 Edits made since the conflict.** The engine refuses to resolve once the file changed after the conflict; the design promises those edits are protected and reconciled separately. Which is the contract for 0.3: (a) keep the refusal and the window explains how to get back to a resolvable state, or (b) change the engine so resolving keeps the later edit as a new change reconciled on top of the choice (engine work, touches ADR-0003)?
  **Answer (Kai, 2026-11-06):** the engine reconciles: resolving is allowed, and a later edit is treated as a change on top of the current version and three-way merged onto the chosen version (clean → applied; overlapping → a new conflict). This is built as a **separate engine story that US00006 depends on** (see Follow-ups); US00006 stays UI plus the read API. Until it lands, the window shows the engine's refusal (Q8).
- [x] **Q5 Binary previews.** Which types get a preview (images only — which formats?), with what size cap, and how do bytes reach the webview without it reading arbitrary paths (e.g. a Tauri command returning the bytes for one conflict side)?
  **Answer (Kai, 2026-11-06):** PNG, JPEG, GIF and WebP get a preview, up to 10 MB per side; SVG is excluded (it can carry script). Other types and larger files show metadata only. The host returns one side's bytes for one conflict id through a Tauri command; the webview gets no file-system access.
- [x] **Q6 Text limits.** Very large text files and non-UTF-8 text not flagged binary: truncated comparison, or metadata-only like binaries? Where is the threshold?
  **Answer (Kai, 2026-11-06):** compare only when every present side is valid UTF-8 and at most 1 MB. Otherwise show the binary-style metadata view with the reason ("Too large to compare here" / "Not readable as text"); Keep / Use still work as whole-file choices.
- [x] **Q7 Deleted sides.** When one side is a deletion, what does that pane show, and how are the buttons worded ("Keep deleted" / "Restore other version")?
  **Answer (Kai, 2026-11-06):** the deleted side shows a placeholder ("Deleted by Bob · 10:50") instead of content, with no diff. Buttons keep the standard labels with a suffix: "Keep current version · deleted" or "Use other version · deleted" for the side that is a deletion.
- [x] **Q8 After resolving.** What does the window show afterwards (back to the list, a confirmation, the resolution "shared on next sync"), and how does it say both versions are kept?
  **Answer (Kai, 2026-11-06):** the window moves to the next unresolved conflict, or back to the list at compact width when none remain, with a dismissible line: "<path>: kept the current version / used the other version. Both versions stay kept; your choice is shared with collaborators on the next sync." The count and tray state follow the next snapshot. A failed resolve (e.g. the engine's refusal) shows the engine's reason in the detail view and changes nothing.

## Acceptance criteria

Status window and engine (Windows tray host; the same model code runs on Linux CI):

1. A workspace with unresolved conflicts shows a **Conflicts** tab with their count; the attention card's **Open conflicts** opens it. With none, the tab shows no count and an empty state.
2. Each list entry shows the path, text or binary (binary marked "whole-file choice"), when the conflict was recorded, and per side: author, device, save time, and which version is in the folder ("Current version · shown") and which is kept ("Other version · kept"). The current version's author comes from its contributing sessions, or "Shared history" with the last `main` commit's author and time when there are none.
3. CLI `status` includes the same per-conflict list fields, so CLI and window read one model.
4. **Review and choose** on a text conflict widens the window to the detail layout: current and other version side by side with line numbers, changed lines marked, and per side author, device, time and requested purpose (or "none recorded"). **Show common base** reveals the base. **Back** returns to the list at compact width.
5. A text conflict where any present side is over 1 MB or is not valid UTF-8 shows the metadata view with "Too large to compare here" or "Not readable as text"; both choices remain available.
6. A binary conflict shows per side the size, type, author, time and purpose. PNG, JPEG, GIF and WebP sides up to 10 MB show an image preview; SVG and all other types show metadata only. The webview receives preview bytes only through a host command keyed by conflict id and side, never a file path.
7. A side that is a deletion shows "Deleted by <author> · <time>" instead of content, and its button reads "… · deleted".
8. **Keep current version** and **Use other version** record the same resolution as `dispatch-mesh conflicts resolve <id> --choose accepted|incoming` (same engine call, same resulting record and folder content), while the tray's runner keeps running. The detail view says both versions stay kept.
9. After a successful choice the window opens the next unresolved conflict, or the list at compact width when none remain, with the confirmation line from Q8. With the next snapshot the count, attention card and tray state update; with no conflicts left the workspace leaves Needs attention for its next applicable state.
10. If the engine refuses (for example, the file has edits made after the conflict), the detail view shows the engine's reason, nothing is recorded, and both choices stay available.
11. No wording in the window uses "ours", "theirs", "accepted", "incoming" or "HEAD".
12. Host tests with real workspaces cover: list metadata for a text and a binary conflict (including one whose current version came from shared history), resolving each way from the host, the refusal path, a deletion side, and the 1 MB / non-UTF-8 fallback. Gates green.

The engine story for edits made after a conflict (Q4, Follow-ups) turns criterion 10's refusal into a reconciliation; US00006 is complete without it.

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

- 2026-11-06 (refinement, Q4): a new engine story is needed — resolving a conflict after the file was edited again reconciles that edit onto the chosen version instead of refusing (applies to CLI `conflicts resolve` as well). Not yet created; create it with `/create-ticket` and link it here.

## References

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]
- [[Dispatch Mesh requirements and architecture]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]]
- [[Story - US00001 - Tray icon and status window host|US00001]]

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
