---
id: US00003
type: story
status: In progress
priority: medium
rank: 2000
version_target: v0.3.0
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings: 2
discussion:
updated: 2026-11-06
completed:
frozen:
---
**Let a user stop synchronizing files that belong to one device, above all Obsidian's `.obsidian/workspace.json`, even after Mesh has already synchronized them.** Today `.meshignore` and `.gitignore` are honored only for files that were never synchronized. Adding a rule for a synchronized file stops capture with "A previously synchronized path is now ignored or unsupported … Restore the ignore policy before syncing". So the one file that most needs excluding can't be excluded once it's been shared. Meanwhile it changes whenever the user opens a note or moves a pane, and it fills the history with layout commits.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As someone who syncs an Obsidian vault with Mesh, I want to exclude files that only describe my own device's state, such as the open tabs and pane layout in `.obsidian/workspace.json`, so that:
- my history and session summaries show my real edits;
- my devices don't overwrite each other's layout or raise conflicts over it;
- clicking around in Obsidian doesn't keep a session open.

## Context

- **Source:** Kai, 2026-11-05, from running this wiki on Mesh (`/create-ticket Exclude files from update - especially .obsidian/workspace.json`).
- **Evidence from this vault:** the wiki workspace has no `.meshignore` or `.gitignore`, so `.obsidian/workspace.json` is synchronized.
  - **31 of 472 commits** in the workspace's history touch it.
  - Their AI summaries describe layout, not content, for example *"Update Obsidian workspace to focus on desktop app design"* and *"Update workspace to focus on tray host story"*.
- **Why the file is a problem beyond the noise:**
  - **Sessions:** every focus change is a save, and sessions close after five minutes without edits ([[ADR-0002 — Group files into sessions with an inactivity boundary|ADR-0002]]), so navigating keeps a session open and delays finalization.
  - **Conflicts:** with two devices the file is edited on both and becomes a recurring conflict ([[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions|ADR-0003]]), even though neither side's version is meant for the other device.
  - **Not all of `.obsidian/` is device-local:** `.obsidian/plugins/dispatch/data.json`, `app.json` and `community-plugins.json` are shared vault settings and must keep syncing. Obsidian's own guidance treats `workspace.json` and `workspace-mobile.json` as per-device.
- **What the code does today:**
  - `Workspace::scan` (`src/engine.rs`) walks the folder with `.gitignore` and `.meshignore`, plus fixed excludes for `.mesh`, `.git`, `.env`, `.env.*`, `*.pem` and `*.key`.
  - It then refuses when a path in `state.observed` still exists on disk but is now ignored. That refusal is the guard that stops an ignore rule turning into a silently recorded deletion.
  - `install` refuses an incoming path that the local policy ignores ("Synchronization paused without changing the file").
- **What the README says:** *Preview boundaries* says to use matching ignore policies on all clients, and that "changing ignore rules for already synchronized files is not supported yet". *Recovery procedures* lists the newly-ignored path as a `needs_attention` cause.
- **Milestone plan:** [[Dispatch Mesh milestone plan|0.2.x]] asked to document ignore-policy behavior and make unsupported cases visible without discarding work. TASK00001 did that, and this ticket closes the gap it documented. The tray (0.3) and guided setup (0.5) are where a default for Obsidian vaults would surface to non-technical users.

## Scope

Sharpened in refinement on 2026-11-06 (decisions under *Open questions*):

- **Stop synchronizing a path that was already synchronized.** The shared `.meshignore` (and `.gitignore`) are the only policy. When capture finds a previously synchronized path that is now ignored but still on disk, it records an explicit, attributed removal in the current session instead of refusing. The file stays on disk, and capture and sync continue. The removal reaches `main` with the session, so a device that joins later doesn't receive the file.
- **Receiving the removal:** a device that receives the removal, with a policy that ignores the path, leaves its local file untouched and stops tracking it.
- **Incoming changes to an excluded path** (a peer that hasn't received the rule yet) are dropped silently. The local file is not touched, sync doesn't pause, nothing appears in `status`, and on `main` the removal wins. This replaces today's "Incoming path is excluded by the local ignore policy" pause, for paths the local policy ignores.
- **Pending work:** the removal replaces the current session's pending edit to the path. Earlier unsettled sessions finalize normally. An open conflict on the path is settled by the removal as a traceable ADR-0003 resolution that keeps both inputs.
- **Re-including:** removing the rule makes local copies ordinary new changes (clean merge or ordinary conflict). No special case and no overwrite.
- **Obsidian default:** `init` writes or extends a visible `.meshignore` with `.obsidian/workspace.json` and `.obsidian/workspace-mobile.json` when `.obsidian/` exists. Nothing is built into the engine, and existing workspaces are not edited automatically.
- **README:** replace the "not supported yet" boundary and the newly-ignored `needs_attention` recovery entry. Describe the new behavior, show the two Obsidian lines for existing vaults, and warn that engines before 0.3.0 move a received excluded file into `.mesh/recovery`, so every device must be updated first.
- **Tests:** engine and CLI tests for every acceptance criterion below, including two devices.

## Out of scope

- A tray or settings UI for editing excludes. The tray shows status only until later 0.x milestones.
- Rewriting history to remove a file's past versions from the remote.
- Selective sync per device for whole folders, e.g. "don't download `01_Sources/` on the phone".
- Changing the fixed secret excludes (`.env`, keys).
- A CLI command for adding excludes (decided 2026-11-06: editing `.meshignore` is the interface).
- Device-local exclude lists (decided 2026-11-06: only the shared `.meshignore` is in scope).

## Open questions

- [x] **What "stop synchronizing" means for the shared copy.** When a device excludes a synchronized file, does `main` keep the file's last version, frozen and untracked, or does `main` record its removal while every device keeps its local copy? The second keeps the repository clean but is a deletion on GitHub.
  - **Answer (Kai, 2026-11-06):** `main` records the removal; every device keeps its local file. The device whose capture first sees the new rule records the removal as an explicit, attributed "stopped synchronizing" change, in the same session as the `.meshignore` edit when that edit was made there. A device that receives the removal together with the rule leaves its local file on disk; it does not apply the removal as a deletion. History keeps the old versions, and a device that joins later doesn't receive the file.
- [x] **Shared or per-device policy.** Should the exclude live in the synchronized `.meshignore`, so every device follows it once it arrives? Or in a device-local list, e.g. `.mesh/config.json`, like Git's `info/exclude`? Or both? `workspace.json` is per-device on every device, but other cases may be one device only.
  - **Answer (Kai, 2026-11-06):** Shared `.meshignore` only. The rule is synchronized and every device follows it once it arrives. `workspace.json` is per-device on every device, so a shared rule fits it. Device-local excludes (an `info/exclude` equivalent) are out of scope and become a follow-up ticket if they are needed.
- [x] **Peers on the old policy.** Until `.meshignore` (or the new engine) reaches a peer, that peer keeps sending changes to the excluded path. Should this device drop them silently, keep them aside as recoverable inputs, or report them in `status`?
  - **Answer (Kai, 2026-11-06):** Ignore silently. On a device that follows the rule, a peer's incoming change to the excluded path doesn't touch the local file, doesn't pause sync, and isn't reported in `status`. On `main` the removal wins over the peer's modification. The peer's version stays in its device-branch history; no extra recovery copy is made.
  - *Consequence for pre-change engines (verified in `recover_apply`, 2026-11-06):* an engine without this change applies the received removal like any other change. It moves its local file into `.mesh/recovery` (not lost, but the layout is gone from that device). The README states this and says to update every device before adding such a rule.
- [x] **Built-in or written default.** Is the Obsidian exclude built into the engine when `.obsidian/` exists? Or does `init` (and later guided setup) write a visible `.meshignore` the user can edit? And how does an existing vault such as this wiki get it?
  - **Answer (Kai, 2026-11-06):** `init` writes a visible `.meshignore`. When `.obsidian/` exists in the folder at `init`, it creates `.meshignore`, or appends to an existing one, with rules for `.obsidian/workspace.json` and `.obsidian/workspace-mobile.json`. A rule already present is not written again. The file is an ordinary synchronized, editable rule file and becomes part of the first captured session. The engine never adds rules to an existing workspace on its own. Existing vaults, this wiki included, get the rules by hand, and the README shows the two lines. No exclude is built into the engine.
  - *Note (refinement, 2026-11-06):* `init` captures existing files as the first session (`src/engine.rs` `init`). A joining device usually starts from an empty folder, so it writes nothing. If two devices both write a `.meshignore` at `init`, the ordinary text merge or conflict path applies.
- [x] **How a user excludes a file.** Is editing `.meshignore` the whole interface, or is there a CLI command (`dispatch-mesh ignore <path>`) that also explains the effect and confirms which synchronized files it removes?
  - **Answer (Kai, 2026-11-06):** Editing `.meshignore` is the whole interface. There is no new CLI command and no separate `status` note. The attributed removal recorded on `main` (see the first question) is the only trace, and it shows in history like any other change.
- [x] **Re-including.** If a rule is removed again, does the device's local file simply become a new change? What happens when the devices' local copies differ by then?
  - **Answer (Kai, 2026-11-06):** The ordinary change path, with no special case. Once the rule is gone, a device's local copy is a new change and is published as an addition. A device that receives that addition while holding its own local copy keeps that copy; `recover_apply` treats it as a local edit and doesn't overwrite it. Identical copies merge cleanly; differing copies become an ordinary conflict under ADR-0003, with both inputs preserved. No file on disk is replaced.
- [x] **Unpublished work on the excluded path.** *(Added 2026-11-06 in refinement.)* When the rule takes effect, the path may still have a change in an open or unpublished draft session, or an unresolved conflict (ADR-0003). Is that pending change published with the session, dropped from it, or kept as a recoverable input? Does an open conflict on the path have to be resolved first? The spec says nothing about this, and `capture` today treats any change to `observed` as an edit.
  - **Answer (Kai, 2026-11-06):** *Drafts:* the removal is the latest change to the path in the current session, so its net effect replaces that session's pending edit to the file. Earlier unsettled sessions keep their edits and finalize normally, and the removal follows them. *Open conflict:* recording the removal settles it, as a traceable resolution (ADR-0003) that selects "removed". Both original inputs stay preserved and recoverable, the local file stays on disk, and capture does not pause.
- [x] **Version target.** Is this a 0.2.x patch, because it affects every Obsidian vault today, or does it belong to 0.3 next to the tray?
  - **Answer (Kai, 2026-11-06):** v0.3.0. It ships with the tray milestone and its two-device tests ([[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]]). Older engines need to be updated first (see the third question), and a minor release suits that better than a patch.

## Acceptance criteria

Each criterion names the setup it needs. "Synchronized" means the path is on `main` and in the device's observed state.

1. **Exclude after sync, one device.** On a workspace where `.obsidian/workspace.json` is synchronized, adding `.obsidian/workspace.json` to `.meshignore` and running capture/sync succeeds without `needs_attention`. The file is unchanged on disk. Once the session finalizes, `main` no longer contains the path, and the commit that removes it is attributed to this device's author and also contains the `.meshignore` change.
2. **Later edits are not synchronized.** After criterion 1, editing the excluded file produces no captured change, opens no session and creates no draft commit.
3. **Other files keep syncing.** In the same workspace, `.obsidian/app.json` and `.obsidian/plugins/dispatch/data.json` are still synchronized after criterion 1, and edits to them are captured as usual.
4. **Second device receives the rule.** With devices A and B both synchronizing the file, B receives A's `.meshignore` change and removal. B's local `workspace.json` stays byte-for-byte unchanged, B's status is not `needs_attention`, and later edits to it on B are not synchronized.
5. **Peer without the rule.** If device B edits the path before receiving the rule, device A, which has the rule, receives that change without pausing, without changing A's local file and without a `status` message. Once both devices have synchronized, `main` does not contain the path.
6. **Pending draft.** If the current session already has an unpublished edit to the path when the rule takes effect, the finalized session contains the removal and not the edit.
7. **Open conflict.** If an unresolved conflict exists on the path when the rule takes effect, recording the removal settles it with a resolution that selects "removed". Both original conflict inputs remain recoverable, and the local file stays on disk.
8. **Re-include, identical copies.** Removing the rule on A, with identical local copies on A and B, brings the path back to `main` with no conflict.
9. **Re-include, differing copies.** With differing local copies, removing the rule produces an ordinary ADR-0003 conflict with both versions preserved. Neither device's file is overwritten on disk.
10. **Obsidian default on init.** `init` in a folder that contains `.obsidian/` and no `.meshignore` creates `.meshignore` with exactly the rules `.obsidian/workspace.json` and `.obsidian/workspace-mobile.json`. With an existing `.meshignore` it appends only the rules that are missing and keeps the existing content. Without `.obsidian/` it writes no `.meshignore`. In every case, the rest of `.obsidian/` is synchronized.
11. **No silent edits to existing workspaces.** Opening or running an existing workspace without these rules never modifies `.meshignore`.
12. **Unchanged guards.** A path that becomes unsupported for another reason (symlink, case collision, size limit) still stops capture with its existing message. The fixed secret excludes are unchanged.
13. **README.** *Preview boundaries* and *Recovery procedures* describe this behavior. They no longer say that changing ignore rules for synchronized files is unsupported, show the two Obsidian lines for existing vaults, and warn that engines before 0.3.0 must be updated before such a rule is added.
14. **Gates.** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` pass, and the new tests cover criteria 1–12.

## Implementation plan

Written 2026-11-06 against `main` at `29d3d58`, after reading `src/engine.rs` (`scan`, `capture`, `render`, `merge_change`, `publish_ready`, `install`, `ignored`, `recover_apply`, `summary_input`, `resolve`), `src/model.rs`, `src/ai.rs` and `tests/engine.rs`. No inline comments or code drift since refinement.

### Core rule

**A tree Mesh computes never contains a path that the tree's own ignore policy excludes.** This holds for the `main` tree written by `publish_ready`, the draft view built by `render`, and the view `install` / `recover_apply` writes to disk. On disk, "not in the tree" means *leave the file alone*, never *delete it*. Every acceptance criterion follows from that rule plus two local steps: capture records the removal, and an open conflict is settled.

The policy is computed **from a tree's own `.gitignore` / `.meshignore` contents plus the fixed excludes**, not from the disk. Three reasons:
- On a receiving device the new `.meshignore` is in the incoming tree before it is on disk. A disk-based check would delete the file in the same apply that brings the rule (AC 4).
- A crash-recovered apply journal (ADR-0007) must reach the same decision as the original apply.
- Two devices computing the same `main` tree must agree on it.

### Steps

1. **`Policy` from a tree** (`src/engine.rs`, new private type or small module `src/policy.rs`).
   - `Policy::from_files(root, &Files)` builds one `ignore::gitignore::Gitignore` per directory, from the `.gitignore` / `.meshignore` entries in the map, using `GitignoreBuilder::add_line`. `Policy::ignores(path)` applies the fixed excludes first (`.mesh/`, `.git/`, `.env`, `.env.*`, `*.pem`, `*.key`), then the nested rules in parent→child order, exactly as `ignored()` walks them today.
   - `ignored()` becomes `Policy::from_files(root, &self.state.observed)`. That is equivalent to the disk, because `install` captures pending disk edits before applying.
   - Unit tests compare `Policy` against the walker in `scan` on a fixture with nested rules, negations and directory rules, so the two never disagree.
   - *Why first:* every later step depends on it, and a mismatch between the walker and `Policy` is the main correctness risk.
2. **`scan`** no longer refuses a newly ignored path. The guard at `engine.rs:343` stays for an observed path that is still on disk but that `Policy::from_files(root, &result)` does **not** ignore: something else skipped it, so refusing stays the safe default. Its message drops "ignored or". Symlinks, case collisions, size limits and reparse points keep their errors (AC 12).
3. **`capture`**: a path in `observed` that is missing from `files` and that the new policy ignores is an *exclusion*, not a deletion.
   - If the session has no change for it yet, record `Change { base: observed, content: None }`, as for a deletion.
   - If the session already has a change, set `content = None` directly instead of calling `git.merge(existing, observed, None)`. Today that merge can fail and park the bytes in `pending_capture` (AC 6, Q8 *drafts*).
   - If the change's base is `None`, it disappears, as today.
   - Then **settle open conflicts on the path** (Q8): for each `unresolved()` conflict on it, insert a `Resolution { selected: None, author: self.config.identity, created_at: now }`. Resolutions already synchronize through the manifest, and concurrent ones keep the existing deterministic tie-break in `fetch`. Also run this settle step from `render` for unresolved conflicts on paths the rendered policy ignores. A device that only *receives* the rule then also clears a stale local conflict, and `publish_ready` stops blocking on it (AC 7).
4. **`render` and `publish_ready`**: while applying a session's changes, apply `.gitignore` / `.meshignore` changes first, rebuild the policy from the current tree, then apply the rest.
   - For a path the policy ignores, `set_file(None)` **without calling `merge_change`**. That creates no modify/delete conflict when a peer's late edit meets the removal; the removal wins on `main` (AC 5, Q3), whichever order the sessions publish in.
   - At the end, drop every path the final tree's policy ignores (`files.retain`), so the invariant holds even for paths no session touched.
   - In `publish_ready` the drop counts toward `effective_changes`. A peer session whose only change was to the excluded path becomes empty and goes through `discard_empty_projection`, as for any other no-op session.
   - Do the same in `summary_input`, so the prepared session matches what will be published.
5. **`install` / `recover_apply`**:
   - Remove the `ensure!(!self.ignored(path))` refusal. The desired tree from `render` already contains no ignored path.
   - In `recover_apply`, build `Policy::from_files(root, &desired)` from the journal. Skip every path in `desired ∪ observed` that the policy ignores: no write, no backup, no removal. Keep it out of `capture_baseline` and `actual_after`, so `observed` never holds an ignored path. This is what leaves B's `workspace.json` byte-for-byte untouched (AC 4).
   - Re-including needs no special code (AC 8, 9). Without the rule, `before = None`, so an identical local copy is skipped (`current == after`), and a differing one goes through the existing "changed by user" branch, which captures it as an addition and produces the ordinary add/add conflict.
6. **Wording of the removal** (`src/model.rs`, `src/ai.rs`):
   - Add `#[serde(skip)] pub excluded: BTreeSet<String>` to `Session`. `summary_input` fills it with the removed paths that the prepared tree's policy ignores.
   - `ai::operation` reports `"excluded"` for those paths. The fallback message then says *"Stop synchronizing <path>"* / *"- Stopped synchronizing (now ignored): <path>"* instead of *Delete*. The provider prompt gets one sentence: *excluded means the file is kept on every device but no longer synchronized*.
   - Update the construction sites in `engine.rs`, `main.rs` and `codec.rs` (`map_blobs`).
   - *Why `serde(skip)` and not a persisted flag on `Change`:* `Session::snapshot_hash` serializes `changes`, and `reconcile_published` matches it against the `Mesh-Snapshot` trailer. A new serialized field would change the hash between engine versions. A pre-0.3 peer would then never recognize the session as published and would keep rendering it as a draft.
7. **`init`** (`Workspace::init`, before the first `capture_at`): if `root/.obsidian` is a directory, create `.meshignore` or append to it, with the lines `.obsidian/workspace.json` and `.obsidian/workspace-mobile.json`. Write only lines that are missing as exact trimmed matches, and add a newline if the existing file lacks a trailing one. The first captured session then contains the rule (AC 10). `open` and `run` never write the file (AC 11).
8. **README**:
   - *Preview boundaries*: changing ignore rules now stops synchronization of already synchronized files without deleting them anywhere. Rules live in the synchronized `.meshignore`.
   - Give the two Obsidian lines for existing vaults, and say that `init` adds them for new ones.
   - Warn: *update every device to 0.3.0 before adding such a rule; older engines move the received file into `.mesh/recovery`.*
   - *Recovery procedures*: remove the newly ignored cause from the `needs_attention` entry.
   - (AC 13)
9. **Gates**: `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` (AC 14).

### Tests

The engine tests in `tests/engine.rs` use the existing `init` / `finish` / `remote` helpers. Two-device tests follow the 13 existing `remote(...)` tests: two workspaces on one bare remote.

| Test | Asserts |
|---|---|
| replaces `changing_ignore_rules_does_not_publish_a_deletion_of_existing_content` | AC 1: capture succeeds, file unchanged on disk, finalized `main` lacks the path, the commit contains `.meshignore`, author is Alice, fallback message says "Stop synchronizing" |
| excluded file edits are not captured | AC 2: no session and no draft revision after editing the file |
| rest of `.obsidian/` keeps syncing | AC 3 |
| second device receives rule | AC 4: B's bytes unchanged, `attention` is `None`, later B edits not captured |
| peer edit before the rule arrives | AC 5: A doesn't pause, A's file is unchanged, `main` lacks the path once both have synced, no conflict record; both publication orders |
| pending draft edit | AC 6 |
| open conflict settled | AC 7: resolution selects `None`, the conflict record keeps both inputs, file on disk |
| re-include identical / differing | AC 8 / AC 9 |
| `init` Obsidian default | AC 10: fresh, existing `.meshignore` with one rule present, no `.obsidian/` |
| open/run don't write `.meshignore` | AC 11 |
| guards unchanged | AC 12: symlink still errors after this change |
| crash during apply | `recover_apply` from a journal containing the rule leaves the file alone (ADR-0007) |
| `Policy` vs walker | unit test, step 1 |

CLI (`tests/cli.rs`): one end-to-end `init` in a folder with `.obsidian/`, then `status`, which shows no attention and lists the `.meshignore`. Newly assertable: that no tree on `main` contains a path its own policy ignores. Add this as a helper assertion in the two-device tests.

### Wiki pages this makes wrong

- `README.md` (step 8), by the developer in this ticket.
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] is frozen and documents today's behavior. It stays as built, and the link from this ticket's *References* is enough.
- The requirements doc, the milestone plan and the ADRs contain no statement about ignore rules that this contradicts (checked 2026-11-06).

### Risks, order and rollback

- **Walker vs `Policy` disagreement** (high impact): a path the walker skips but `Policy` doesn't ignore is caught by the guard in step 2 and refused, not deleted. The step 1 tests pin both. *Order:* step 1 before steps 2–5, because each of them calls it.
- **`recover_apply` before `scan`/`capture`** (data safety): if the capture side lands first without step 5, a receiving device would delete the file on apply. Land steps 2–5 in one commit, or step 5 first. *Reason:* the apply path is the only one that writes or removes files on disk.
- **Pre-0.3 peers**: they move the received file into `.mesh/recovery` (Q3 consequence). This is mitigated only by the README warning, which is why it ships in a minor version.
- **Automatic resolutions**: they are authored with the local identity. A user reading the conflict history sees a resolution they never clicked. This is accepted in Q8; the fallback wording in step 6 explains the removal.
- **Rollback**: no storage or transport format changes (`serde(skip)` only), so reverting the commits restores the old behavior. Workspaces that excluded files meanwhile would then hit the old refusal again until the rule is removed.

### Size

Roughly 3–4 focused days: `Policy` and its tests 0.5; scan/capture/settle 0.5; render/publish 1; apply 0.5; wording and init 0.5; two-device tests 1; README and gates 0.25. Order-of-magnitude estimate, not a commitment. The two-device tests are the most likely to stretch it.

### ADRs

Plan signed off by Kai on 2026-11-06 by invoking `/develop US00003`. Extracted at sign-off as `proposed`: [[ADR-0013 — Derive exclusion from each tree's own ignore policy]]. Applying ADR-0003 and ADR-0007 needs no new ADR.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

<!-- GUIDE: Written by /test-plan — `- [ ]` manual checks only, excluding what the
     automated suites already assert. Keep `open_tests:` in sync as items are ticked. -->

## As-built notes

### 2026-11-06 — development (`/develop`, branch `us00003-exclude-device-local-files`)

Built in commits `25ca6c5` (engine and tests) and `2ae22f6` (README, AGENTS.md). Gates green: `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test` (110 tests over both packages, 39 engine tests of which 14 are new). Demonstrated with the CLI on two workspaces sharing a local bare remote. Alice added the two Obsidian rules: Bob's `workspace.json` stayed on disk and his later edit stayed local; `main` dropped the file in Alice's commit together with `.meshignore`, whose message lists "Stopped synchronizing (now ignored): .obsidian/workspace.json"; both devices showed `status: synced`, no attention.

Deviations from the plan:

- **`.ignore` is part of the policy.** The folder walker already honored `.ignore` files (the `ignore` crate's default), while the old `ignored()` check did not. `Policy` follows the walker's precedence (`.meshignore` > `.ignore` > `.gitignore`, deepest directory wins within a kind, an ignored directory can't be re-included). The README now names all three files.
- **Rules from folders above the workspace** are still honored by the walker but are not part of a tree's policy, because they don't synchronize. A synchronized file that such a rule starts to skip falls under the remaining `scan` guard ("now unsupported … no deletion was recorded"), with a test. The README says these rules are not part of the policy.
- **Open conflicts are settled only in `render`**, not in `capture` as well. `sync` renders on the recording device and on every receiving device, so one place covers both.
- **`ignored()` was removed** instead of being re-implemented on `Policy`. Its only caller was the `install` refusal that this ticket removes.
- **No text of an excluded file is sent to an AI provider.** Its operation is reported as `excluded` with `content_available: false`, since its old text says nothing about the session.
- **Mutation check of the new tests.** Removing the apply-side filter fails the second-device and crash-recovery tests. Removing the render/publish skip at first did *not* fail the peer-edit tests: the conflict it creates was settled at once. Those tests now assert that no conflict record exists for the path, and they fail without the skip.
- **`AGENTS.md`** gained a behavioral-invariant bullet pointing to ADR-0013, because the agent guide must stay consistent with the ADRs.

### 2026-11-06 — code review blocks the hand-off

Review of `2ae22f6` found F1 (initialization writes through an unsupported symlink before rejecting it; AC 12 / safety guard) and F2 (an ignored rule file makes the AC 1 exclusion fail; shared-policy contract needs a decision). Keep `In progress`; do not run `/test-plan`. The remediation steps are recorded below. No implementation was changed in this review.

## Code review

### 2026-11-06 — branch us00003-exclude-device-local-files at 2ae22f6

**Verdict: blocked.** Two blocking findings and one non-blocking test gap. This ticket cannot proceed to `/test-plan` or leave Review as built.

**Scope and gates.** Independent review in a session that authored none of the implementation. Reviewed the complete ticket, linked ADR-0002/0003/0007/0013, both commits in `origin/main` (`29d3d58`) to `HEAD`, all changed source and tests, and surrounding capture/render/publication/recovery code. No open PR was found for this branch. `cargo fmt --check` and `cargo clippy --all-targets -- -D warnings` pass. Full `cargo test` is running; its final result will be recorded before this review is delivered. Additional CLI reproductions used disposable local folders, no live remote or AI provider. Evidence: `target/us00003-review-evidence.json`; synthetic workspaces retained at `C:/Users/kai/AppData/Local/Temp/mesh-us00003-review-b0go7qud`.

1. **F1 — [P1, blocking] Reject an unsupported `.meshignore` link before writing its target.** `src/engine.rs:1441` reads through the link and `src/engine.rs:1460` writes through it. Reproduction: create `.obsidian/`, an external text file containing `original\n`, and a `.meshignore` symlink to that external file; invoke `init`. The command fails with “Symbolic links are unsupported”, but the external file already contains both appended Obsidian rules. The scan guard runs after `write_obsidian_excludes`, so it cannot protect the write. This breaks the unchanged unsupported-path safety guard (AC 12) and modifies user data outside the workspace on failed initialization. **Fix:** validate `.meshignore` with symlink/reparse-aware metadata before any read or mutation, reject unsupported targets, and install updates without following a substituted link. Add an initialization regression asserting failure leaves the external target byte-for-byte unchanged; cover Windows reparse points as applicable. The current size-limit/ancestor-rule test does not cover the symlink case promised in the plan.

2. **F2 — [P2, blocking contract mismatch] An excluded rule file splits walker policy from tree policy.** `src/engine.rs:348–352` builds policy only from files the walker returned, while the walker also applies ignored rule files left on disk. Reproduction matching AC 1: synchronize `.obsidian/workspace.json` and a `.gitignore` containing `.meshignore`; then create `.meshignore` containing `.obsidian/workspace.json` and invoke `sync`. It fails with “A previously synchronized path is now unsupported: .obsidian/workspace.json”, instead of successfully recording the exclusion. A `.gitignore` that ignores itself and `note.md` reproduces the same mismatch. File bytes are preserved, but the promised workflow pauses. The policy/walker fixture does not test rule files excluded by other rules or by themselves. **Decision required before fixing:** the ticket promises exclusion without this exception, while the shared-policy design requires the rule to synchronize. Either make policy files participate in the shared policy consistently even when ordinary rules match them, or explicitly declare this configuration unsupported and define the actionable diagnosis/recovery. Record that decision through `/update-ticket` (and ADR-0013 if its rule changes), then align scan, tree policy and tests. Merely reading the hidden rules from disk into the tree matcher would introduce device-local policy and would not satisfy ADR-0013.

3. **F3 — [P3, non-blocking] Pin the excluded-file provider payload.** `src/ai.rs:364–375` adds the useful guarantee that excluded text is omitted even with `send_text`, but `tests/provider.rs` has no exclusion-specific assertion. Add a mock-provider case with excluded text and a normal text edit: assert `operation: excluded`, `content_available: false`, absent before/after text for the excluded path, and normal text sharing for the other path. No live provider is needed. This is regression coverage for the recorded privacy-related deviation, not a demonstrated disclosure in this build.

**Acceptance criteria and plan.** The new tests substantiate the ordinary AC 1–11 flows, both late-edit publication orders, identical/differing re-inclusion, and journal replay. AC 13 is reflected in the README. AC 1 has the uncovered F2 configuration; AC 12 fails F1, and AC 14's coverage claim misses that symlink case even if the mechanical suite passes. The deviations concerning `.ignore`, ancestor rules, render-only conflict settlement, removal of `ignored()`, and omission of excluded text from provider input are dated in the as-built notes. No unrelated commits were found.

**What should stay.** Deriving apply policy from the journal protects the receiving device and crash replay from disk-policy timing differences. Keeping `excluded` out of serialized changes preserves existing snapshot hashes. The two-publication-order tests assert absence of conflict records, which is stronger than checking only the final tree; the recorded mutation checks explain why that assertion matters. Original conflict inputs remain retained when exclusion records an automatic resolution.

**Before this leaves review — remediation plan for a separate session:**

- Resolve F2's policy-file contract decision with the requester and record it through `/update-ticket` before changing that behavior.
- In a separate `/develop US00003` session, fix F1 and implement the agreed F2 behavior, with the reproductions above as regressions. Add F3's mock-provider assertion if practical.
- Run the mechanical gates, document fixes and actual results, and clear `open_findings` to empty for the re-check of these blocking findings. Re-check is part of this same review; `/test-plan` waits for `open_findings: 0`.

## Follow-ups

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[Dispatch Mesh milestone plan]]: 0.2.x ignore-policy documentation item
- [[ADR-0002 — Group files into sessions with an inactivity boundary|ADR-0002]], [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions|ADR-0003]]
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]]: documented the current ignore behavior and its recovery entry
- [[ADR-0013 — Derive exclusion from each tree's own ignore policy|ADR-0013]]: the exclusion rule this ticket introduces
- Code: `src/engine.rs` (`scan`, `install`, `ignored`), `README.md` (*Preview boundaries*, *Recovery procedures*)

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
