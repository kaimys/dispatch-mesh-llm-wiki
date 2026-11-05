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

## [2026-11-05] update | ADR-0001–0010 folded into tickets
- [[Story - US00001 - Tray icon and status window host]]: control-interface question answered via [[ADR-0009 — Embed the engine with a runner handle and published status]] (open_questions 7 → 6); icon states, attention reason and Quit narrowed by ADR-0009/[[ADR-0010 — Report workspace status and session stages separately]]; context updated.
- [[Story - US00002 - Local engine control interface for the tray host]]: criterion 1 now names ADR-0009 and requires setting it `accepted` at Done.
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: plan step 4 requires lock-free status per [[ADR-0007 — Persist state transactionally with recovery journals]]; ADR section lists governing ADRs; Git-classification ADR still a candidate.
- [[Task - TASK00002 - Choose the Git backend before bundling Git]]: constraints from ADR-0003/0006/0008 added; merge-equivalence question sharpened.
- ADR-0010 amended: offline definition moved into Decision; rejected push, unknown Git failure and AI-summary failure → `needs_attention`; lock-free status read (Kai's approval of the TASK00001 review points).

## [2026-11-05] plan | US00002 implementation plan drafted
- [[Story - US00002 - Local engine control interface for the tray host]]: `## Implementation plan` written: 8 ordered steps, starting once TASK00001 steps 2–4 land. It also covers the `acquire`/`run`/`Runner::start` split, bounded busy-lock retry on shutdown, a scan-free per-tick snapshot, and tests for criteria 3–7. It asks [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]] step 4 for a scan-free `status_from` core. No new ADRs; [[ADR-0009 — Embed the engine with a runner handle and published status]] is to be amended at Done. Awaiting Kai's sign-off.

## [2026-11-05] develop | TASK00001 code-complete
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: implemented on branch `task00001-stabilization-gaps`, commit `08c0b02`; gates green locally (63 tests, 19 new). As-built notes recorded; assignee set. Awaiting `/code-review TASK00001` in a fresh session.
- [[Dispatch Mesh requirements and architecture]]: desktop-state sentence annotated with the six-state model (ADR-0010).

## [2026-11-05] code-review | TASK00001 changes required at 08c0b02
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: independent review recorded against `main...08c0b02`; all mechanical gates passed locally on Windows (63 tests). Three blocking findings: sync-failure persistence can be skipped under lock contention (reproduced), HTTP-date retry delays are omitted, and the discarded status stage lacks an automated assertion. One non-blocking gap requests actual AI-failure persistence/recovery coverage through CLI summarization.
- Set `open_findings: 3`; kept `In progress`, unfrozen, and left `open_tests` unchanged. Added a dated as-built blocker entry and repair plan for a fresh `/develop` session; `/test-plan` remains blocked. No implementation code changed and no PR comment was posted. Disposable probe evidence remains in ignored `target/review-task00001/`.

## [2026-11-05] develop | TASK00001 review fixes
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: B1–B3 and N1 from the review of `08c0b02` fixed in `7ac5316`; gates green (68 tests); `open_findings` cleared to empty. Awaiting a fresh `/code-review TASK00001`.

## [2026-11-05] code-review | TASK00001 re-review at 7ac5316
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: re-reviewed the repair commit; independently passed formatting, Clippy and all 68 tests on Windows. Closed B1, B3 and N1; the original contention probe observed five failures with none missing from durable status.
- B2 remains blocking for the rejected RFC 850/asctime HTTP-date variants of `Retry-After`; recorded RFC 9110 recipient requirements and concrete probe evidence. Added non-blocking N2 for impossible dates being normalized instead of rejected. Set `open_findings: 1`; kept `In progress`, unfrozen, and left `open_tests` unchanged. Added a dated as-built hand-off blocker and the next repair steps. No implementation changes or PR comment.

## [2026-11-05] develop | TASK00001 re-review fixes
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: B2 (all HTTP-date forms) and N2 (impossible dates rejected) from the re-review of `7ac5316` fixed in `291c263`; gates green (69 tests); `open_findings` cleared to empty. Awaiting a fresh `/code-review TASK00001`.

## [2026-11-05] research | Mesh competitor profiles and SWOT
- Created [[Competitor overview]] and profiles for [[gitwatch]], [[SparkleShare]], [[Obsidian Git]], [[Agentic Git Sync]], [[git-annex assistant]], [[Syncthing]], and [[Jujutsu]] (architectural reference), from [[Dispatch Mesh concept]]. Each includes description, repository/homepage, dated GitHub metrics, documented features, Mesh differences, adaptation recommendations, and primary-source links.
- Created [[Dispatch Mesh SWOT analysis]] from the profiles, distinguishing implemented preview behavior from roadmap features and analyst hypotheses from observed facts.
- Queried live REST stars/forks and GraphQL discussion-thread totals/enabled state. Marked git-annex canonical GitHub metrics N/A and separated unofficial con/git-annex counts; noted SparkleShare's funded reboot and inaccessible homepage.
- Listed all nine pages in [[index]]. Registered manual refresh before decision reuse in [[Competitor overview]]; no recurring automation created. This was document research, not competitor runtime or Mesh engine verification.

## [2026-11-05] code-review | TASK00001 re-review at 291c263
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: independently passed formatting, Clippy and all 69 tests on Windows. Closed the earlier B2 format-rejection and N2 invalid-date cases; 509 independent calendar comparisons and six original-format reproductions passed.
- Recorded new blocking B4: RFC 850's 50-year rule compares only years, so a timestamp just beyond the actual cutoff incorrectly reports a roughly 50-year retry delay rather than an expired date. Retained probe evidence and a repair plan; set `open_findings: 1`, kept `In progress` and unfrozen, and left `open_tests` unchanged. Restored the missing review-section heading without altering historical entries. No implementation changes or PR comment.

## [2026-11-05] develop | TASK00001 B4 fix
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: B4 (RFC 850 50-year cutoff on the complete timestamp) from the re-review of `291c263` fixed in `c079e9d`; gates green (69 tests); `open_findings` cleared to empty. Awaiting a fresh `/code-review TASK00001`.

## [2026-11-05] code-review | TASK00001 clean at c079e9d
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: independently verified the B4 fix, with 360 cutoff cases, both original reproductions and 509 earlier calendar comparisons. Formatting, Clippy and all 69 tests passed locally on Windows. Confirmed the files reviewed before the author committed match `c079e9d`; no new findings.
- Recorded 0 blocking and 0 non-blocking findings; set `open_findings: 0`. Kept `In progress`, unfrozen, and left `open_tests` unchanged. `/test-plan TASK00001` is next against the reviewed commit. No implementation changes or PR comment; no live remote/provider or Linux/macOS checks.

## [2026-11-05] test-plan | TASK00001 → Review
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: manual test plan written for `c079e9d` (5 items: CI on Linux, real network loss, real refused login via GCM, locale with translated Git, README review); `open_tests: 5`; contract zone frozen 2026-11-05; status → Review.

## [2026-11-05] fix | TASK00001 Linux CI failure
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: CI run 42018003838 failed on ubuntu-latest (platform-dependent fake remote in the lock-contention test; Windows job cancelled by fail-fast). Test-only fix `af1d3a0` pushed; `open_findings` cleared pending a re-review of that diff.

## [2026-11-05] fix | TASK00001 Linux lock race and CI fail-fast
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: spurious `WorkspaceBusy` on Linux (flock held briefly by spawned children) fixed by a 500 ms lock retry in `07903c4`; CI matrix no longer fail-fast. Re-review needed for `af1d3a0` + `07903c4`.

## [2026-11-05] ux | Claude Design briefing
- [[Claude Design briefing — desktop app]]: new design brief in `06_UX/` for the 0.3 tray app (icon states, menu, status window, conflicts, presence, notifications), with a flow-level sketch of 0.5 guided setup. It is built from ADR-0010, ADR-0009, ADR-0003, the requirements, the milestone plan, US00001 and the README status/recovery tables, and it lists US00001's open UI questions for the design to inform. Index updated.

## [2026-11-05] test | TASK00001 manual checks complete
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: checks 2 (real network loss) and 3 (refused login via GCM) passed with Kai; check 4 (locale on a translating Git) automated as an Ubuntu CI step in `41a543b` and passed (run 42033691572). `open_tests: 0`. Re-review of `af1d3a0`..`41a543b` still pending before Done.

## [2026-11-05] decision | TASK00001 no re-review
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: Kai decided one code review is enough; post-review commits `af1d3a0`..`41a543b` not re-reviewed (recorded in As-built notes). Ready for Done.

## [2026-11-05] merge | TASK00001 into main
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]: fast-forwarded into `main` at `41a543b`; CI green on both platforms (run 42035665583). Card still in Review until moved to Done.

## [2026-11-05] develop | US00002 implemented
- [[Story - US00002 - Local engine control interface for the tray host]]: in-process runner built as planned on `us00002-engine-control-interface` (`f2b3e1c`); as-built notes record the deviations (tokio `sync` feature, status split done here, marker-only body check). Gates green. `README.md` and `AGENTS.md` updated in the code change. Status stays `In progress`; `open_findings` empty pending `/code-review US00002`. [[ADR-0009 — Embed the engine with a runner handle and published status]] amendment remains due at Done.

## [2026-11-05] code-review | US00002 changes required at f2b3e1c
- [[Story - US00002 - Local engine control interface for the tray host]]: independent review of `main...f2b3e1c` recorded with 2 blocking findings and 1 non-blocking finding. Full `cargo test` failed in the new summary-recovery test when CLI `session finish` hit runner lock contention; the runner-suite rerun reproduced it, while the isolated test passed. Formatting and clippy passed; the separate storage/transport suite passed 8/8. Checks were local Windows only, with no live remote/provider requests.
- Set `open_findings: 2`; kept `In progress`, unfrozen, and left `open_tests` unchanged. Added a dated as-built blocker entry and repair steps for a fresh `/develop US00002`: stabilize the failing test, add missing host-error/busy-filtering assertions and correct the overstated credential-test evidence. `/test-plan` remains blocked. No implementation code changed, no contract rewrite was made and no PR comment was posted.

## [2026-11-05] develop | US00002 review findings fixed
- [[Story - US00002 - Local engine control interface for the tray host]]: R1 (busy-retry in runner-test CLI calls), R2 (published capture/sync error and held-lock tests) and R3 (child-process credential sentinel test, dated correction of the as-built claim, remaining deviations recorded) addressed in `0e4c9a6`. Gates green on two consecutive full runs. `open_findings` cleared to empty pending a fresh `/code-review US00002`.

## [2026-11-05] code-review | US00002 fixes clear at 0e4c9a6
- [[Story - US00002 - Local engine control interface for the tray host]]: independent re-review resolves R1–R3 with 0 blocking and 0 non-blocking findings. Formatting, clippy and the full local Windows test suite passed (79 tests); a separate runner-suite run also passed 10/10. No live remote/provider requests were made.
- Set `open_findings: 0`, preserving status, `open_tests` and the unfrozen contract. `/test-plan US00002` is next against `0e4c9a6`; ADR-0009 amendment/acceptance remains due at completion. No implementation code changed and no PR comment was posted.

## [2026-11-05] test-plan | US00002 test plan written, frozen, moved to Review
- [[Story - US00002 - Local engine control interface for the tray host]]: manual plan for reviewed build `0e4c9a6` with 3 items (Ctrl+C in a real terminal, Ctrl+C during a real network outage, Linux CI on the pushed branch); `open_tests: 3`. Gates re-run green (79 tests). Contract zone frozen 2026-11-05; status `In progress` → `Review`. ADR-0009 amendment/acceptance remains due at Done.

## [2026-11-05] test | US00002 manual test plan complete
- [[Story - US00002 - Local engine control interface for the tray host]]: all 3 manual checks passed and recorded (Ctrl+C in a real terminal; Ctrl+C during a real network outage against `kaimys/mesh-manual-test`; CI run `42047824881` green on ubuntu-latest and windows-latest). `open_tests: 0`. Status stays `Review`; Done still requires the ADR-0009 amendment and acceptance (criterion 1).

## [2026-11-05] decision | ADR-0009 amended and accepted
- [[ADR-0009 — Embed the engine with a runner handle and published status]]: rewritten to the API as built under [[Story - US00002 - Local engine control interface for the tray host]] (acquire/run(lease) split, optional initial snapshot, stop-on-drop, runtime built on the caller thread, deferred named-pipe/Unix-socket extension, rejected single-entry-point alternative); status `proposed` → `accepted`. [[index]] row updated. US00002 branch fast-forwarded into `main`.
