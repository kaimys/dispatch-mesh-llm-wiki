---
type: log
owner: "[[Kai]]"
updated: 2026-11-04
---
# Log

Append-only. One entry per change: `## [YYYY-MM-DD] <type> | <title>`, followed by the pages touched.

## [2026-11-04] setup | Dispatch board and wiki structure
- Turned `dispatch/wiki` into an Obsidian vault with the Dispatch board.
- Moved the concept docs: `README.md` → `04_Discovery/git-sync/README.md` (restores the `[[04_Discovery/git-sync/README]]` links); requirements, milestone plan and naming → `02_Product/`.
- Added [[index]], this log, [[Kai]] and the templates in `00_Start-Here/Templates/`.

## [2026-11-04] ticket | US00001 Tray icon and status window host
- Created [[Story - US00001 - Tray icon and status window host]] (Backlog, v0.3.0) from step 1 of [[Dispatch Mesh milestone plan#0.3 — tray application|0.3 — tray application]]; 6 open questions, including the toolkit and control-interface decisions the plan requires before 0.3.
- Listed in [[index]].

## [2026-11-04] refine | US00001 Tray icon and status window host
- Started refinement of [[Story - US00001 - Tray icon and status window host]]: Backlog → Refinement. Verified the context against the code (run loop and status derivation live in `src/main.rs`; three-value `status`; offline reads as needs attention; 0.2.x control interface not yet defined). Added a seventh open question (attention reason in the shell) and draft acceptance criteria.

## [2026-11-05] ticket | US00002 Local engine control interface for the tray host
- Created [[Story - US00002 - Local engine control interface for the tray host]] (Backlog, v0.2.5) from the 0.2.x item "Define the local engine/control interface"; 6 open questions.

## [2026-11-05] ticket | TASK00001 Close remaining 0.2.x stabilization gaps
- Created [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]] (Backlog, v0.2.5) bundling the partly done 0.2.x items: interrupted-publication and rename tests, six-state status model, 429 rate-limit vs. quota, storage-growth and recovery docs; 6 open questions.
- Linked both from [[Story - US00001 - Tray icon and status window host]] (References) and listed both in [[index]].

## [2026-11-05] refinement | US00002 Local engine control interface for the tray host
- [[Story - US00002 - Local engine control interface for the tray host]] → Refinement. 7 questions (1 added from code review of the run loop) answered by Kai: in-process engine, runner-published status snapshot, `Runner::start` → `RunnerHandle` on its own thread, AI-summary failures persisted, refuse-and-retry when the runner lock is held; cross-process channel deferred. Acceptance criteria written; open_questions 0.

## [2026-11-05] refinement | TASK00001 Close remaining 0.2.x stabilization gaps
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]] → Refinement. 6 questions answered by Kai: keep as one ticket, workspace summary + per-session stage with fixed precedence, offline = connection-level errors only, local-only uploaded-revision record, replace `status` values, README-only docs. Acceptance criteria written; open_questions 0.
