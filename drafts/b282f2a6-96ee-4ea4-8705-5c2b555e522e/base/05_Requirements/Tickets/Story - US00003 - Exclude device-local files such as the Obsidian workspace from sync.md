---
id: US00003
type: story
status: "Backlog"
priority: medium
rank: 2000
version_target:
size:
assignee:
owner: "[[Kai]]"
open_questions: 7
open_tests:
open_findings:
discussion:
updated: 2026-11-05
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

A first cut, to be sharpened in refinement:

- **Stop synchronizing a path that was already synchronized.** A file newly matched by an ignore rule leaves the synchronized set without being deleted from any device's disk. The engine records this as an explicit, attributable change, not as an inferred deletion. Capture and sync continue instead of pausing.
- **The incoming side:** a peer's change to a path this device now excludes neither pauses sync nor overwrites the local file. The exact behavior is decided in refinement.
- **Obsidian default:** excludes for `.obsidian/workspace.json` and `.obsidian/workspace-mobile.json` when the folder is an Obsidian vault, without excluding the rest of `.obsidian/`.
- **README:** replace the "not supported yet" boundary and the `needs_attention` recovery entry with the new behavior.
- **Tests:** engine and CLI tests for excluding an already-synchronized file on one device and on two devices. Also: a peer still on the old policy, re-including a file, and the Obsidian default.

## Out of scope

- A tray or settings UI for editing excludes. The tray shows status only until later 0.x milestones.
- Rewriting history to remove a file's past versions from the remote.
- Selective sync per device for whole folders, e.g. "don't download `01_Sources/` on the phone".
- Changing the fixed secret excludes (`.env`, keys).

## Open questions

- [ ] **What "stop synchronizing" means for the shared copy.** When a device excludes a synchronized file, does `main` keep the file's last version, frozen and untracked, or does `main` record its removal while every device keeps its local copy? The second keeps the repository clean but is a deletion on GitHub.
- [ ] **Shared or per-device policy.** Should the exclude live in the synchronized `.meshignore`, so every device follows it once it arrives? Or in a device-local list, e.g. `.mesh/config.json`, like Git's `info/exclude`? Or both? `workspace.json` is per-device on every device, but other cases may be one device only.
- [ ] **Peers on the old policy.** Until `.meshignore` (or the new engine) reaches a peer, that peer keeps sending changes to the excluded path. Should this device drop them silently, keep them aside as recoverable inputs, or report them in `status`?
- [ ] **Built-in or written default.** Is the Obsidian exclude built into the engine when `.obsidian/` exists? Or does `init` (and later guided setup) write a visible `.meshignore` the user can edit? And how does an existing vault such as this wiki get it?
- [ ] **How a user excludes a file.** Is editing `.meshignore` the whole interface, or is there a CLI command (`dispatch-mesh ignore <path>`) that also explains the effect and confirms which synchronized files it removes?
- [ ] **Re-including.** If a rule is removed again, does the device's local file simply become a new change? What happens when the devices' local copies differ by then?
- [ ] **Version target.** Is this a 0.2.x patch, because it affects every Obsidian vault today, or does it belong to 0.3 next to the tray?

## Acceptance criteria

<!-- GUIDE: Testable statements, each checkable by someone without this ticket's history. -->

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

## References

- [[Dispatch Mesh milestone plan]]: 0.2.x ignore-policy documentation item
- [[ADR-0002 — Group files into sessions with an inactivity boundary|ADR-0002]], [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions|ADR-0003]]
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]]: documented the current ignore behavior and its recovery entry
- Code: `src/engine.rs` (`scan`, `install`, `ignored`), `README.md` (*Preview boundaries*, *Recovery procedures*)

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
