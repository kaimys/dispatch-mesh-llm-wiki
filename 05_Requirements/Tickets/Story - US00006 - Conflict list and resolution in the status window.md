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
updated: 2026-11-07
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
- **Readable design (since 2026-11-06):** [[Dispatch Mesh desktop app — design spec]], section *Conflicts (3d, 3e, 3f)* plus *Tokens* and *Components*; frame images `06_UX/frames/03d-conflicts-list.png`, `03e-text-conflict.png`, `03f-binary-conflict.png`; markup with sample data in `06_UX/frames/source/03d–03f-*.dc.html`. The PDF and the HTML bundle have no extractable text. Where the spec and a PNG disagree, the spec wins. Where this ticket and the spec disagree, the decisions recorded below win (2026-11-07).
- **Builds on [[Story - US00012 - Status window layout follows the design frames|US00012]]** (Kai, 2026-11-07: developed before this ticket). It restyles the window to the frames: tokens, status glyphs, top row, state card and the Activity / Conflicts section nav, whose Conflicts tab holds a placeholder that this ticket replaces (US00012 Q2). US00012 also adds `SessionStatus.device` for "You" (its Q5), and decides that devices are never named, because device ids are random UUIDs (its Q6). This ticket uses those tokens and components and does not restyle them.
- CLI today: `conflicts list`, `show`, `export`, `resolve --choose accepted|incoming`. The tray already shows needs attention with "N unresolved conflicts" ([[Story - US00001 - Tray icon and status window host|US00001]]).
- [[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]] (AI-proposed merges, v0.4.0) builds on this review surface; its Q1 asks which manual review capabilities must exist first.
- Order update (Kai, 2026-11-07): US00005 is done; [[Story - US00012 - Status window layout follows the design frames|US00012]] comes before this ticket. Original suggested 0.3 order (milestone plan, 2026-11-06): [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → [[Story - US00006 - Conflict list and resolution in the status window|US00006]] → [[Story - US00007 - Recent-save presence in the status window|US00007]] → [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] → [[Story - US00009 - Pause and resume synchronization|US00009]] → [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]].

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
- Either side may be a deletion (`None`). `binary` is set when any side contains a NUL byte or is not valid UTF-8 (`model::binary`), so a text conflict always has valid UTF-8 sides. *(Corrected 2026-11-06 during planning: refinement wrongly said non-UTF-8 text could stay flagged as text.)*
- `Workspace::open` does not wait for the operation lock: it fails with `WorkspaceBusy` after 500 ms (`src/engine.rs`, `operation_lock`). The runner holds the lock during local work, so a host call must retry, as the runner's own shutdown capture does (`capture_for_shutdown`, 10 s). `Workspace::inspect` reads committed state lock-free and read-only.
- No diff library is in the dependency tree; text merging is done by Git.
- Precedent from [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]: the host calls `Workspace::finish()` in-process next to the runner (no `RunnerHandle` change), and workspace detail is read through `Status` (ADR-0010 amendment).

- [x] **Q1 Layout.** The design's detail frames are ~1040 px wide, side by side with changed lines marked; the window is 420–480 px. Does the same window widen for detail, does detail open in its own window, or is the comparison inline/stacked in the compact window? Are changed lines highlighted (needs a line diff), or are the two versions shown plain?
  **Answer (Kai, 2026-11-06):** the same status window widens to about 1040 px while conflict detail is open and returns to its compact width on Back. Versions side by side, changed lines marked; the line diff is computed in Rust (a small diff crate such as `similar`, chosen in the plan).
- [x] **Q2 Engine access.** How does the host get list metadata (authors, devices, times per side) and on-demand detail (bytes, base, purposes)? Proposal: extend `ConflictStatus` in `Status` with list metadata (amends ADR-0010), add an engine function for one conflict's detail read on demand, and resolve via `Workspace::resolve` in-process like US00005's finish — no ADR-0009 change.
  **Answer (Kai, 2026-11-06):** as proposed. `ConflictStatus` in `Status` gains list metadata (per side: author, device, time, size, deletion; amends [[ADR-0010 — Report workspace status and session stages separately|ADR-0010]], CLI `status` shows it too). One engine function returns a single conflict's detail on demand. The host resolves through `Workspace::resolve` in-process, like US00005's finish. No `RunnerHandle` or ADR-0009 change.
- [x] **Q3 Current-version attribution.** The record does not store who made the current version or when. Derive it from `accepted_sessions` (author, device, last save, purpose of those sessions) and, when it came from shared history, show "Shared history" with the commit's author/time — or store it in the conflict record going forward (a format change)?
  **Answer (Kai, 2026-11-06):** derive, no format change. Current version: the author, device, last save and requested purpose of the sessions in `accepted_sessions` (several contributors are all listed). When it is empty, the current version comes from shared history: show "Shared history" with the author and time of the last commit on `main` that touched the path. The base is labelled "Before both changes" with the same lookup.
- [x] **Q4 Edits made since the conflict.** The engine refuses to resolve once the file changed after the conflict; the design promises those edits are protected and reconciled separately. Which is the contract for 0.3: (a) keep the refusal and the window explains how to get back to a resolvable state, or (b) change the engine so resolving keeps the later edit as a new change reconciled on top of the choice (engine work, touches ADR-0003)?
  **Answer (Kai, 2026-11-06):** the engine reconciles: resolving is allowed, and a later edit is treated as a change on top of the current version and three-way merged onto the chosen version (clean → applied; overlapping → a new conflict). This is built as a **separate engine story that US00006 depends on** ([[Story - US00010 - Resolve a conflict after the file was edited again|US00010]]); US00006 stays UI plus the read API. Until it lands, the window shows the engine's refusal (Q8).
- [x] **Q5 Binary previews.** Which types get a preview (images only — which formats?), with what size cap, and how do bytes reach the webview without it reading arbitrary paths (e.g. a Tauri command returning the bytes for one conflict side)?
  **Answer (Kai, 2026-11-06):** PNG, JPEG, GIF and WebP get a preview, up to 10 MB per side; SVG is excluded (it can carry script). Other types and larger files show metadata only. The host returns one side's bytes for one conflict id through a Tauri command; the webview gets no file-system access.
- [x] **Q6 Text limits.** Very large text files and non-UTF-8 text not flagged binary: truncated comparison, or metadata-only like binaries? Where is the threshold?
  **Answer (Kai, 2026-11-06):** compare only when every present side is valid UTF-8 and at most 1 MB. Otherwise show the binary-style metadata view with the reason ("Too large to compare here" / "Not readable as text"); Keep / Use still work as whole-file choices.
  *Note (2026-11-06, planning):* non-UTF-8 content is already binary by the engine's definition (see engine facts), so for text conflicts only the 1 MB limit applies; non-UTF-8 files get the binary view.
- [x] **Q7 Deleted sides.** When one side is a deletion, what does that pane show, and how are the buttons worded ("Keep deleted" / "Restore other version")?
  **Answer (Kai, 2026-11-06):** the deleted side shows a placeholder ("Deleted by Bob · 10:50") instead of content, with no diff. Buttons keep the standard labels with a suffix: "Keep current version · deleted" or "Use other version · deleted" for the side that is a deletion.
- [x] **Q8 After resolving.** What does the window show afterwards (back to the list, a confirmation, the resolution "shared on next sync"), and how does it say both versions are kept?
  **Answer (Kai, 2026-11-06):** the window moves to the next unresolved conflict, or back to the list at compact width when none remain, with a dismissible line: "<path>: kept the current version / used the other version. Both versions stay kept; your choice is shared with collaborators on the next sync." The count and tray state follow the next snapshot. A failed resolve (e.g. the engine's refusal) shows the engine's reason in the detail view and changes nothing.

**Design decisions at planning (Kai, 2026-11-07)**, from reading [[Dispatch Mesh desktop app — design spec]] against this ticket. None of these was an open question, so the counter is unchanged:

- [x] **Button suffix.** Footer buttons carry the side's author, as in frame 3f, for text and binary alike: "Keep current version (Alice)" / "Use other version (Bob)". This replaces frame 3e's changed-value suffix ("· 60 days").
- [x] **Edits made after the conflict, before US00010.** The design's note ("That edit is protected and reconciled separately once you choose") is not true until [[Story - US00010 - Resolve a conflict after the file was edited again|US00010]] ships. Instead, the detail view shows an honest note in the same style: "You saved this file again at <time>, after the conflict was recorded. Your edit is safe. Choosing a version is possible once the file matches the current version again." Both buttons are disabled while that holds. US00010 switches to the design's wording and enables them.
- [x] **One primary button.** **Review and choose** is a secondary button on every list card, so the components rule "one primary button per view" holds. Frame 3d draws it as primary.
- [x] **No "saved copy" for conflict sides.** The binary detail says "Images can't be combined. Choose which whole file stays in the folder; the other version stays kept." Frame 3f's "kept in history as a saved copy" is not used, because "saved copy" means a recovery file since US00005.

## Acceptance criteria

Status window and engine (Windows tray host; the same model code runs on Linux CI):

1. A workspace with unresolved conflicts shows a **Conflicts** tab with their count; the attention card's **Open conflicts** opens it. With none, the tab shows no count and an empty state.
2. Each list entry shows the path, text or binary (binary marked "whole-file choice"), when the conflict was recorded, and per side: author, save time, and which version is in the folder ("Current version · shown") and which is kept ("Other version · kept"). The current version's author comes from its contributing sessions, or reads "Shared history" when there are none. *(Kai, 2026-11-06, at planning: the list carries in-memory data only, so no Git lookup runs on each runner tick; the commit's author and time appear in the detail view, criterion 4.)* *(2026-11-07: "device" dropped from this criterion and from criterion 4 per US00012 Q6. A device has no readable name; the author reads "You" for this device's contributions, per US00012 Q5.)*
3. CLI `status` includes the same per-conflict list fields, so CLI and window read one model.
4. **Review and choose** on a text conflict widens the window to the detail layout: current and other version side by side with line numbers, changed lines marked, and per side author, time and requested purpose (or "none recorded"). A current version from shared history shows the author and time of the last commit on `main` that touched the path, when that commit's content is the current version. **Show common base** reveals the base. **Back** returns to the list at compact width.
5. A text conflict where any present side is over 1 MB shows the metadata view with "Too large to compare here"; both choices remain available. (Content that is not valid UTF-8 is binary by the engine's definition and gets the binary view, criterion 6.)
6. A binary conflict shows per side the size, type, author, time and purpose. PNG, JPEG, GIF and WebP sides up to 10 MB show an image preview; SVG and all other types show metadata only. The webview receives preview bytes only through a host command keyed by conflict id and side, never a file path.
7. A side that is a deletion shows "Deleted by <author> · <time>" instead of content. Its button names the deletion ("Use other version (deleted by Bob)"), which replaces the "· deleted" suffix of the Q7 answer (planning, 2026-11-07).
8. **Keep current version** and **Use other version** record the same resolution as `dispatch-mesh conflicts resolve <id> --choose accepted|incoming` (same engine call, same resulting record and folder content), while the tray's runner keeps running. The buttons carry the side's author ("Keep current version (Alice)"; "(shared history)" when the current version has no contributing session). The footer says "Both versions stay in history whichever you choose." (binary: "Whole-file choice only. Both versions stay in history."). **Review and choose** on the list is a secondary button.
9. After a successful choice the window opens the next unresolved conflict, or the list at compact width when none remain, with the confirmation line from Q8. With the next snapshot the count, attention card and tray state update; with no conflicts left the workspace leaves Needs attention for its next applicable state.
10. When the file was saved again after the conflict was recorded, the detail view shows the note from the planning decisions with the time of that save, and both buttons are disabled. If the engine refuses anyway (a race with a save, or a busy workspace), the detail view shows the engine's reason, nothing is recorded, and the buttons follow the latest state.
11. No wording in the window uses "ours", "theirs", "accepted", "incoming" or "HEAD".
12. Host tests with real workspaces cover: list metadata for a text and a binary conflict (including one whose current version came from shared history), resolving each way from the host, the refusal path, a deletion side, and the 1 MB / non-UTF-8 fallback. Gates green.
13. The list, text detail and binary detail match frames 3d, 3e and 3f in structure and use the spec's tokens in light and dark theme, except where the planning decisions above differ. The comparison is checked side by side with screenshots, as in the spec's *How to check a build*, and recorded in the test plan.

The engine story for edits made after a conflict (Q4, Follow-ups) turns criterion 10's note and refusal into a reconciliation; US00006 is complete without it.

## Implementation plan

*Rewritten 2026-11-07 against `main` at `403b165` (US00005 done, BUG00001), the design spec and the planning decisions. It replaces the 2026-11-06 draft, which predated US00005's build.*

### Ground truth that shapes the plan

- **Status and engine.** `ConflictStatus` is still `{ id, path, binary }` (`src/status.rs`), built in memory by `Workspace::status_from` for CLI `status` and every runner snapshot. US00005 added session times, purposes, titles, `last_sync` and `saved_copies` there; conflicts were untouched.
- **Conflict record.** `model::Conflict` has base/accepted/incoming bytes, the other version's `session` and `author`, `binary`, `created_at` and `accepted_sessions`. Sessions stay in `state.manifest.sessions` after they settle, so contributors' `author`, `device`, `changes[path].last_edit` and `intents` are in memory. `model::binary` means NUL or not valid UTF-8.
- **Resolve.** `Workspace::resolve(id, incoming)` captures, refuses when `state.observed[path] != accepted`, records, and renders. CLI `conflicts resolve` calls exactly that. `state.observed` is the last captured folder view, so "edited after the conflict" can be computed from committed state.
- **Commits on `main`** carry the session author (`Git::commit_objects`), so `git log -1 <main> -- <path>` names a real person.
- **Lock.** `Workspace::open` fails with `WorkspaceBusy` after 500 ms. US00005's `Host::finish` (`tray/src/host.rs`) retries `open` for 30 s inline; the engine's own `acquire_lock` is private. `Workspace::inspect` reads committed state lock-free and read-only.
- **Tray.**
  - `WorkspaceView` has `action: Option<&'static str>` (today only `finish_session`), `sessions` and `saved_copies`.
  - The commands are `workspaces`, `retry`, `finish_session` (async, `spawn_blocking`) and `show_saved_copy`; the window sends ids only.
  - The CSP allows `img-src 'self' data:`. Capabilities: `core:event:default`. The window is 440 × 520 in `tauri.conf.json`; US00012 makes the default 440 × 640.
  - No diff crate in `Cargo.lock`.
- **Design.** Frames 3d–3f and the spec's *Conflicts* section. The detail window is "about 1000 × 760". Differing lines are tinted `attention-card` with a "≠" gutter mark named for screen readers. The base strip is `#EFEEEA` in light, with no dark value given. The other version's box has a 1 px dashed `#8A8C91` border, and the current version's a 1.5 px solid `text` border.

### Steps, in order

1. **Host: one helper to wait for the lock** (`tray/src/host.rs`). Extract US00005's inline 30 s retry from `Host::finish` into a private `open_engine(path)`, and use it for both finish and resolve. No engine API change: the retry is a host concern, and the CLI keeps failing fast on purpose.
2. **Engine: list metadata in `Status`** (`src/status.rs`). `ConflictStatus` keeps `id`, `path` and `binary` and gains:
   - `created_at`;
   - `current` and `other`, each a `ConflictSide { contributors, shared_history, deleted, size }`;
   - `contributors`: a list of `{ author, device, last_saved, purposes }`, from the sessions in `accepted_sessions` (current) or the conflict's `session` (other). `last_saved` is that session's `changes[path].last_edit`, and `purposes` are its intents whose paths include the conflict's path;
   - `shared_history`: the current side has no contributing session;
   - `deleted`: the side is `None`; `size`: the byte length.

   In-memory only: no Git and no bytes. JSON names say `current` and `other`, never accepted/incoming.
3. **Engine: one conflict's detail on demand** (new `src/conflicts.rs`, an `impl Workspace` block like `src/status.rs`; `src/git.rs`). `Workspace::conflict_detail(id) -> Result<ConflictDetail>` returns:
   - the `ConflictStatus` fields, plus the `base`, `current` and `other` bytes;
   - `edited_after`: `Some(time)` when `state.observed[path]` differs from the current version. The time is the latest `last_edit` of this device's session changes to that path after `created_at`, falling back to `created_at`;
   - `history` for the current side and the base: the author and time of `git log -1 <state.main> -- <path>`, **only when** that commit's blob at `path` equals the side's bytes. Otherwise the last commit may postdate the version and would name the wrong person.

   Add `Git::last_commit(reference, path) -> Option<CommitInfo>` with `validate_path`. The method works on a `Workspace::inspect`, so it never waits for the runner. CLI `conflicts show` is unchanged.
4. **ADR-0010 amendment** (wiki, with steps 2–3): the new `ConflictStatus` fields. The rule that `Status` stays in memory, with no Git or content, and that per-conflict content, history and `edited_after` come through `conflict_detail` on demand.
5. **Host** (`tray/src/host.rs`). Each method is refused, with a message, unless the workspace runs in this host (as for `finish`), and checks the conflict id against the latest snapshot's `unresolved_conflicts`.
   - `conflict_detail(workspace, conflict)` calls `Workspace::inspect(path)?.conflict_detail(id)`.
   - `conflict_preview(workspace, conflict, side)` (`current`, `other` or `base`) returns `(mime, bytes)` only when the bytes sniff as PNG, JPEG, GIF or WebP by magic number and are at most 10 MB. SVG is never an image here. The sniffing function is public, so the model uses the same rule.
   - `resolve(workspace, conflict, choice)` (`current` or `other`) calls `open_engine` and then `resolve(id, choice == other)`. It returns `Ok`, or the engine's message (`{error:#}`).
6. **Tray model** (`tray/src/model.rs`; new dependency `similar` in `tray/Cargo.toml`), pure and unit-tested; all wording lives here:
   - **Workspace view.** `WorkspaceView.action` gains `open_conflicts` when the reason is the unresolved-conflict count, and `WorkspaceView` gains `conflicts`, the list entries:
     - path and kind: "Text", "Image · whole-file choice", "File · whole-file choice";
     - "recorded <time>";
     - two side boxes, "Current version · shown" and "Other version · kept", each with the author ("You" when the contributor's device is this device, per US00012 Q5; "Shared history"; never a device, per US00012 Q6) and "saved <time>";
     - the heading "N unresolved conflicts" and the explanation line from 3d.

     Times are epoch seconds, formatted by the window as US00005 does.
   - **Detail view.** `conflict_detail_view(detail, this_device) -> ConflictDetailView` holds:
     - **Header and labels:** the breadcrumb count "n of N"; the summary line "<current author> and <other author> both changed this file. Pick the version to keep in the folder." (binary: the 3f sentence as decided); the side titles and badges "In the folder now" / "Kept, not shown"; and the author line with purposes, or "Requested purpose: none recorded";
     - **Base strip:** "Common base", "Before both changes", plus " · shared history, <date>" when history is known;
     - `body`, which is one of:
       - `Text { current_lines, other_lines, base_lines }`, each line with number, text and a `changed` flag;
       - `Metadata { reason }`, "Too large to compare here" when a present side is over 1 MB;
       - `Binary { sides }`, with saved by, saved, size, purpose and a preview flag;
     - `note`: the planning decision's later-edit note when `edited_after` is set, and then `can_choose: false`;
     - **Buttons:** "Keep current version (<author>)" / "Use other version (<author>)". With several contributors the first name is followed by "+N". The current version from shared history reads "(shared history)"; a deletion reads "(deleted by <author>)";
     - **Footer:** "Both versions stay in history whichever you choose." (binary: "Whole-file choice only. Both versions stay in history.").
   - **Line marks.** They come from `similar::TextDiff::from_lines(current, other)` with a 200 ms deadline. On timeout no line is marked and the view says "Changes could not be marked here", so a pathological file cannot hang the window.
   - **After a choice.** `next_after(resolved, conflicts)` picks the next unresolved conflict in list order, or `None`. `confirmation(path, choice)` returns the Q8 line.
7. **Tauri wiring** (`tray/src/main.rs`, `tray/tauri.conf.json`):
   - **Commands:**
     - `conflict_detail`;
     - `conflict_preview`, which returns `tauri::ipc::Response` raw bytes (an `ArrayBuffer` in JS);
     - `resolve_conflict`;
     - `set_layout(compact|detail)`.

     The first three run through `spawn_blocking`, as `finish_session` does.
   - **Layout.** `set_layout("detail")` remembers the current size and resizes to 1000 × 760 logical pixels, clamped to the monitor's work area. `"compact"` restores the remembered size. Resizing happens in Rust, so the webview needs no window permission.
   - **CSP:** `img-src` gains `blob:`. Capabilities are unchanged.
8. **Window** (`tray/ui/app.js`, `app.css`), using US00012's tokens, buttons, badge and section nav:
   - **Navigation:** Conflicts tab content per 3d; the attention card's **Open conflicts** switches to it.
   - **Detail view** per 3e/3f:
     - breadcrumb "‹ Conflicts / n of N";
     - mono 18 path heading;
     - **Show common base** toggle and base strip;
     - two columns with line numbers;
     - ≠ marks with the accessible name "Differs from the other version";
     - image previews through `URL.createObjectURL` (revoked on leaving);
     - the metadata list, the note and the footer.
   - **Pending resolve:** both buttons are disabled with a busy label. On error the message appears inline. On success the view moves on per `next_after` with the confirmation line. A conflict that disappears from the snapshot while open sends the window back to the list with "This conflict was resolved elsewhere."
   - **New tokens** in `app.css`: `base-strip` (`#EFEEEA`, with a derived dark value checked for 4.5:1) and `kept-border` (`#8A8C91`, with a dark value). Add both to the spec's token table.
   - **Rules:** text only through `textContent`; `data-key` and an accessible name on every control; changed lines carry the text mark, not only the tint.
9. **Docs:**
   - README: the `status` conflict fields, and the tray section now lists conflict review.
   - AGENTS.md repository map: the tray is no longer "status only".
   - The ADR-0010 amendment (step 4).
   - The design spec: the planning decisions noted under *Conflicts*, and the two new tokens. Kai owns that page; I propose the edit in the same change.

**Order and why.**
- **After US00012:** this ticket uses its tokens, components, top row and section nav, and replaces the Conflicts tab's placeholder (US00012 Q2). Building conflict views on the old styles means restyling them twice.
- **Engine and host before the window** (steps 2–3 before 5, 5 before 8): host tests need real conflict data, and the window has no logic of its own to test.
- **ADR amendment with the code:** step 4 lands together with steps 2–3, so the ADR never describes a different model.

### Tests

- **Engine** (`tests/engine.rs`, `tests/cli.rs`):
  - `status` lists a two-device text conflict with both sides' authors, devices, `last_saved` and an intent-scoped purpose;
  - a binary conflict with sizes;
  - a delete-vs-edit conflict with `deleted`;
  - a current version from `main` (Alice publishes; Carol, offline on the old base, edits the same line; Bob, with no session of his own, receives Carol's draft): `contributors` is empty and `shared_history` is true. If that setup doesn't yield an empty `accepted_sessions`, record why in the as-built notes and cover the case with a constructed state;
  - `conflict_detail` returns the exact bytes; `history` appears only when the commit content matches; `edited_after` is set after a later local save and unset otherwise;
  - CLI `status` keeps every existing key.
- **Tray model** (unit tests):
  - list and detail wording, including "You", "Shared history", "Deleted by …", "(deleted by Bob)", "(shared history)", "+N" and the binary sentence without "saved copy";
  - the 1 MB boundary (exactly 1 MB compares, one byte more does not);
  - sniffing: PNG, JPEG, GIF and WebP yes; SVG, a renamed text file and 10 MB + 1 no;
  - line marks for the retention sample;
  - the later-edit note sets `can_choose: false`;
  - `next_after` at the start, middle and end, and the confirmation line;
  - no model string contains "ours", "theirs", "accepted", "incoming", "HEAD" or "saved copy" in a conflict view.
- **Host** (`tray/tests/host.rs`, with a local bare remote and two workspaces):
  - resolve each way while the runner runs; the folder and record equal what CLI `conflicts resolve` produces;
  - after a later save, `conflict_detail` reports `edited_after` and `resolve` returns the engine's refusal without recording anything;
  - all three methods are refused for a workspace held by another runner, and for an unknown conflict id;
  - `conflict_preview` returns `None` for a non-image;
  - `finish` still works through `open_engine`.
- **Manual** (for `/test-plan`):
  - screenshots of the list, text detail and binary detail against 3d/3e/3f in light and dark (criterion 13);
  - widening and restoring at 100 % and 150 % scaling and on a small screen;
  - image previews;
  - Narrator on the tabs, ≠ marks, panes and buttons;
  - a window-made resolution picked up by a second client (the full two-device check is TASK00003).
- **When US00010 lands**, the later-edit note and refusal tests turn into reconciliation tests. That change belongs to US00010.

### External changes and migration

- `status` JSON: `unresolved_conflicts[]` gains fields, and the existing ones are unchanged. No state or synchronized-format change: everything is derived from existing records.
- New tray dependency `similar` (MIT/Apache-2.0). The CSP adds `blob:` to `img-src`.

### Wiki pages this makes wrong

- [[ADR-0010 — Report workspace status and session stages separately]]: amended in step 4.
- README and AGENTS.md: step 9.
- [[Dispatch Mesh desktop app — design spec]], *Conflicts*: still describes the frame's value suffix, the "protected and reconciled" note, a primary button on every card and "saved copy". Step 9 updates it to the planning decisions.

### Risks and rollback

- **Window resizing** across DPI settings and monitors: clamped to the work area, covered by manual checks.
- **Large IPC payloads** (a 10 MB image per side): raw bytes, on demand only.
- **Diff cost:** capped by the 1 MB limit and the 200 ms deadline.
- **Lock wait:** a resolve during a long local operation waits up to 30 s with a busy label, as Finish session does, then shows `WorkspaceBusy`'s message; nothing is recorded.
- **Stale detail:** `edited_after` comes from the last capture, so a save in the last tick can slip past the note. The engine's own guard still refuses, and the window shows the reason (criterion 10). A conflict that vanished sends the window back to the list.
- **Depends on US00012's outcome** (chrome and nav). If its decisions change the top row, the breadcrumb sits wherever US00012 puts the back link.
- **Rollback:** revert the commits. There is no data or format migration either way.

### Decisions for review

1. The list shows "Shared history" without author or time, and the detail adds the commit (Kai, 2026-11-06).
2. A commit on `main` is shown only when its content matches the version.
3. The line diff runs in the tray (`similar`, 200 ms deadline), not in the engine.
4. Previews travel as raw IPC bytes into `blob:` URLs, never as file paths. The type comes from magic bytes.
5. The 30 s lock wait from US00005 is shared through a host helper, with no engine API change.
6. Detail and resolve work only for workspaces this host runs.
7. Detail layout 1000 × 760 logical pixels (design spec), clamped, with the compact size restored on Back.
8. Button author suffix: the first contributor plus "+N" for several, "(shared history)" and "(deleted by …)". This applies Kai's 2026-11-07 decision to the cases the frames don't show.
9. The detail summary line is generic ("<A> and <B> both changed this file."), without the frame's "the same line within a minute", which would need line-level and timing claims the engine doesn't make.

### Size

Order of magnitude: about five to seven focused days, roughly two for the engine and host with tests and three to five for the model, wiring and window. US00012 removes the styling groundwork from this estimate. Not a commitment.

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

- 2026-11-06 (refinement, Q4): a new engine story is needed — resolving a conflict after the file was edited again reconciles that edit onto the chosen version instead of refusing (applies to CLI `conflicts resolve` as well). Created as [[Story - US00010 - Resolve a conflict after the file was edited again|US00010]] (Backlog, v0.3.0).

## References

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]
- [[Dispatch Mesh requirements and architecture]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]]
- [[Story - US00001 - Tray icon and status window host|US00001]]

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
