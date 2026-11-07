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

## Acceptance criteria

Status window and engine (Windows tray host; the same model code runs on Linux CI):

1. A workspace with unresolved conflicts shows a **Conflicts** tab with their count; the attention card's **Open conflicts** opens it. With none, the tab shows no count and an empty state.
2. Each list entry shows the path, text or binary (binary marked "whole-file choice"), when the conflict was recorded, and per side: author, device, save time, and which version is in the folder ("Current version · shown") and which is kept ("Other version · kept"). The current version's author comes from its contributing sessions, or reads "Shared history" when there are none. *(Kai, 2026-11-06, at planning: the list carries in-memory data only, so no Git lookup runs on each runner tick; the commit's author and time appear in the detail view, criterion 4.)*
3. CLI `status` includes the same per-conflict list fields, so CLI and window read one model.
4. **Review and choose** on a text conflict widens the window to the detail layout: current and other version side by side with line numbers, changed lines marked, and per side author, device, time and requested purpose (or "none recorded"). A current version from shared history shows the author and time of the last commit on `main` that touched the path, when that commit's content is the current version. **Show common base** reveals the base. **Back** returns to the list at compact width.
5. A text conflict where any present side is over 1 MB shows the metadata view with "Too large to compare here"; both choices remain available. (Content that is not valid UTF-8 is binary by the engine's definition and gets the binary view, criterion 6.)
6. A binary conflict shows per side the size, type, author, time and purpose. PNG, JPEG, GIF and WebP sides up to 10 MB show an image preview; SVG and all other types show metadata only. The webview receives preview bytes only through a host command keyed by conflict id and side, never a file path.
7. A side that is a deletion shows "Deleted by <author> · <time>" instead of content, and its button reads "… · deleted".
8. **Keep current version** and **Use other version** record the same resolution as `dispatch-mesh conflicts resolve <id> --choose accepted|incoming` (same engine call, same resulting record and folder content), while the tray's runner keeps running. The detail view says both versions stay kept.
9. After a successful choice the window opens the next unresolved conflict, or the list at compact width when none remain, with the confirmation line from Q8. With the next snapshot the count, attention card and tray state update; with no conflicts left the workspace leaves Needs attention for its next applicable state.
10. If the engine refuses (for example, the file has edits made after the conflict), the detail view shows the engine's reason, nothing is recorded, and both choices stay available.
11. No wording in the window uses "ours", "theirs", "accepted", "incoming" or "HEAD".
12. Host tests with real workspaces cover: list metadata for a text and a binary conflict (including one whose current version came from shared history), resolving each way from the host, the refusal path, a deletion side, and the 1 MB / non-UTF-8 fallback. Gates green.

The engine story for edits made after a conflict (Q4, Follow-ups) turns criterion 10's refusal into a reconciliation; US00006 is complete without it.

## Implementation plan

### Ground truth that shapes the plan

Checked against the code at `9b98471` (branch `us00003-exclude-device-local-files`; the conflict code is as on `main`):

- `Status.unresolved_conflicts` is built in `Workspace::status_from` (`src/status.rs`) from in-memory state, for CLI `status` and every runner snapshot (ADR-0009, ADR-0010). It must stay free of Git calls, hence the attribution decision recorded under criterion 2.
- `model::Conflict` holds base/accepted/incoming bytes, the other version's `session` and `author`, `binary`, `created_at` and `accepted_sessions` (session id → change fingerprint). Sessions stay in `state.manifest.sessions` after they settle, so a contributing session's `author`, `device`, `changes[path].last_edit` and `intents` (purpose + paths) are available in memory.
- `model::binary` = contains NUL or not valid UTF-8, for any side.
- Mesh commits on `main` carry the session author as Git author (`Git::commit_objects`), so `git log -1 <main> -- <path>` names a real author. `Git` has `text(args)` and `validate_path`.
- `Workspace::resolve(id, incoming)` captures, refuses when the folder differs from the current version, records the `Resolution`, and renders. CLI `conflicts resolve` calls exactly that.
- `Workspace::open` fails with `WorkspaceBusy` after 500 ms; `Workspace::inspect` is lock-free and read-only; `runner::capture_for_shutdown` retries `open` for 10 s.
- Tray: `tray/src/model.rs` holds all wording (pure, unit-tested); `tray/src/host.rs` owns one `RunnerHandle` per workspace and is Tauri-free; `tray/src/main.rs` exposes commands `workspaces` and `retry`; `tray/ui/app.js` renders only through `textContent`. The window is 440 × 520 (`tauri.conf.json`). The CSP already allows `img-src 'self' data:`. Capabilities grant only `core:event:default`; app commands need no extra permission.
- No diff crate in `Cargo.lock`.

### Steps, in order

1. **Shared helper: wait for the operation lock** (`src/engine.rs`, `src/runner.rs`). Add `Workspace::open_waiting(root, timeout)`, which retries `open` on `WorkspaceBusy` the way `capture_for_shutdown` does, and make `capture_for_shutdown` use it. *Why first:* both the host's resolve (step 5) and US00005's Finish session need it. US00005's plan assumes `open` waits, which it does not, so whichever ticket is developed first adds the helper and the other reuses it.
2. **Engine: list metadata in `Status`** (`src/status.rs`). `ConflictStatus` keeps `id`, `path` and `binary` and gains:
   - `created_at`;
   - `current` and `other`, each a `ConflictSide { contributors, shared_history, deleted, size }`;
   - `contributors`: a list of `{ author, device, last_saved, purposes }` from the sessions in `accepted_sessions` (current) or the conflict's `session` (other). `last_saved` is that session's `changes[path].last_edit`, and `purposes` are its intents whose paths include the conflict's path;
   - `shared_history`: `true` when the current side has no contributing session;
   - `deleted`: the side is `None`; `size`: the byte length.

   Pure in-memory work, no Git and no bytes in `Status`. JSON field names say `current` and `other`, never accepted/incoming.
3. **Engine: one conflict's detail on demand** (new `src/conflicts.rs`, an `impl Workspace` block like `src/status.rs`; `src/git.rs`). Add `Workspace::conflict_detail(id) -> Result<ConflictDetail>`: the `ConflictStatus` fields plus `base`, `current` and `other` bytes, plus `history` for the current side and the base. `history` is the author and time of `git log -1 <state.main> -- <path>`, given **only when** that commit's blob at `path` equals the side's bytes. Otherwise it is `None`, because the last commit may postdate that version and would name the wrong person. Add `Git::last_commit(reference, path) -> Option<CommitInfo>` with `validate_path`. Called on a `Workspace::inspect`, so it never waits for the runner. CLI `conflicts show` is unchanged.
4. **ADR-0010 amendment** (wiki, with steps 2–3): the new `ConflictStatus` fields. The rule that `Status` stays in-memory, with no Git or content, and that per-conflict content and history come through `conflict_detail` on demand. Written with the code so the ADR never describes a different model.
5. **Host** (`tray/src/host.rs`). All new methods run only for a workspace whose state is `Running` in this host; otherwise they return a refusal message, because another runner owns that folder.
   - `conflict_detail(workspace, conflict)`: checks that the id is in the latest snapshot's `unresolved_conflicts`, then calls `Workspace::inspect(path)?.conflict_detail(id)`.
   - `conflict_preview(workspace, conflict, side)` (`side` is `current`, `other` or `base`): returns `(mime, bytes)` only when the bytes sniff as PNG, JPEG, GIF or WebP by magic number and are at most 10 MB. Everything else gets `None`. SVG is never sniffed as an image. The webview never sends a path.
   - `resolve(workspace, conflict, choice)` (`choice` is `current` or `other`): checks the id is unresolved in the latest snapshot, then calls `open_waiting(path, 10 s)` and `resolve(id, choice == other)`. It returns `Ok` or the engine's message (`{error:#}`). The next snapshot carries the result, so no extra notification is needed.
6. **Tray model** (`tray/src/model.rs`; new dependency `similar` in `tray/Cargo.toml`), pure and unit-tested:
   - `WorkspaceView` gains `conflicts`: the list entries with all wording ("Text", "Image · whole-file choice" or "Binary · whole-file choice", "Current version · shown", "Other version · kept", "Shared history", "You" for this device's sessions as in US00005, "Deleted by <name>") plus epoch times that the window formats. It also gains the attention action `open_conflicts` when the reason is the unresolved-conflict count.
   - `conflict_detail_view(detail, this_device) -> ConflictDetailView`, which contains:
     - the side headers (contributors, purposes or "none recorded", "In the folder now" / "Kept, not shown") and the base label "Before both changes", with history when known;
     - `body`: `Text { current_lines, other_lines, base_lines }` with a changed mark per line, or `Metadata { reason }` ("Too large to compare here" when any present side is over 1 MB), or `Binary { previews }` (a preview flag per side from the same sniffing as the host, which the host exports);
     - the button labels, with the " · deleted" suffix for a deletion side;
     - the "Both versions stay kept…" line.
   - Line marks come from `similar::TextDiff::from_lines(current, other)` with a 200 ms deadline. On timeout every line is unmarked and the view says "Changes could not be marked", so a pathological file cannot hang the window.
   - `next_after(resolved, conflicts)` picks the next unresolved conflict in list order, or `None` to return to the list. `confirmation(path, choice)` returns the Q8 line.
7. **Tauri wiring** (`tray/src/main.rs`, `tray/tauri.conf.json`):
   - Commands `conflict_detail`, `conflict_preview` (returns `tauri::ipc::Response` raw bytes, an `ArrayBuffer` in JS), `resolve_conflict` and `set_layout(compact|detail)`. All except `set_layout` run on a blocking thread, so neither Git nor the lock wait touches the UI thread.
   - `set_layout("detail")` stores the current size and resizes to 1040 × 720 logical pixels, clamped to the monitor's work area; `"compact"` restores the stored size. Resizing happens in Rust, so the webview gets no window permission.
   - The CSP adds `blob:` to `img-src` for previews; the capabilities stay unchanged.
8. **Window** (`tray/ui/app.js`, `app.css`):
   - **Tabs and list:** **Activity** | **Conflicts N** tabs, with the activity list from US00005. The attention card's **Open conflicts** switches to the Conflicts tab. Conflict cards follow design page 6, with an empty state.
   - **Detail view:** design pages 7–8. "‹ Conflicts / n of N", path, two panes with line numbers and ≠ marks, **Show common base** toggle, image previews via `URL.createObjectURL` (revoked when leaving), metadata table, and a footer with the buttons.
   - **Resolving:** while a resolve is pending both buttons are disabled with a busy label. A returned error is shown inline and the buttons are enabled again. On success the view moves on per `next_after` and shows the confirmation line. If an open conflict disappears from the snapshot (resolved by CLI or another device), the window returns to the list with "This conflict was resolved elsewhere."
   - **Rules:** text only through `textContent`; every control has a `data-key` and an accessible name; changed lines carry a text mark (≠), not only a tint.
9. **Docs:** README (`status` conflict fields; the tray section now lists conflict review), the AGENTS.md repository map (tray no longer "status only"), and the ADR-0010 amendment (step 4).

**Order and why.**
- **After US00005.** Develop this after US00005 is merged: both extend `Status`, `WorkspaceView` and `app.js`, and US00005's activity list becomes the Activity tab here. Starting first would mean building the tabs twice.
- **Engine before host, host before UI.** Steps 2–3 before 5 because the host's tests need real conflict data. Step 5 before 8 because the window has no logic of its own to test against.
- **Step 1 can land in either ticket**, as explained above.

### Tests

- **Engine** (`tests/engine.rs`, `tests/cli.rs`):
  - `status` lists a two-device text conflict with both sides' authors, devices, `last_saved` and an intent-scoped purpose;
  - a binary conflict with sizes;
  - a delete-vs-edit conflict with `deleted`;
  - a conflict whose current version came from `main` (Alice publishes; Carol, offline on the old base, edits the same line; Bob, with no own session, receives Carol's draft): `contributors` is empty and `shared_history` is true. If that setup does not produce an empty `accepted_sessions`, record why in the as-built notes and cover the case with a constructed state;
  - `conflict_detail` returns the exact bytes, and `history` only when the commit content matches;
  - `open_waiting` succeeds after a held lock is released and gives `WorkspaceBusy` after its timeout;
  - CLI `status` keeps every existing key.
- **Tray model** (unit tests):
  - wording for list entries and side headers, including "You", "Shared history", "Deleted by …" and the "· deleted" suffix;
  - the 1 MB boundary (exactly 1 MB compares, one byte more does not);
  - image sniffing: PNG, JPEG, GIF and WebP yes; SVG, a renamed text file and 10 MB + 1 byte no;
  - line marks for the retention sample;
  - `next_after` at the start, middle and end of the list, and the confirmation line;
  - an assertion that no model string contains "ours", "theirs", "accepted", "incoming" or "HEAD" (criterion 11).
- **Host** (`tray/tests/host.rs`, with a local bare remote and two workspaces):
  - resolve each way from the host while its runner runs, and the folder and record equal what CLI `conflicts resolve` produces;
  - the refusal path: an edit after the conflict returns the engine's message and records nothing;
  - all three methods are refused for a workspace held by another runner, and for an unknown conflict id;
  - `conflict_preview` returns `None` for a non-image.
- **Newly assertable:** conflict attribution per side becomes engine-level state checked by tests instead of something only `conflicts show` hints at.
- **Manual** (for `/test-plan`): widening and restoring on Windows at 100 % and 150 % scaling and on a small screen; image previews; dark mode; Narrator on tabs, panes and buttons; and a resolve made in the window being picked up by a second client (the full two-device check is TASK00003).
- **When US00010 lands**, the host's refusal-path test turns into a reconciliation test. That change belongs to US00010.

### External changes and migration

- `status` JSON: `unresolved_conflicts[]` gains fields; the existing ones are unchanged. No state or synchronized-format change: everything is derived from existing records.
- New dependency `similar` (tray only, MIT/Apache-2.0). The CSP adds `blob:` to `img-src`.

### Wiki pages this makes wrong

- [[ADR-0010 — Report workspace status and session stages separately]]: amended in step 4.
- README and AGENTS.md: step 9.
- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]'s plan states that `Workspace::open` waits for the lock. It does not (see step 1). That ticket's owner corrects it at its own sign-off; this ticket does not edit it.

### Risks and rollback

- **Window resizing** across DPI settings and monitors: clamped to the work area, covered by manual checks.
- **Large IPC payloads** (a 10 MB image per side): sent as raw bytes, never base64 in JSON, and only on demand.
- **Diff cost:** capped by the 1 MB limit and the 200 ms deadline.
- **Lock wait:** a resolve during a long local operation waits up to 10 s with a busy label, then shows `WorkspaceBusy`'s message. Nothing is recorded in that case.
- **Stale detail:** the engine's own guard rejects a resolve against changed content, and a conflict that vanished sends the window back to the list.
- **Rollback:** revert the commits; no data or format migration either way.

### Decisions for review

1. The list shows "Shared history" without author or time; the detail adds the commit (Kai, 2026-11-06).
2. Commit history is shown for the current version and the base only when the commit's content matches, never "the last commit that touched the file" on its own.
3. The line diff runs in the tray (`similar`, 200 ms deadline), not in the engine.
4. Previews travel as raw IPC bytes into `blob:` URLs, never as file paths. Image type comes from magic bytes, not the file extension.
5. `open_waiting` with a 10 s limit, shared with the runner's shutdown capture and US00005's Finish session.
6. Detail and resolve are only offered for workspaces this host runs.
7. The detail layout is 1040 × 720 logical pixels, clamped, and the compact size is restored on Back.

### Size

Order of magnitude: about five to seven focused days, roughly two for the engine and host with tests, and three to five for the model, wiring and window. Not a commitment.

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
