---
id: TASK00002
type: task
status: "Backlog"
priority: medium
rank: 1000
version_target: v0.4.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 6
open_tests:
open_findings:
discussion:
updated: 2026-11-05
completed:
frozen:
---
Decide, in an ADR, whether Mesh keeps driving the `git` executable or moves to a Git library (libgit2 via `git2`, or gitoxide via `gix`), and for which operations. Then introduce a backend interface in `src/git.rs` so the engine stops passing raw Git arguments, and move the local object, tree and ref operations onto the chosen backend. This closes the "Git distribution" decision the milestone plan requires before shipping 0.4, and may shrink what 0.4 has to bundle.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As the developer of Mesh, I want local Git operations to run in-process through a typed interface, so that the runner's per-tick work stays cheap as history grows, failures are classified by error type rather than by message text, and 0.4 packaging does not hinge on shipping a whole Git distribution for operations a library can do.

## Context

- Raised by Kai on 2026-11-05: "git.rs is using the git command line tool, wouldn't it be better to use a library instead?"
- Milestone source: [[Dispatch Mesh milestone plan]] → *0.4 — bundled Git and Windows packaging*:
  - "Package a pinned, tested Git distribution and the helpers needed by the supported authentication methods."
  - "Resolve the packaged executable explicitly."
  - Exit criterion: Mesh "starts and synchronizes on a clean Windows VM with no Git on `PATH`".
  - *Decisions to close during delivery*: "Git distribution, credential-helper packaging, installer format, and distribution review — needed before shipping 0.4".
  - 0.5 adds guided sign-in and secure credential storage.
- **Git CLI usage today** (`src/git.rs`, 303 lines; requires Git on `PATH`, tested with 2.35.1):
  - *Local plumbing on Mesh's private store:* `hash-object`, `cat-file`, `ls-tree`, `read-tree`/`write-tree`/`commit-tree` with a temporary `GIT_INDEX_FILE`, `update-ref`, `rev-parse`, `log`, `merge-base`, `check-ref-format`, `config`, `init`, and `merge-file` for three-way text merges.
  - *Network:* `fetch` and `push` over HTTPS with `GIT_TERMINAL_PROMPT=0` and `GCM_INTERACTIVE=never`. Credentials come from the user's Git Credential Manager, or an SSH agent.
- `src/engine.rs` calls `git.run`/`git.text` with raw argument lists at **12** call sites, besides the typed helpers in `git.rs`. A backend swap needs those behind an interface first.
- **Cost drivers:**
  - Every operation spawns a process, which is comparatively slow on Windows.
  - The runner syncs about every 3 s, and [[Story - US00002 - Local engine control interface for the tray host|US00002]] adds a status snapshot every tick.
  - `reconcile_published` runs `git log --format=%B` over all of `main` on every sync, so its cost grows with history.
  - Errors arrive as translated stderr text; [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] adds `LC_ALL=C` as the interim fix.
- **Why the CLI is still valuable:**
  - Through Git Credential Manager it provides GitHub browser sign-in, two-factor, token refresh and SSH agents with no Mesh code.
  - A library needs Mesh-supplied credential callbacks, which Mesh only owns once 0.5's credential storage exists.
  - The 44 automated tests and the live GitHub verification (`reports/github-workflow-findings-2026-11-04.md`) were run against real Git, including `merge-file` behavior that conflict handling depends on.
- **Candidates (to verify during refinement):**
  - *libgit2 / `git2`:* mature. Supports push, merge-file (`git_merge_file`) and custom credential callbacks. A C dependency, buildable vendored. Licensed GPLv2 with a linking exception.
  - *gitoxide / `gix`:* pure Rust, fast local plumbing, MIT/Apache licensed. Push support has historically lagged behind fetch; its current state must be checked.
- Recommendation from the 2026-11-05 discussion, not yet a decision:
  1. Put a backend interface in `git.rs`.
  2. Move local operations to a library before 0.4.
  3. Keep `fetch`/`push` on the bundled Git CLI with Git Credential Manager for now.
  4. Revisit network transport at 0.5, when Mesh owns credential storage.

## Scope

- An ADR in `07_Engineering/Decisions/` choosing the backend per operation group (local objects and trees, refs and history, three-way merge, network), with the rationale, licensing and distribution notes, and the consequence for 0.4's bundling scope.
- A backend interface in `src/git.rs` covering every operation the engine and transport use, so `src/engine.rs` no longer passes raw Git argument lists.
- Move the operation groups the ADR assigns to a library onto it; network operations stay on the CLI unless the ADR decides otherwise.
- The existing test suites pass unchanged against the new backend. Add tests for any behavior the old CLI path provided implicitly, such as ref-name validation, merge-file output and blob reachability after garbage collection.
- A measured comparison (before/after) of sync-cycle and status-snapshot cost on a workspace with a long `main` history.
- **Constraints from accepted ADRs** (added 2026-11-05). The backend ADR must preserve all three, and say how:
  - [[ADR-0008 — Store content as blobs referenced by metadata]]: native Git blobs stay reachable through the device snapshot tree after garbage collection; referenced objects are validated before use; legacy snapshots stay readable. ADR-0008 explicitly leaves the backend open.
  - [[ADR-0006 — Let Mesh own Git operations in managed folders]]: when repository state is unexpected, preserve captured work and raise attention. Never reset, rewrite or force-push history, whichever backend performs the operation. ADR-0006 explicitly does not choose the backend.
  - [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]: conflict records and selections are bound to exact input revisions. A backend whose clean-merge decisions or merged bytes differ from `git merge-file` changes which inputs become conflicts (see *Merge equivalence*).

## Out of scope

- Bundling, pinning and packaging Git itself, and the installer (the rest of 0.4).
- Credential storage and guided sign-in (0.5).
- Changing the on-disk repository or metadata formats. Objects, refs and trailers stay compatible with clients using the CLI backend.
- Fixing the full-history scan in `reconcile_published` beyond what the backend change gives for free (separate ticket if the measurement shows it is needed).

## Open questions

- [ ] **Library choice.** `git2` (libgit2) or `gix` (gitoxide) for the local operations? Check current `gix` support for writing trees and commits, merge-file and push, Windows build friction, and license and notice obligations for 0.4's distribution review.
- [ ] **Network transport.** Should `fetch`/`push` stay on the Git CLI with Git Credential Manager until 0.5, or move to the library now with Mesh-supplied credentials? If they stay, does 0.4 still bundle a Git distribution, only for network and credential helpers?
- [ ] **Merge equivalence.** Must library three-way merges produce byte-identical results to `git merge-file`, including conflict-marker format? Clients on different versions may be merging the same inputs during the transition. *Added 2026-11-05:* under [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]], a merge that is clean on one client but conflicting on another produces a conflict record on only one side. That raises the bar from "merged text looks right" to "the same inputs give the same clean/conflict outcome on every client version".
- [ ] **Transition.** Switch backends in one release, or keep the CLI backend selectable (config or feature flag) for a release as a fallback?
- [ ] **Performance target.** What cost is acceptable per sync cycle and per status snapshot, and on what history size? That target decides whether `reconcile_published` needs its own ticket.
- [ ] **Timing.** Should this land early in 0.4, before packaging work starts, or be pulled forward into 0.3, given that US00002's per-tick status snapshot multiplies the process-spawn cost?

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

- [[Dispatch Mesh milestone plan]] — 0.4 bundled Git and Windows packaging; 0.5 guided setup; Decisions to close during delivery
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] — interim `LC_ALL=C` fix for error classification
- [[Story - US00002 - Local engine control interface for the tray host|US00002]] — per-tick status snapshot
- Code: `src/git.rs`, `src/engine.rs` (`network_call`, `push_manifest`, `reconcile_published`), `src/transport.rs`
- `reports/github-workflow-findings-2026-11-04.md` — live verification against the Git CLI

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
