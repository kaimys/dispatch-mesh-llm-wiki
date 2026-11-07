---
id: US00006
type: story
status: "Review"
priority: high
rank: 5000
version_target: v0.3.0
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests: 9
open_findings: 0
discussion:
updated: 2026-11-07
completed:
frozen: 2026-11-07
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

Manual checks for the conflict list and resolution in the status window, on branch `us00006-conflict-resolution` at `2066559` (Windows, debug build). Only what the suites and the checks I ran do not cover: how the window looks against frames 3d–3f in both themes, the timeout message, the window resizing on real displays, and a choice made in the window reaching a second client. Wording, line marks, the 1 MB and image rules, resolving each way, the refusal path and the deletion side are asserted by the model, engine and host tests.

**Setup (once, PowerShell, repository root).** Build with `cargo build`, then run the script below. It creates a disposable remote with Alice and Bob and gives Bob three unresolved conflicts, which Bob's tray shows. The tray runs on Bob's workspace, so "You" is Bob and Alice made the current version (the one in Bob's folder).

| File | Kind | What it is for |
|---|---|---|
| `policies/retention.md` | text | Only the "Retention: N days" line differs (Alice 60, Bob 90). Frame 3e. |
| `assets/logo.png` | image | Alice's is blue, Bob's orange, 240 × 150. Frame 3f. |
| `data/export.csv` | text | Two files of 40 000 lines each, drawn from 64 values in two unrelated orders. The exact line diff cannot finish in its 200 ms deadline, so no line is marked. These are the same inputs as the unit test `a_pathological_file_cannot_hang_the_comparison`. |

```powershell
$t = "$env:TEMP\mesh-us6"; $m = (Resolve-Path .\target\debug\dispatch-mesh.exe).Path
$A = "$t\Alice"; $B = "$t\Bob"
if (Test-Path $t) { Remove-Item $t -Recurse -Force }
New-Item -ItemType Directory $A, $B | Out-Null; git init -q --bare "$t\remote.git"
@'
import os, struct, sys, zlib
folder, who = sys.argv[1], sys.argv[2]
def put(rel, data):
    p = os.path.join(folder, *rel.split("/")); os.makedirs(os.path.dirname(p), exist_ok=True)
    open(p, "wb").write(data)
def png(rgb, w=240, h=150):
    raw = b"".join(b"\0" + bytes(rgb) * w for _ in range(h))
    def chunk(k, d): return struct.pack(">I", len(d)) + k + d + struct.pack(">I", zlib.crc32(k + d))
    return b"\x89PNG\r\n\x1a\n" + chunk(b"IHDR", struct.pack(">IIBBBBB", w, h, 8, 2, 0, 0, 0)) + chunk(b"IDAT", zlib.compress(raw)) + chunk(b"IEND", b"")
def shuffled(seed):
    s, out = seed, []
    for _ in range(40000):
        s = (s * 6364136223846793005 + 1442695040888963407) % (1 << 64)
        out.append("line %d\n" % ((s >> 33) % 64))
    return "".join(out).encode()
days = {"base": 30, "alice": 60, "bob": 90}[who]
put("policies/retention.md", ("# Retention\n\nDocuments are kept for a fixed period.\n\nRetention: %d days\n\nAfter that they move to the archive.\nThe archive is reviewed once a year.\n" % days).encode())
put("assets/logo.png", png({"base": (120, 120, 120), "alice": (31, 111, 235), "bob": (214, 90, 49)}[who]))
put("data/export.csv", {"base": b"line 0\n", "alice": shuffled(1), "bob": shuffled(2)}[who])
'@ | Set-Content "$t\make.py" -Encoding utf8
& $m --workspace $A init --name Alice --email alice@example.test --remote "$t\remote.git"
& $m --workspace $B init --name Bob --email bob@example.test --remote "$t\remote.git"
python "$t\make.py" $A base
& $m --workspace $A session finish; & $m --workspace $A sync; & $m --workspace $A sync
& $m --workspace $B sync
python "$t\make.py" $A alice; python "$t\make.py" $B bob
& $m --workspace $A sync; & $m --workspace $B sync
& $m --workspace $B conflicts list
.\target\debug\dispatch-mesh-tray.exe --workspace $B
```

The last `conflicts list` prints three conflicts (`data/export.csv`, `assets/logo.png`, `policies/retention.md`). Open the window from the tray icon. Windows must be in **light** mode for checks 1–4 and 6–9, and the last check changes `retention.md`, so do it last. The spec's *How to check a build against this page* applies to checks 1–3 and 5: screenshots next to the frames in `06_UX/frames/`, recorded under *Results*. **Cleanup:** Quit the tray from its menu, then `Remove-Item $env:TEMP\mesh-us6 -Recurse -Force`.

Where the build differs from a frame on purpose, that is a recorded decision (ticket *Open questions*, planning decisions of 2026-11-07), not a defect: buttons name the side's author (frame 3e's "· 60 days" suffix is not used), **Review and choose** is a secondary button on every card (3d draws it primary), the binary sentence has no "saved copy", and the metadata list has a **Type** row.

- [ ] **Kai: Light theme, conflict list against frame 3d (AC 1, 2, 13).** In the Conflicts tab (reached from the attention card's **Open conflicts**), screenshot the list at the default size next to `03d-conflicts-list.png`. Expected: the tab shows the count 3. The heading is "3 unresolved conflicts" with the explanation line below it. There is one card per file with a file icon, the path in mono, "Text · recorded …" (the PNG: "Image · whole-file choice") and two side boxes: **Current version · shown** with Alice, a 1 px solid dark border; **Other version · kept** with "You", a dashed grey border; both with "saved <time>". No device is named. **Review and choose** is a secondary button on every card and there is no primary button in the view. Tokens, spacing and type match the frame apart from the decisions above.
- [ ] **Kai: Light theme, text detail against frame 3e (AC 4, 13).** Choose **Review and choose** on `policies/retention.md`. Screenshot next to `03e-text-conflict.png`. Expected: the window widens to about 1000 × 760 and stays on screen. Breadcrumb "‹ Conflicts / n of 3", path heading in mono, the toggle **Show common base**, a summary line "Alice and You both changed this file. Pick the version to keep in the folder." Two columns with line numbers. The current column has a 1.5 px solid border and "In the folder now"; the other has a dashed border and "Kept, not shown". Only the "Retention:" line is tinted, with a "≠" in its gutter, in both columns; the other lines are not. Each column names its author and time and says "Requested purpose: none recorded". **Show common base** reveals a strip with the caps label "Common base" and "Before both changes" above the columns, showing "Retention: 30 days"; pressing it again hides it. Footer "Both versions stay in history whichever you choose." Buttons: primary **Keep current version (Alice)**, secondary **Use other version (You)**, both enabled. Do not press either yet.
- [ ] **Kai: Light theme, binary detail and previews against frame 3f (AC 6, 13).** Back (the "‹ Conflicts" crumb), then **Review and choose** on `assets/logo.png`. Screenshot next to `03f-binary-conflict.png`. Expected: the sentence "Images can't be combined. Choose which whole file stays in the folder; the other version stays kept." Both previews show, on a checkerboard: Alice's blue in the current box, Bob's orange in the other. Each side lists Saved by, Saved, Size, Type (PNG) and Purpose. The footer reads "Whole-file choice only. Both versions stay in history." and there is no side-by-side text and no "≠".
- [ ] **Kai: "Changes could not be marked here" in the window (AC 4; review round 2).** Back, then **Review and choose** on `data/export.csv`. Expected: the view opens within a few seconds without freezing the window. Above the two columns, a summary-style line says "Changes could not be marked here". Both columns still show their 40 000 lines with line numbers and scroll, no line is tinted and no "≠" shows. Both buttons are enabled. Then open `policies/retention.md` again: the message is absent there and its "Retention:" line is marked. Screenshot the `export.csv` view.
- [ ] **Kai: Dark theme, all four views (AC 13).** Switch Windows to dark mode (Settings → Personalization → Colors → Choose your mode: Dark), close the window with X and reopen it from the tray icon. Go through the list, `policies/retention.md`, **Show common base**, `assets/logo.png` and `data/export.csv`, with a screenshot of each next to its frame. Expected: the window follows the theme and every view uses the dark tokens. Text, line numbers and the "≠" mark are readable on their backgrounds. The marked line is still tinted. The base strip is a visible, darker-than-page strip with readable text, not the light `#EFEEEA`. The dashed border of the other version can be seen against the page, and so can the solid border of the current one. The checkerboard and previews are visible. "Changes could not be marked here" is readable above the unmarked columns of `export.csv`. Switch back to light afterwards.
- [ ] **Kai: Widening and Back at 100 % and 150 % (AC 4, 13).** In Settings → System → Display, set the scale to 100 %, reopen the window from the tray, open `policies/retention.md`, then press **Back**; repeat and press Escape instead. Do the same at 150 %. Expected at each scale: opening the detail grows the window to about 1000 × 760 logical pixels (at 100 %, 1000 × 760 px; at 150 %, as much of that as the work area allows), wholly inside the work area with its bottom edge above the taskbar and no part cut off. Back and Escape each return the window to the compact size it had before, without a flicker to another size. The two columns and both footer buttons are visible without horizontal scrolling. Set the scale back afterwards.
- [ ] **Kai: Small screen (AC 4).** Make the work area smaller than the detail window: set the scale to 200 % on a 1920 × 1080 display (work area about 960 × 520 logical), or use a display of that size. Reopen the window and open `policies/retention.md`. Expected: the window fills the work area and is no larger than it, with its title bar and edges on screen. The detail view's content scrolls, so both footer buttons can be reached. **Back** restores the compact window as it was before. Set the scale back afterwards.
- [ ] **Kai: A maximized window is left alone (AC 4).** Maximize the status window, open a conflict, then press **Back**. Expected: the window stays maximized the whole time; it is neither shrunk to 1000 × 760 nor restored to the compact size. Close it with X and reopen it from the tray: it is still maximized.
- [ ] **Kai: A choice made in the window reaches a second client (AC 8, 9).** Do this last. With `policies/retention.md` open, press **Use other version (You)**. Expected in the window: the buttons are disabled with a busy label for a moment, then the window shows the next conflict and a dismissible line "policies/retention.md: used the other version. Both versions stay kept; your choice is shared with collaborators on the next sync." The count drops to 2. Then, in PowerShell: `& $m --workspace $A sync` twice (the variables from setup, in the same session), and open `$t\Alice\policies\retention.md`. Expected: it reads "Retention: 90 days", and `& $m --workspace $A conflicts list` no longer lists `policies/retention.md`; the other two files stay listed. This is one machine and one Windows account, so it does not prove two separate hosts; that is TASK00003.

**Verified already, so not in the list above:**
- **By me, 2026-11-07, at `2066559`:** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` green. The setup script runs end to end and gives Bob exactly the three conflicts above. CLI `status` on Bob's workspace names Alice as the contributor of each current version and Bob as each other version's. The three files are valid text or PNG as intended. After CLI `conflicts resolve <id> --choose incoming` on `policies/retention.md` and syncs on both sides, Alice's folder holds "Retention: 90 days" and her conflict list drops that file, so the last check's expected result holds for the engine. I did not run the window: the app needs a user session to click through, and I did not switch the Windows theme or scale of your machine.
- **By the review:** the app code for the message (`conflictBody` in `tray/ui/app.js`), the serialization names, and that the unit test fails when the deadline is not reached.
- **By the developer's app run (as-built notes):** the list with "You" and "Alice", the "≠" line, **Show common base**, both image previews through `blob:` URLs under the CSP, a delete-versus-edit conflict, the later-save note with both buttons disabled, Escape returning to the list, and the confirmation line and next conflict after each choice.
- **By the automated suites:** all wording and the forbidden words, the 1 MB boundary, image sniffing and the 10 MB cap, line marks, `next_after`, list and detail metadata for each kind of conflict, resolving each way while the runner runs (same record and folder as the CLI), the refusal path, refused workspaces and ids, and the `status` JSON keys.
- **Not in the plan on purpose:** screen-reader checks (`dispatch/invariants.md`, *No screen-reader tests*); accessible names and keyboard order were built and reviewed, not part of the manual pass.

**Results.**

*(Empty. Testers add dated entries here, tick the items above and lower `open_tests`.)*

## As-built notes

- **2026-11-07 — development started** (Kai confirmed the run plan: "develop US00006 until it has a test plan"; no gate waived). Status moved `Refinement` → `In progress`, `assignee: Kai`. Branch `us00006-conflict-resolution`, stacked on `main` (`bdf8684`); the `/orchestrate` run cut it, so `/develop` step 2 (pull) was skipped.
- **2026-11-07 — drift since the plan (written against `403b165`, before US00012).** Built against the current code; the frozen plan text is unchanged:
  1. `WorkspaceView.action` already returned `open_conflicts`, the attention card's **Open conflicts** button already switched to the Conflicts tab. Plan step 6's first bullet and step 8's navigation bullet were done; reused.
  2. The Conflicts tab already had its count badge, accessible name (`conflicts_tab`) and `CONFLICTS_REASSURANCE`. Replaced the placeholder (`model::conflicts_placeholder`, `WorkspaceView.conflicts_placeholder`) with `WorkspaceView.conflicts` (the list and the empty state).
  3. The window already opens at 440 × 640.
  4. `SessionStatus` and `Status` already carry `device`; the "You" rule uses it.
  5. Widening reuses the launch fit: `model::widen` fits both axes with `fit_vertically` (it works along either axis), `set_layout` leaves a maximized window alone (both ways), never shrinks a window already larger than 1000 × 760, and on Back restores only the remembered size.
- **2026-11-07 — deviations from the plan, each small:**
  - **Module.** The conflict views live in `tray/src/model/conflicts.rs`, a submodule of `model.rs` (1.7k lines already), not inside it. `AGENTS.md`'s repository map says so.
  - **Engine tests** are in a new `tests/conflicts.rs` (the status fields, `conflict_detail`, the `status` JSON keys, the shared-history setup from the plan), not in `tests/engine.rs` and `tests/cli.rs`. The shared-history setup (Alice publishes, Carol edits from the old base offline, Bob without a session of his own) does give an empty `accepted_sessions`, so no constructed state was needed.
  - **`ConflictSide.size`** is a `u64` that is `0` for a deletion; `deleted` tells them apart.
  - **Detail position and "next".** "n of N" and the conflict to open next come from the engine's committed state (`Workspace::status_from` on the inspected or opened workspace), not from the runner's latest snapshot, which lags a resolve by up to a second and would otherwise say "1 of 2" for the last conflict. `next_after` wraps: after the last conflict it names the first one that remains, and returns `None` only when no other conflict is left (Q8).
  - **Diff.** `similar` 3.2.1, `TextDiff::configure().deadline(200 ms)`. The crate has no timeout flag, so a run that took the whole deadline counts as timed out and marks no line ("Changes could not be marked here").
  - **Wording the plan left open.** A deletion gets its own summary line ("Bob deleted this file while you changed it."; "You" reads "you" mid-sentence). A non-image binary says "These files can't be combined." (the frame's "Images" sentence only when a side is an image). The metadata list adds **Type** (sniffed by magic bytes, else the extension) and shows no dimensions. The "≠" mark is named per column ("Differs from the other version" in the current column, "… the current version" in the other). A text file over 1 MB gets the same whole-file layout as a binary one, with its reason.
  - **Live note.** The open detail re-reads the conflict whenever the runner publishes a snapshot, and after a refused choice, so a later save shows its note and disables the buttons without reopening (criterion 10, "buttons follow the latest state"). Only the note changes; the versions shown stay as loaded.
  - **Host API.** `Host::conflict_detail` returns the finished `ConflictDetailView` (it needs the device and the list order from the snapshot and the engine), `resolve_conflict` returns `Resolved { confirmation, next }`, `conflict_preview` returns the bytes with their image type or `None`. All three refuse a workspace the engine does not run here and a conflict the snapshot does not list.
  - **Base history.** As planned, only the last commit on `main` touching the path counts. For the base it often matches (the base is usually the last published version); when someone published again, the strip reads just "Before both changes".
- **2026-11-07 — verified by running the app** (debug build, two real workspaces on a local remote): the list with "You" and "Alice", the text comparison with the "≠" line and the purpose of a scoped intent, **Show common base**, an image conflict with both previews through `blob:` URLs under the CSP, a delete-versus-edit conflict ("Deleted by Alice", "Use other version (deleted by Alice)", the file removed from the folder), the later-save note appearing live with both buttons disabled, Escape returning to the list at the compact size, and the confirmation line plus the next conflict after each choice. Screenshot comparison against frames 3d–3f in light and dark and at other scalings stays for the test plan (criterion 13).
- **2026-11-07 — observed, not changed:** a save made after the conflict was recorded made the runner record a second conflict for the same file (the engine merges the new edit against the other version). That is the situation US00010 reconciles; until then the window shows the later-save note on it.
- **2026-11-07 — code review blocked the hand-off** (round 1, at `73334c1`): finding 1 fails AC 4 when the line diff runs out of time. The window then marks no line and does not show "Changes could not be marked here". The ticket stays `In progress`, and `/test-plan` waits for the fix and its re-check. See *Code review*.
- **2026-11-07 — code review round 1, finding 1 fixed** (fix commit `2066559`, on top of `73334c1`). Finding 1 (AC 4, blocking): `conflictBody` in `tray/ui/app.js` now renders `body.marks_unavailable` as a `detail-summary` line above the columns, only for a text comparison (`data-key="marks-unavailable"`). `a_pathological_file_cannot_hang_the_comparison` now asserts that `marks_unavailable` is `Some(MARKS_UNAVAILABLE)` and that no line is `changed`. While writing that assertion it turned out that the review's example, 40 000 distinct lines per side, never reached the 200 ms deadline (the diff finished in about 0.1 s and marked every line), so the test had not covered the timeout at all. The input is now 40 000 lines drawn from 64 distinct values in two unrelated pseudo-random orders, which does hit the deadline. Gates green (`cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`). I did not run the app for this change, only `node --check` on `app.js`; the message needs a file that times out the diff, which is a case for the test plan. Findings 2–8 are non-blocking and stay as follow-ups. `open_findings` cleared to empty; the re-check of finding 1 belongs to round 2 of the review.

## Code review

### 2026-11-07 — branch us00006-conflict-resolution at 73334c1, stacked on main

**Scope and gates.** Range `bdf8684..73334c1`, two commits, both US00006: `1a3c2df` (engine and `status`) and `73334c1` (tray). Read:
- the whole ticket, with the five drift points Kai confirmed on 2026-11-07 (as-built notes);
- ADR-0003, ADR-0009 and the ADR-0010 amendment;
- the design spec's *Conflicts* section;
- every file in the diff, with its tests.

No PR is open. This review ran the gates on Windows at `73334c1`, with a clean tree that includes the README, AGENTS.md and wiki edits. `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and the full `cargo test` pass (190 tests, 0 failed). The window was reviewed from source; this review did not run the app. The developer's app run is in the as-built notes, and the screenshot comparison belongs to the test plan (criterion 13).

**Verdict.** Blocked: one blocking finding, seven non-blocking. US00006 stays `In progress`, and `/test-plan` waits for the fix and its re-check.
- Criteria 1–3 and 5–12 hold as written. Criterion 13 is manual.
- The code follows the plan as amended by the as-built notes. Every deviation has a dated as-built entry.

1. **Blocking · AC 4 · When the line diff runs out of time, no line is marked and the window does not say so.**
   - **What happens.** `tray/src/model/conflicts.rs:693` sets `Body::Text.marks_unavailable` to "Changes could not be marked here" when `similar` reaches the 200 ms deadline, and leaves every line unmarked. `tray/ui/app.js` never reads `marks_unavailable`: `conflictBody` renders only the path, the summary, the base strip, the metadata reason and the two columns.
   - **Scenario.** A text conflict under 1 MB in which both people rewrote most lines, such as a regenerated export or a reformatted file (the unit test's 40 000 distinct lines per side is one case). The window shows the two versions side by side with no "≠" mark and no explanation. In this view, that reads as "no line differs", and the user is choosing which version stays in the folder.
   - **Why it blocks.** AC 4 promises marked changed lines. Plan step 6 allows the timeout only together with the message, and the as-built note on the diff says the message is shown.
   - **Fix.** Render `body.marks_unavailable` in `conflictBody`, as a line above the columns that uses the existing tokens. Extend `a_pathological_file_cannot_hang_the_comparison` (`conflicts.rs:1348`) to assert that `marks_unavailable` is `Some(MARKS_UNAVAILABLE)` and that no line is `changed` when the deadline was hit.

2. **Non-blocking · Back during a choice drops the confirmation line.**
   - **What happens.** The "‹ Conflicts" crumb (`tray/ui/app.js:537`) stays enabled while a choice is under way, though Escape is guarded (line 693). Clicking it closes the detail, so `choose()` returns early when the resolve succeeds (`if (state.closed) return;`). The Q8 line never appears, even though the choice was recorded.
   - **Fix.** Disable the crumb while `state.busy` is set, as Escape already behaves.
3. **Non-blocking · "This conflict was resolved elsewhere." when the engine stops.**
   - **What happens.** `closeGoneConflicts` (`tray/ui/app.js:649–660`) treats an empty list as a resolution. When the workspace's runner stops (state `Stopped`), the workspace has no status and no entries. An open conflict then closes with "This conflict was resolved elsewhere.", which is false.
   - **Fix.** When the workspace has no running status (`ws.retry`), close the detail without that line. Keep the line for a conflict that left a live list.
4. **Non-blocking · The keep button does not name a deletion that came from shared history.**
   - **What happens.** `suffix` (`tray/src/model/conflicts.rs:477–485`) checks `shared_history` before `deleted`. The button then reads "Keep current version (shared history)", although AC 7 asks the button to name the deletion; AC 8's "(shared history)" rule collides with AC 7 only in this case. The list card (`app.js:304`) does not say "deleted" either, because a shared-history side has no time to show.
   - **Fix.** Choose the wording, for example "(deleted, shared history)", and record it in an as-built note. The panel's placeholder already reads "Deleted by <commit author> · <time>".
5. **Non-blocking · The summary line ignores a known commit author.**
   - **What happens.** `summary` (`tray/src/model/conflicts.rs:586–611`) uses `who()`. When `current_history` names Alice, the detail says "Shared history and Bob both changed this file." while the panel beside it says "Alice · shared history".
   - **Fix.** Use the commit author in the sentence when the detail has one.
6. **Non-blocking · The live note re-reads the whole conflict on every snapshot.**
   - **What happens.** While a conflict is open, `refreshNote` (`tray/ui/app.js:435–451`) calls `conflict_detail` on every published snapshot and keeps only `note` and `can_choose`. Each call loads the state, runs up to six Git processes, recomputes the diff (up to 200 ms) and sends every line of all three versions, which is several MB for a 1 MB file.
   - **Fix.** Add a small host command that returns `edited_after` for one conflict. This is performance only: the `refreshing` guard already prevents overlapping calls.
7. **Non-blocking · A text file over 1 MB says "No preview for this file type."**
   - **What happens.** A text file over 1 MB gets the binary layout (`tray/src/model/conflicts.rs:563–568`, `app.js:470–479`). The layout shows an empty preview frame with that line, under "Too large to compare here".
   - **Fix.** Leave the preview frame out for `Body::Metadata`.
8. **Non-blocking · AC 12 asks for host tests where the plan placed engine tests.**
   - **What happens.** List metadata for a text and a binary conflict is covered with real workspaces in the engine suite `tests/conflicts.rs`, where the plan's *Tests* section assigns it. Only the shared-history case is also covered in `tray/tests/host.rs`.
   - **Fix.** The coverage exists. If the criterion's wording should match, annotate AC 12 at the freeze.

**What held up.**
- **History names the right person.** `history_of` (`src/conflicts.rs:131–142`) names a commit only when its blob is exactly the version. It hashes with `hash-object --stdin`, which applies no filters, and queries with `--literal-pathspecs`. The test that replaces the base with bytes no commit holds pins this. It is what keeps the detail from naming the wrong person after a later publication.
- **Position and "next" come from committed state.** "n of N" and the next conflict come from the engine's committed state, not from the lagging snapshot (`tray/src/host.rs`). The list, the breadcrumb and `next_after` share one ordering function, `conflicts::ordered`. Keep both.
- **Previews never expose a path.** The format is sniffed by magic bytes, with SVG excluded. The 10 MB cap lives in one public function that the host and the model both use. Bytes travel as raw IPC, keyed by workspace, conflict id and a validated side, and `blob:` is the only CSP addition. The host test sends `../../note.md` as a side and gets a refusal.
- **Strong evidence for AC 8.** The host resolve test builds two identical collisions and resolves one through the host while the runner runs, the other with a direct `Workspace::resolve`. It then compares the records and the folder content byte for byte.
- **Forbidden words cannot slip in.** The wording test walks every string of every serialized view, so a new label cannot bring in "incoming" or "saved copy" unnoticed.
- **The lock wait stays in the host.** `open_engine` keeps the 30 s wait a host concern, and the CLI still fails fast.
- **`Status` stays in memory,** as the ADR-0010 amendment requires: a snapshot carries no Git output and no file bytes, and everything else comes on demand through `conflict_detail` on `Workspace::inspect`.

**Before this leaves review.**
- A fresh `/develop US00006` session fixes finding 1: the window shows `marks_unavailable`, and the pathological test asserts it. Findings 2–8 are optional for that session.
- Round 2 re-checks finding 1 at the fix commit. `/test-plan` follows.

### 2026-11-07 — branch us00006-conflict-resolution at 2066559, stacked on main (round 2)

**Scope and gates.** This round re-checks the blocking finding from round 1.
- **Range.** `73334c1..2066559`: one commit, `2066559` (US00006). It changes `tray/ui/app.js` (3 lines) and one test in `tray/src/model/conflicts.rs`. The rest of `bdf8684..2066559` was reviewed in round 1 and is not reviewed again.
- **Read.** The round 1 entry, the as-built entry of the fix round, and the whole fix diff. Also the code that diff relies on: `compare`, the serde attributes of `Body`, `el`, and `codeColumn`.
- **Gates.** This review ran them on Windows at `2066559`, with a clean tree. `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and the full `cargo test` pass (190 tests, 0 failed), including the changed `a_pathological_file_cannot_hang_the_comparison`. `node --check tray/ui/app.js` passes.
- **Not run.** This review did not run the app.

**Verdict.** Clean. Finding 1 is fixed, and the fix diff brings no new finding. `open_findings` is now `0`, which ends the code review of US00006. The ticket can go to `/test-plan`.

**Finding 1 (AC 4): fixed.**
- **Window.** `conflictBody` (`tray/ui/app.js:531–533`) shows `body.marks_unavailable` as a `detail-summary` line between the base strip and the two columns. The line appears only when `body.kind === "text"` and the field is set.
  - The names match the Rust side: `Body` is serialized with `tag = "kind"` and `rename_all = "snake_case"`, so the window receives `kind: "text"` and `marks_unavailable` as the string, or as `null`, which the check skips.
  - The message is added as a text node through `el`, not as markup.
- **Model.** Unchanged and already correct.
  - `compare` (`conflicts.rs:402–421`) treats a diff that used the whole 200 ms as timed out and then marks no line.
  - `detail_view` turns that into `Some(MARKS_UNAVAILABLE)` (line 693).
  - A side that is a deletion never runs the diff, so the line cannot appear for one.
- **Test.** `a_pathological_file_cannot_hang_the_comparison` now asserts both parts of the timeout behaviour: no line is `changed`, and `marks_unavailable == Some(MARKS_UNAVAILABLE)`.
  - Because of that assertion, a green run proves the deadline was reached. The test can no longer pass on the fast path without anyone noticing, which the round 1 version did.
  - The input is fixed. Two seeded linear congruential generators produce 40 000 lines per side from 64 values, so every run and platform gets the same files.
  - An exact diff of two random orders like these needs tens of thousands of edits across 80 000 lines (an estimate). That takes orders of magnitude longer than 200 ms, even in an optimized build, so the test does not rely on a slow machine.

**The fix diff, read as new code.** No finding.
- The line sits where round 1 asked and uses the existing summary style.
- It is not a control, so it needs no `data-key` or accessible name. The diff gives it a `data-key` anyway.
- Nothing nearby contradicts it. The column region's accessible name ("…, N lines") makes no claim about differences.

**Not re-checked.** Round 1's findings 2–8 are non-blocking. The fix did not touch them, and they stay under *Follow-ups* as they were.

**What held up.** The developer did more than add the assertion round 1 asked for.
- They found that round 1's own example input, all-distinct lines, finishes in about 0.1 s. So the old test had never exercised the timeout.
- They replaced it with an input that does reach the deadline.
- Keep the assertion on `marks_unavailable`. It is what keeps this test honest.

**Before this leaves review.**
- Nothing blocks. `/test-plan` is next, at `2066559` or later.
- The suite cannot check that the line is visible in the window, and the fix session did not run the app. The manual pass should therefore cover it:
  - write the test's two inputs as the two versions of a text conflict;
  - open it;
  - confirm "Changes could not be marked here" above the unmarked columns, in light and dark theme.

## Follow-ups

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

- 2026-11-06 (refinement, Q4): a new engine story is needed — resolving a conflict after the file was edited again reconciles that edit onto the chosen version instead of refusing (applies to CLI `conflicts resolve` as well). Created as [[Story - US00010 - Resolve a conflict after the file was edited again|US00010]] (Backlog, v0.3.0).
- 2026-11-07 (code review at `73334c1`): non-blocking findings 2–8 of the review entry. They cover Back during a choice, "resolved elsewhere" when the engine stops, the wording for a deletion from shared history, the summary's author, the cost of the live note, the preview line for a text file over 1 MB, and AC 12's wording. Each is small enough for the fix session or later polish, so no ticket was created.

## References

- [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]
- [[Dispatch Mesh requirements and architecture]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]]
- [[Story - US00001 - Tray icon and status window host|US00001]]

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
