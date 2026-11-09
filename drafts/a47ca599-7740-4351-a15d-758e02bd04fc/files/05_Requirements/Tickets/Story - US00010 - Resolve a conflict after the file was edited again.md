---
id: US00010
type: story
status: Refinement
priority: high
rank: 4500
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings:
discussion:
updated: 2026-11-09
completed:
frozen:
---
Today, choosing a version for a conflict fails with "File has newer edits; review them before resolving this conflict" when the file was saved again after the conflict was recorded. Nothing offers a way out except restoring the current version by hand. With this change, resolving is allowed. The later edit counts as a change on top of the current version and is three-way merged onto the chosen version: a clean merge is applied, and overlapping edits become a new conflict. Neither the later edit nor either conflict input is lost. This applies to CLI `conflicts resolve` and to the status window ([[Story - US00006 - Conflict list and resolution in the status window|US00006]]), which call the same engine operation.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a collaborator who kept working on a file after Mesh recorded a conflict in it, I want to choose a version without first undoing my newer edit, so that my newer work is kept and combined with the version I chose.

## Context

- Origin: [[Story - US00006 - Conflict list and resolution in the status window|US00006]] Q4, answered by Kai on 2026-11-06: "the engine reconciles … built as a separate engine story that US00006 depends on". US00006 is complete without this story; until this story lands, its window shows the engine's refusal (US00006 acceptance criterion 10).
- The [[Claude Design briefing — desktop app]] §6.3 item 4 and the conflict-detail frame in [[Dispatch Mesh desktop app.pdf]] promise this: "You saved this file again at 11:20, after the conflict was recorded. That edit is protected and reconciled separately once you choose."
- [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]: a resolution selects one of the recorded inputs against their contributing revisions, and later distinct edits are reconciled separately. AGENTS.md states the same invariant. This story makes that hold for a *local* later edit at resolve time, not only for publication order.
- **Engine today (verified 2026-11-06):**
  - `Workspace::resolve(id, incoming)` (`src/engine.rs`, around line 1414) calls `capture_at(now())`. Its guard `state.observed[path] == conflict.accepted` then refuses when the folder differs from the current version. Otherwise it inserts a `Resolution { selected, author, created_at }` and calls `render()`.
  - `capture_at` captures saved edits into the local session as a `Change { base, content }`. The base of a later edit is the folder view the user edited, which is the current version, and the change merges with an existing open change via `git.merge` (line ~492).
  - `merge_change` / `matching_resolution` (line ~984 onward) let a recorded resolution govern the original inputs whatever order they are published in. A conflict is created through `state.conflicts.entry(..).or_insert_with(..)` with `accepted_sessions` holding the contributing revisions.
  - No test covers the refusal path. Resolve tests: `tests/engine.rs` lines ~149, ~370, ~437.
- **Engine re-read during refinement (2026-11-09):**
  - `resolve` is now at `src/engine.rs` ~1536 and the guard at ~1549. The guard ignores the choice: "keep current" is refused as well as "choose incoming".
  - `render()` rebuilds the folder from `main` plus every unsettled session. A later edit captured as its *own* change (base = current version) is therefore already three-way merged onto the selected version by `merge_change` once a resolution exists. On overlap, `merge_change` records a new conflict with the selected version on the current side and keeps that current side in the folder, as ADR-0003 requires ("keep the accepted view visible").
  - If the user's open session already holds their conflict input for the path, `capture` folds the later edit into that same change (`git.merge`, ~540). The change's fingerprint then no longer matches `accepted_sessions`, so `matching_resolution` stops recognising it as the recorded input. Nothing prunes `state.conflicts`. This is the core of Q4.
  - While the conflict is open, `publish_ready` skips every session that contains the path, so both contributors' sessions (including their other files) stay unpublished.
- CLI: `dispatch-mesh conflicts resolve <id> --choose accepted|incoming` (`src/main.rs`).
- **Order relative to [[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]] (Kai, 2026-11-09):** US00010 comes first. US00004 does not make this story unnecessary, for three reasons:
  - It addresses a different problem: two recorded inputs Git cannot merge, versus a third version saved after the conflict.
  - Whole-version choice stays its fallback for binary files, for users without AI consent (ADR-0005), and as one of its review paths. A later edit blocks all of these today.
  - US00004's own scope needs a resolution that "preserve[s] later edits" (its Q7).
  
  Design consequence: build the reconciliation as "the resolved content plus a later change", not as "a selected input plus a later change". US00004 can then reuse it unchanged with an approved composed text as the resolved content.

## Scope

- `Workspace::resolve` no longer refuses when the file changed after the conflict was recorded. The later local edit is reconciled onto the selected version: a three-way merge with the current version as base, the later edit as one side and the selected version as the other.
- A clean merge is written to the folder and stays the local user's change, in their session with its own attribution and purpose. It never becomes part of the resolution record.
- Overlapping edits show the selected version in the folder on every device and record a new conflict: the current side is the selected version (original author), the other side is the later edit (local user's session). The later edit is never discarded. The original conflict is still marked resolved with the user's choice (Q2, Kai, 2026-11-09).
- The resolution record keeps selecting exactly one recorded input (ADR-0003); synchronization and publication order behave as today.
- A save to a path with an unresolved conflict whose input belongs to the active session closes that session and starts a new one (Q4).
- Later changes received from other devices are reconciled the same way as local ones (Q5).
- CLI wording: `conflicts resolve` reports whether a later change was kept (current version chosen), combined, or produced a new conflict. It names the later change's author, and it words the case where the chosen version deletes the file separately (Q1, Q3, Q5).
- Status window, the later-edit part only. The design spec assigns this to US00010 ("US00010 enables them and switches to …"). Kai decided on 2026-11-09, while planning, to include it here rather than in a follow-up ticket, so that the window never disables both buttons with text that is no longer true. Changes:
  - **Buttons:** both stay enabled after a later edit.
  - **Note:** the later-edit note becomes "You saved this file again at 11:20, after the conflict was recorded. That edit is protected and reconciled separately once you choose."
  - **Confirmation:** after resolving, it says whether the later change was kept, combined or became a new conflict.
- Regression tests with real workspaces for each outcome, for both choices, and across two devices with either publication order.

## Out of scope

- Conflict UI beyond the later-edit change below: the list, detail view layout and other wording in the status window belong to [[Story - US00006 - Conflict list and resolution in the status window|US00006]].
- Writing a combined third version by hand, and AI-proposed merges ([[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]]).
- Changing how conflicts are detected or recorded in the first place.
- Repairing workspaces whose session already folded a later edit into a recorded conflict input before this release (Q4, Kai, 2026-11-09; the console is a preview and no such workspace is known).

## Open questions

Recorded here: no tracker or chat is configured.

- [x] **Q1 Keep current version with a later edit.** Choosing the current version is a no-op on the folder, since the later edit is already based on it. Is that the whole behavior: the later edit simply stays as the user's next change?
  - **Answer (Kai, 2026-11-09):** Yes. The folder stays unchanged, the later edit remains an ordinary change of the local user (own attribution and purpose), and the conflict is marked resolved with the current version selected. `conflicts resolve` reports that the later edit was kept, alongside "combined" and "new conflict" for the other choice. Rejected alternatives: staying silent, because the two choices would then report differently and the protected edit would go unconfirmed; folding the later edit into the resolution, because the resolution would select a version that is not a recorded input (ADR-0003) and new work would be mixed into a record that holds another author's input. Precondition: the later edit must stay a change separate from the recorded conflict input (see Q4). Otherwise the next `render()` can merge the other input against the later edit and record a fresh conflict right after "keep current".
- [x] **Q2 Overlapping later edit.** When the merge overlaps, which version stays in the folder (proposal: the later edit, never overwritten), and how are the new conflict's sides attributed? Proposal: the current side is the local user's later-edit session, and the other side is the selected version with its original author.
  - **Answer (Kai, 2026-11-09):** The **selected version** shows in the folder on every device, not the later edit. The new conflict's current side is the selected version, attributed to its original author through `accepted_sessions`; its other side is the later edit, attributed to the local user's later-edit session. Example: Alice chooses Bob's `Meet at 10:00 at the main entrance.` while her later edit says `Meet at 9:15 at the station.` Every device shows Bob's line, and the new conflict holds Alice's 9:15 line as the other side. Why: every device renders the same folder, so keeping the later edit would put text the chooser did not choose in front of the other author (Bob would lose the line he was chosen for). Showing the selected version matches today's `merge_change` behavior and ADR-0003 ("keep the accepted view visible"). Accepted cost: the chooser's later text leaves the open file and is recoverable from the new conflict. The CLI (and later the US00006 window) says so.
- [x] **Q3 Binary files and deletions.** A later edit to a binary file cannot be merged, and either side may be a deletion. Does choosing the other version then always produce a new whole-file conflict, or does the later edit win?
  - **Answer (Kai, 2026-11-09):** The Q2 rule applies unchanged to every file type, with no special case. This follows `git.merge` (`src/git.rs` ~398): when neither side equals the base, a binary file or a deletion always conflicts. Outcomes:
    - Binary later edit, other version chosen: a new whole-file conflict; the folder shows the chosen file and the later edit is the other side.
    - Binary later edit, current version kept: no-op (Q1).
    - Later edit is a deletion, other version chosen: a new conflict; the folder shows the chosen file and "deleted" (local user) is the other side.
    - Chosen version is a deletion while the later edit modified the file: a new conflict; the file is removed from the folder and the later edit is the other side.
    - Both are deletions: clean; the file stays deleted.
    
    The case where the chosen version deletes the file gets its own CLI wording (the chosen version deletes the file; the later edit is kept as a new conflict) and its own regression test. Rejected: letting the later edit win for these cases, because that overrides the user's choice for some file types only and contradicts Q2.
- [x] **Q4 Session state of the later edit.** The later edit may sit in an open session, a closed one waiting for finalization, or one already published, since finalization waits only for sessions containing an unresolved path. How is its change rebased onto the selected version in each case, so that publication on `main` is exactly "selected version → later edit" and no empty or duplicate commit appears?
  - **Findings (2026-11-09):**
    - *Already published* cannot occur: while a conflict is unresolved, `publish_ready` skips every session that contains its path.
    - *Separate session* (open or closed): already handled. `render()` and, at publication, `merge_change` three-way merge the later change onto the selected version. The later session depends on the contributors' sessions, so it publishes after them.
    - *Same open session as the user's recorded conflict input*: broken today. `capture` folds the later edit into the recorded input, its fingerprint changes and `matching_resolution` no longer recognises it.
  - **Answer (Kai, 2026-11-09):** **Split at capture.**
    - **Rule:** when a save touches a path with an unresolved conflict whose current or other input belongs to the active session, Mesh closes that session first and captures the save into a new session (the mechanism `finish` uses). The recorded input stays byte-exact from the moment the conflict exists, and the later edit becomes its own change with the conflict's version as base.
    - **Example:** Alice's open session holds `trip.md` (her input) and `notes.md`, and she adds `Bring tickets.` to `trip.md` at 11:20. If she then chooses Bob's version, three commits publish in order: her first session with only `notes.md` (its `trip.md` input was rejected), Bob's session, then her new session with `Bring tickets.` None is empty or duplicated.
    - **Accepted cost:** the user's work lands in two commits, and the split happens even if the conflict is never resolved.
    - **Rejected:** splitting at resolve time. Until the resolve, the folded change keeps `render()` wrong: it records a second conflict and leaves the original unresolvable.
    - **Not covered:** workspaces that already hold a folded change from before this release (see Out of scope).
- [x] **Q5 Received changes after the conflict.** The guard compares against the folder, which also holds content received from other devices after the conflict. Is a received later change reconciled the same way, or is only a local edit in scope?
  - **Answer (Kai, 2026-11-09):** **The same way as a local edit; in scope.** A received later change is another device's session and therefore already a separate change. After the resolution, `render()` merges it onto the selected version. A clean merge is applied; an overlap records a new conflict with the selected version showing (Q2, Q3). If the other author's later edit would fold into their own session holding their conflict input, the Q4 split happens on their device at capture, since every device runs the same rule. Example: Bob adds `Bring snacks.` to `trip.md` at 11:15 after C1 was recorded; Alice can resolve C1 and Bob's line is combined with her choice. `conflicts resolve` names the author of the later change ("Bob's 11:15 edit was combined"). Rejected: local edits only, because the guard would then need new code to tell local from received changes, solely to refuse, and the chooser would depend on the other author undoing their edit.
- [x] **Q6 Compatibility.** Does this need any change to the synchronized conflict or resolution format? If it does, how do clients that don't have the change yet behave, given that they still refuse? (Expectation: no format change, because the resolution stays a choice between inputs and the reconciliation is an ordinary session change.)
  - **Answer (Kai, 2026-11-09):** **No format change.** The `Resolution` record is unchanged and still selects one recorded input. New conflicts use the existing `Conflict` record and its deterministic id. Split sessions are ordinary sessions. The manifest stays `version: 2`. Mixed versions interoperate: an older client renders the same folder from the same records. For its own later edits, it keeps today's refusal and its capture still folds (Q4) until it is upgraded; no data is lost. The release note states this. Rejected: enforcing a minimum client version, because it needs a version handshake that does not exist (itself a format change) and would block old devices over a bug that loses no data.

## Acceptance criteria

Terms: the *current* version is the one in the folder when the conflict was recorded; the *other* version is the one kept aside. A *later change* is any save to the conflicted path after the conflict was recorded, local or received.

1. With a later change that does not overlap the other version, `conflicts resolve <id> --choose incoming` succeeds. The folder shows the other version with the later change merged in, the conflict is resolved with the other version selected, and the CLI reports that the later change was combined.
2. With any later change, `--choose accepted` succeeds and leaves the folder unchanged. The conflict is resolved with the current version selected, the later change remains an unpublished change of its author, and the CLI reports that it was kept.
3. With a later change that overlaps the other version, `--choose incoming` succeeds and the original conflict is resolved with the other version selected. On every device, the folder shows the other version, and a new conflict lists the other version as current (attributed to its original author) and the later change as other (attributed to its author). The CLI reports the new conflict.
4. For a binary file with a later change, choosing the other version yields a new whole-file conflict as in 3. The later file is retrievable byte for byte.
5. When the chosen version is a deletion and the later change modifies the file, the file is absent from the folder and a new conflict holds the later change. The CLI says that the chosen version deletes the file and the later change was kept as a new conflict. When both are deletions, the file stays deleted and no conflict is recorded.
6. When the later change is a deletion and the other version is chosen, the folder shows the other version and a new conflict has "deleted" as its other side.
7. Saving a conflicted path whose current or other input belongs to the device's open session closes that session and records the save in a new session. The recorded input's bytes are unchanged, and no further conflict is recorded for that path until the user resolves.
8. After resolving, publication on `main` contains the contributors' sessions, then the later change's session, attributed to its author. No published commit is empty, and none repeats content already on `main`.
9. A later change received from another device is reconciled as in 1–6, and the CLI names its author.
10. With two devices, for both choices and either publication order of the inputs, both devices end with the same folder content and the same unresolved-conflict list.
11. The synchronized manifest stays `version: 2` with unchanged `Resolution` and `Conflict` records. A client built without this change, syncing with one that has it, shows the same folder.
12. In the status window's conflict detail, a later save shows the note "You saved this file again at <time>, after the conflict was recorded. That edit is protected and reconciled separately once you choose." Both **Keep current version** and **Use other version** stay enabled. After choosing, the confirmation says whether the later change was kept, combined or became a new conflict, and in the last case the new conflict appears in the list.
13. Regression tests with real workspaces in `tests/engine.rs` cover 1–11, including the previously untested path where the file changed before resolving. Tray model tests cover 12. `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` pass.

## Implementation plan

*Written 2026-11-09 against `main` at `c48141b`, after refinement (Q1–Q6) and the planning decision to include the window's later-edit change (Scope).*

### Ground truth that shapes the plan

- **Resolve** (`src/engine.rs` ~1536):
  - It captures, then refuses both choices when `state.observed[path] != conflict.accepted` (~1549). Otherwise it inserts a `Resolution`, saves and calls `render()`.
  - It returns `Result<()>`, so callers can't tell what happened to a later change.
- **Render** (~988):
  - It rebuilds the folder from `main` plus every unsettled session, in `created_at` order, gated by `dependencies`.
  - `merge_change` (~1076) already three-way merges a later change that lives in its own session onto the resolved content. On overlap it records a conflict whose current side stays in the folder (Q2), and it rejects inputs the resolution did not select.
  - So no new merge code is needed. What is missing is keeping the later change *separate* (Q4) and reporting the outcome.
- **Capture** (~463):
  - It reuses the device's open session.
  - For a path the session already changed, it applies only the new delta to the existing change (`git.merge`, ~540). On failure it stops with `DependentEditsPreserved` / `RECEIVED_DRAFT_DEPENDENCY`.
  - A new session's `dependencies` are all other unsettled sessions (~484), so a split-off session publishes after the session it was split from.
- **`matching_resolution`** (~1156) identifies inputs by `conflict.session` plus exact base/content (other side), or by the fingerprint in `accepted_sessions`, with a legacy inference for empty `accepted_sessions` (current side).
- **Publication.** `publish_ready` (~1352) skips sessions that hold an unresolved path and turns a session with no effective change into a discard (`discard_empty_projection`) instead of an empty commit.
- **Callers of `resolve`:**
  - CLI `conflicts resolve` (`src/main.rs:176`) prints a fixed sentence.
  - `Host::resolve_conflict` (`tray/src/host.rs` ~392) builds `conflicts::confirmation(path, choice)` (`tray/src/model/conflicts.rs:811`).
- **Window:**
  - `later_edit_note` (`tray/src/model/conflicts.rs:655`) says choosing works once the file matches again.
  - `detail_view` sets `can_choose: note.is_none()` (~771), and `tray/ui/app.js:454` sets `can_choose = !note` on refresh.
  - `Workspace::edited_after` (`src/conflicts.rs:130`) stays the source of the note's time.
- **Tests:**
  - `tests/conflicts.rs::detail_reports_a_later_save_and_resolving_is_then_refused` asserts the refusal.
  - `text_conflict` (`tests/conflicts.rs:58`) builds a two-device text conflict on Bob. Bob's open session holds the current side, which is exactly the Q4 case.
  - `tests/engine.rs` ~370 and ~420 hold the publication-order tests to extend.
- **Docs that become wrong:**
  - `README.md:147` ("disables both buttons … reconciling it is planned separately").
  - Design spec `06_UX/… design spec.md` lines 55 and 226 ("until US00010").
  - [[Resolving conflicts after later edits]] ("planned").

### Steps, in order

1. **Failing tests first** (`tests/engine.rs`, `tests/conflicts.rs`). Move `text_conflict` into a shared `tests/common/mod.rs` so both files can use it, and add a binary variant and a deletion variant. Write the AC 1–11 scenarios (see *Tests*) and the Q4 fold reproduction: Bob saves again in his open session, syncs, and the conflict list must not gain a second entry for `note.md`. These fail today, either with the refusal or with the fold. Writing them first pins down the expected outcomes before the engine changes.
2. **Capture: split instead of fold** (`src/engine.rs`, `capture`).
   - **Rule:** before choosing the session id, check each changed, non-excluded path. If an unresolved conflict on that path has an input in the active session, close the active session first: `closed = true`, `discarded = changes.is_empty()` (the same as `finish`). The save then opens a new session, whose `dependencies` include the closed one.
   - **Helper** `fn holds_conflict_input(&self, session: &Session, path: &str) -> bool`:
     - `conflict.session == session.id`;
     - or `accepted_sessions` contains `session.id`;
     - or, with empty `accepted_sessions`, the same legacy inference `matching_resolution` uses.
     
     One helper, so capture and resolution agree on what an input is.
   - **Why before step 3:** without the split, removing the guard turns the most common case into a wrong result. That case is the user's own open session holding their side. The resolution would no longer match the folded input, and `render()` would record a second conflict.
   - **Side effect, intended:** a save to a conflicted path no longer hits the `RECEIVED_DRAFT_DEPENDENCY` fold failure for that path.
3. **Resolve: no guard, report the outcome** (`src/engine.rs`, plus the type in `src/conflicts.rs`, which already holds the conflict presentation API).
   - Delete the guard. `resolve` returns `Result<ResolveOutcome>`:
     ```rust
     pub enum ResolveOutcome {
         NoLaterChange,
         Kept { later: Vec<LaterChange> },              // current version chosen
         Combined { later: Vec<LaterChange> },
         NewConflict { id: String, later: Vec<LaterChange>, selected_deletes: bool },
     }
     pub struct LaterChange { pub author: Identity, pub device: String, pub saved_at: u64 }
     ```
   - **Computing it:**
     - `later` is every unsettled session with a change on the path that is not an input of this conflict (by the step 2 helper).
     - With no later change, the outcome is `NoLaterChange`, which keeps today's wording.
     - Otherwise: `Kept` if the current version was chosen; `NewConflict` if `render()` left an unresolved conflict on the path that was not there before (`selected_deletes` when the selected version is `None`); else `Combined`.
     - Everything is computed from engine state, never from guessing at the folder, so a received change (Q5) is covered too.
   - **Failures:** `render()` errors as today, for example "New local edits arrived during synchronization". The resolution is already saved, as today.
4. **CLI wording** (`src/main.rs:176`). Print one sentence per outcome and keep the "Run sync …" tail:
   - `NoLaterChange`: today's sentence.
   - `Kept`: "Kept the current version; your 11:20 edit stays as your next change."
   - `Combined`: "Bob's 11:15 edit was combined with the chosen version."
   - `NewConflict`: "Your 11:20 edit overlaps the chosen version; it is kept as new conflict <id>."
   - `NewConflict` with `selected_deletes`: "The chosen version deletes this file; your 11:20 edit is kept as new conflict <id>."
   
   Use "your" for the local identity and the author's name otherwise. Times are local `HH:MM`, like the window.
5. **Window** (`tray/src/model/conflicts.rs`, `tray/src/host.rs`, `tray/ui/app.js`).
   - **Note:** `later_edit_note` gets the design wording: "You saved this file again at {TIME}, after the conflict was recorded. That edit is protected and reconciled separately once you choose."
   - **Buttons:** `detail_view` sets `can_choose: true`. Keep the field rather than removing it, so `app.js` still has one switch. In `app.js:454`, stop deriving `can_choose` from the note.
   - **Confirmation:** `confirmation(path, choice, &outcome)` appends the step 4 sentence after the existing text, so the two surfaces word it the same way. A shared formatter lives in the tray model; the CLI keeps its own short `format!`, because the engine crate must not depend on the tray. `Host::resolve_conflict` passes the outcome through.
   - **New conflict:** after `NewConflict`, `next_after` already moves to the next conflict, and the new conflict appears in the list on the next snapshot. No extra plumbing.
6. **Docs** (same branch):
   - `README.md:147`: buttons stay enabled and the later change is reconciled. Link the three outcomes. Also add a line to the CLI `conflicts resolve` section.
   - Design spec lines 55 and 226: drop "until US00010" and keep only the post-US00010 wording. Owner: Kai, updated by the developer in the same change.
   - [[Resolving conflicts after later edits]]: switch "planned" to as-built, update line references, set `verified`/`source_revision`.
   - Each wiki change gets a `log.md` entry.
7. **Gates:** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test`, including the tray crate's tests.

### Tests (which layer asserts what)

- **`tests/engine.rs`**, real two- and three-device workspaces over a bare remote:
  - **AC1:** later non-overlapping edit, choose other → folder = merge, conflict resolved, `Combined`.
  - **AC2:** later edit, choose current → folder unchanged, `Kept`, the later session unsettled and owned by the local author.
  - **AC3:** overlap → every device's folder = selected; a new conflict with current = selected (`accepted_sessions` names the original author's session) and other = the later edit (`session` = the later session); `NewConflict`.
  - **AC4–6:** binary, chosen deletion, later deletion, both deleted.
  - **AC7:** the split at capture. The original session is closed with its input byte-exact (fingerprint unchanged), and syncing before resolving adds no conflict.
  - **AC8:** publication. Both inputs then the later session; after all devices finish and sync, `git log` on `main` has no empty commit and no commit that changes nothing, and the later commit's author is the later editor. Both choices and either publication order, extending the ~370/~420 pattern.
  - **AC9:** Bob's later edit received by Alice → `Combined` naming Bob.
  - **AC10:** both devices converge on folder content and unresolved ids.
  - **AC11:** `manifest.version == 2` after the whole flow; `Resolution` and `Conflict` serialize with exactly today's keys (a serde snapshot).
- **`tests/conflicts.rs`:** rename the refusal test to `detail_reports_a_later_save_and_resolving_reconciles_it`. `edited_after` is still reported, and resolve now succeeds.
- **`tests/cli.rs`:** one case per wording branch of `conflicts resolve`.
- **Tray model unit tests** (`tray/src/model/conflicts.rs`): note wording, `can_choose` true with a note, `confirmation` for each outcome (AC12).
- **Newly assertable:** the outcome type makes "what happened to my later edit" testable at the engine boundary. Before, it could only be inferred from the folder.
- **Left to the manual test plan:** a mixed-version check, a 0.2.5 client from `dist/` syncing with this build (AC11's second half; there's no way to run an old binary in `cargo test`), and the window flow.

### Risks and mitigations

- **More sessions and commits.** A conflict now cuts the user's session when they keep editing the path. Later work lands in its own commit with its own AI summary. Accepted in Q4. Existing tests that keep editing a conflicted path in one session may change their expectations. Step 1 surfaces them.
- **Dependency ordering.** The split-off session depends on the closed one through `dependencies`, so it can't publish first. AC8 asserts the order rather than relying on `created_at`.
- **Legacy conflict records** (empty `accepted_sessions`). The helper reuses `matching_resolution`'s inference, so old records split the same way. Repairing already-folded workspaces is out of scope.
- **Outcome misreport.** `NewConflict` detection compares the unresolved ids on the path before and after `render()`. A conflict that existed before on the same path (a different C2) isn't counted as new.
- **Rollback.** No format change, so reverting the code is safe. Sessions split by this build stay ordinary sessions that an older build publishes normally.

### Order that matters, and why

- Tests (1) come before code, because the Q4 fold is invisible until a test syncs after the later edit.
- The capture split (2) comes before removing the guard (3), because removing the guard alone gives wrong results in the most common case.
- The engine outcome (3) comes before the CLI (4) and the window (5), because both only format it.
- Docs (6) ship in the same change, because the README and design spec otherwise describe the disabled buttons the build no longer has.

### Size

Order of magnitude: roughly 150 lines of engine change, 50–80 in the tray, and 400–600 lines of tests. One development session plus review.

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

- [[Story - US00006 - Conflict list and resolution in the status window|US00006]] (origin, Q4)
- [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]] (conflict detail frame)
- Code: `src/engine.rs` (`resolve`, `capture_at`, `merge_change`, `matching_resolution`), `src/main.rs` (`conflicts resolve`), `tests/engine.rs`

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
