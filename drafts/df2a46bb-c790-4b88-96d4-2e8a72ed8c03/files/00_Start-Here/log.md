---
type: log
owner: "[[Kai]]"
updated: 2026-11-05
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

## [2026-11-05] move | Verification reports into the wiki
- Moved `reports/github-workflow-test-2026-11-04.md` and `reports/github-workflow-findings-2026-11-04.md` from the source repository to `08_Delivery-and-QA/Reports/` as [[github-workflow-test-2026-11-04]] and [[github-workflow-findings-2026-11-04]]. The content is unchanged; only `owner` frontmatter was added.
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]] now links to the findings page instead of the old repository path.
- Listed in [[index]].

## [2026-11-05] ticket | TASK00001 locale-independent Git errors
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: added (Kai's decision) forcing `LC_ALL=C` on every Git invocation so offline/attention classification reads untranslated messages; new acceptance criterion 6a; linked to TASK00002.

## [2026-11-05] ticket | TASK00002 Choose the Git backend before bundling Git
- Created [[Task - TASK00002 - Choose the Git backend before bundling Git]] (Backlog, v0.4.0) from the question whether `src/git.rs` should use a Git library instead of the CLI; 6 open questions. Listed in [[index]].

## [2026-11-05] plan | TASK00001 implementation plan drafted
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: `## Implementation plan` written (9 ordered steps, review points, cross-ticket effects on US00002/US00001, ADR candidates). Awaiting Kai's sign-off; no ADRs created yet.

## [2026-11-05] decision | Record agreed Mesh architecture decisions
- Created ADR-0001–ADR-0008 as accepted records of the implemented console-preview decisions: separate draft transport and immutable history; multi-file sessions and the five-minute boundary; recoverable conflicts; scoped agent intent; configurable opt-in AI; managed Git ownership; transactional recovery; reference-based blob storage. Each page is listed and linked in [[index]] and owned by [[Kai]]. Original individual decision dates are unavailable and remain unset; the recording date and provenance are explicit.
- Created ADR-0009 and ADR-0010 as proposed (agreed, not yet built) from Kai's 2026-11-05 answers: the in-process runner interface and workspace/session status model. Added dated record-zone links to [[Story - US00002 - Local engine control interface for the tray host]], [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]] and [[Story - US00001 - Tray icon and status window host]]. Ticket states, counters and implementation-plan approval are unchanged.
- Reconciled outdated transport, intent-command and finish-command wording in [[Dispatch Mesh requirements and architecture]], added governing ADR links and page ownership, and clarified ADR status vocabulary in [[index]]. TASK00002's Git backend choice is explicitly left open; its ADR will be handled separately. Product-only decisions remain in the requirements and milestone plan.
