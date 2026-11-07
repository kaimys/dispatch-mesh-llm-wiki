---
type: log
owner: "[[Kai]]"
updated: 2026-11-06
---
# Log

Append-only. One entry per change: `## [YYYY-MM-DD] <type> | <title>`, followed by the pages touched.

## [2026-11-06] code-review | US00003 blocked at 2ae22f6
- Recorded the independent review in [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]]: two blocking findings (init writes through a `.meshignore` symlink before rejection; ignored rule files split walker/tree policy) and one non-blocking mock-provider coverage gap. Set `open_findings: 2`; kept `In progress` and left `open_tests` and the contract zone unchanged. Retained disposable CLI reproduction evidence under `target/us00003-review-evidence.json`. Independently reran all mechanical gates on Windows: formatting, Clippy and all 110 tests pass. A separate development session fixes the findings; the policy-file contract decision precedes F2's fix, and `/test-plan` remains blocked.

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

## [2026-11-05] release | v0.2.5
- Created [[v0.2.5]] (listed in [[index]]). Scope: [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]] and [[Story - US00002 - Local engine control interface for the tray host]], both promoted `Done` → `Released` (`completed` unchanged). Gates green (79 tests); CI run `42049287074` green on main. `Cargo.toml`/`Cargo.lock` bumped to 0.2.5, commit `85bfac3`, annotated tag `v0.2.5`, both pushed at Kai's request; packaged `dist/dispatch-mesh-0.2.5-windows-x64.zip`. No GitHub Release object created. Deferred: US00001 (v0.3.0), TASK00002 (v0.4.0) — unchanged targets.

## [2026-11-05] release | v0.2.5 GitHub release text
- Created [[v0.2.5 GitHub release]] in `08_Delivery-and-QA/Releases/GitHub/` (listed in [[index]] under [[v0.2.5]]): paste-ready GitHub release body with ticket, ADR and milestone-plan links to `kaimys/dispatch-mesh-llm-wiki`, breaking-change and upgrade notes, and package checksums. No `version:` frontmatter, so the Release Plan does not count it as a second release.

## [2026-11-05] refine | US00001 icon state set answered
- [[Story - US00001 - Tray icon and status window host]]: closed the *Icon state set* question from the Claude Design result ([[Dispatch Mesh desktop app.pdf]], deliverable 1). Four icons (Up to date, Working shared by the three in-progress states, Offline, Needs attention), the tooltip names the exact state, and Paused stays reserved until the pause ticket extends ADR-0010. Scope note, acceptance criteria and references updated; `open_questions` 6 → 5.

## [2026-11-05] refine | US00001 design answers recorded
- [[Story - US00001 - Tray icon and status window host]]: closed three more questions from the Claude Design result ([[Dispatch Mesh desktop app.pdf]]). Workspaces per host: several; the tray shows the highest-precedence state; already-running is per workspace. Attention reason: shown in the window header for needs attention and offline; its actions come with later tickets. Quit: never prompts (Variant A). Summary, scope, out of scope and acceptance criteria updated; `open_questions` 5 → 2 (UI toolkit and single or separate binary remain).

## [2026-11-05] refine | US00001 toolkit and binary decided
- [[Story - US00001 - Tray icon and status window host]]: closed the last two questions. UI toolkit: Tauri 2, with a Windows spike before the implementation plan, which records it as an ADR (closes the milestone's "Tray UI toolkit" item). Binary: separate `dispatch-mesh-tray.exe`; the CLI stays a console program. Context, scope and acceptance criteria updated; `open_questions` 2 → 0. Status stays `Refinement` until Kai ends refinement.

## [2026-11-05] refine | US00001 refinement ended
- [[Story - US00001 - Tray icon and status window host]]: refinement ended by Kai. Acceptance criteria marked final; added criteria for the tooltip format, Narrator labels and the tray menu with the Quit hint. `open_questions: 0`, `version_target: v0.3.0`. Status stays `Refinement`; next step is `/implementation-plan`, starting with the Tauri spike.

## [2026-11-05] plan | US00001 implementation plan
- [[Story - US00001 - Tray icon and status window host]]: folded in code drift (0.2.5 shipped `runner.rs` and `status.rs`), then wrote the implementation plan for review. Spike first as a gate, Cargo workspace with a `dispatch-mesh-tray` package, a Tauri-free presentation model and host core, tray, static status window, docs. No library change planned. ADR-0011 (Tauri 2) and ADR-0012 (separate executable) to be extracted after sign-off. Status stays `Refinement`.

## [2026-11-05] develop | US00001 started; ADR-0011 and ADR-0012
- [[Story - US00001 - Tray icon and status window host]]: development started (already `In progress`; assignee Kai), branch `us00001-tray-host`. Plan signed off; identifier `de.eightnine.mesh` chosen by Kai.
- New ADRs (proposed): [[ADR-0011 — Build the desktop host with Tauri 2]], [[ADR-0012 — Ship the desktop host as a separate executable serving several workspaces]]. Listed in [[index]]. [[Dispatch Mesh milestone plan]]: noted which 0.3 decisions are closed.

## [2026-11-05] develop | US00001 code complete
- [[Story - US00001 - Tray icon and status window host]]: tray host built on `us00001-tray-host` (`bf7a68f`, `091180e`); as-built notes and the agent's app run recorded; gates green on Windows, Linux CI pending a push. `open_findings` left empty. Next: `/code-review US00001` in a fresh session.
- [[Claude Design briefing — desktop app]]: dated note that the toolkit is decided (ADR-0011).

## [2026-11-05] ticket | US00003 Exclude device-local files such as the Obsidian workspace from sync
- Created [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync]] (Backlog, no version target) from Kai's request to exclude files such as `.obsidian/workspace.json`; 7 open questions. Evidence: 31 of 472 commits in this wiki's history touch that file, and an ignore rule for an already-synchronized path currently pauses capture.
- Listed in [[index]].

## [2026-11-05] code-review | US00001 shutdown race blocks hand-off
- Reviewed [[Story - US00001 - Tray icon and status window host]] on `us00001-tray-host` at `091180e`: 1 blocking finding (Quit does not await in-flight workspace startup/final capture) and 1 non-blocking finding (planned offline tooltip reason omitted). Recorded reproduction, AC assessment and fix steps in the ticket; `open_findings: 1`, status remains `In progress`, no freeze or test-plan changes.
- Reviewer reran fmt, Clippy and all 96 tests successfully on Windows. Linux CI and human taskbar/toast/Narrator checks remain outstanding. No implementation fixes made.

## [2026-11-05] develop | US00001 review fixes
- [[Story - US00001 - Tray icon and status window host]]: fixed both review findings (`794e6c0` Quit waits for starts in flight, with regression test; `da2d5ee` offline reason in tooltips). `open_findings` cleared to empty pending re-review. Next: `/code-review US00001` in a fresh session.

## [2026-11-05] code-review | US00001 review fixes verified
- Re-reviewed [[Story - US00001 - Tray icon and status window host]] at `da2d5ee`: both earlier findings resolved; 0 blocking and 0 non-blocking findings. Recorded the independent re-review and set `open_findings: 0`; status remains In progress, contract and test-plan counters unchanged.
- Independently reran fmt, Clippy and all 97 tests on Windows successfully. Next: `/test-plan US00001` against the reviewed commit; Linux CI and human taskbar/menu/toast/Narrator validation remain pending.

## [2026-11-05] test-plan | US00001 Tray icon and status window host
- [[Story - US00001 - Tray icon and status window host]]: manual test plan for `da2d5ee` (11 checks: notification area, light/dark/grayscale, scaling, click and first-close toast, menu and Quit hint, state transition, Quit with an open session, Narrator, Linux CI). Agent verified the CLI-side and window checks itself. `open_tests: 11`, contract zone frozen 2026-11-05, status In progress → Review.

## [2026-11-06] test | US00001 manual tests and Linux build fix
- [[Story - US00001 - Tray icon and status window host]]: Kai ran the manual plan for `da2d5ee`. Checks 1–10 passed (2 with a corrected tooltip expectation, "Not syncing"; 9 on a redo after the wrong tray was quit; 10 accepted by Kai without hearing the state-change announcement). Check 11 failed: Linux CI could not compile the tray (missing `tray/icons/icon.png`).
- Fixed in `296a09d` (PNG app icon from `icon.ico`, listed in `bundle.icon`). `open_tests` and `open_findings` cleared, status Review → In progress. Next: `/code-review US00001`, then `/test-plan US00001`.
## [2026-11-06] documentation | Device branch format
- Added [[Device branch format]] in `07_Engineering/`: corrected the branch name to `mesh/devices/<device-id>` and explained device identity versus user identity, the tree layout, all version-two manifest fields and values, session creation and indefinite retention, and the before/after roles of `base/` and `files/`.
- Checked against source revision `296a09d`, README, and existing storage/transport test assertions. Included deletion versus empty-file semantics, conflict and resolution records, legacy compatibility, and preview limits. Documentation-only; no runtime or remote tests run.
- Listed in [[index]] and registered Kai's refresh responsibility on format, lifecycle, or retention changes and before reuse with a newer release. No recurring automation configured.
- CI on `296a09d` (run 42183263365) green on `ubuntu-latest` and `windows-latest`.

## [2026-11-06] update | Inline competitor research refresh prompt
- Added a Dispatch inline prompt to [[Competitor overview]] to refresh all linked profiles, the overview, and [[Dispatch Mesh SWOT analysis]] using current primary sources and dated GitHub metrics. Uses the dispatch-mesh repository alias and the configured default agent. Preserves metric definitions, implemented-versus-planned distinctions, source attribution, and wiki logging. The refresh itself was not run.
- Correction (Kai): a fix found in testing does not reset the review or test counters. Restored `open_findings: 0`, set `open_tests: 0` (check 11 passed on `296a09d`), status back to Review.
## [2026-11-06] documentation | SQLite database format
- Added [[SQLite database format]] in `07_Engineering/`: exact three-table DDL, every column, a logical ER diagram distinguishing JSON references from SQL foreign keys, the format-two state wrapper, all local state properties, and content-reference locations.
- Explained local SHA-256 blobs versus remote Git blobs, transactional saves, WAL versus the filesystem apply journal, legacy backup/migration, indefinite retention, and data outside SQLite. Listed the page and its refresh responsibility in [[index]].
- Checked against source revision `29d3d58` and existing storage/transport test assertions. Documentation-only; no runtime or remote tests run.

## [2026-11-06] research | Competitor source-level storage approaches
- Added dated, revision-pinned storage architecture sections to [[gitwatch]], [[SparkleShare]], [[Obsidian Git]], [[Agentic Git Sync]], [[git-annex assistant]], [[Syncthing]], and [[Jujutsu]]: Git/peer representation, local stores, recovery/retention, and assessed pros and cons, with source-file links.
- Created [[Competitor storage approaches]] in `07_Engineering/` and listed it in [[index]]. Selected three proposals: inspectable local operation history, retention based on reachability/recovery guarantees, and explicit shared-policy/device-configuration boundaries. Recorded fit, costs, validation needs, and Mesh invariants; no implementation or ticket changes.
- Static review of pinned source snapshots only; no competitor builds or runtime benchmarks. Retained source evidence in ignored `target/competitor-source-review/`. Used the explicitly unofficial git-annex upstream mirror after the canonical browser failed; read relevant Git objects directly after long documentation paths prevented a complete Windows checkout. Existing activity metrics were not refreshed. Registered manual refresh before design adoption.

## [2026-11-06] documentation | Competitor storage tree layouts
- Added annotated tree layouts to [[gitwatch]], [[SparkleShare]], [[Obsidian Git]], [[Agentic Git Sync]], [[git-annex assistant]], [[Syncthing]], and [[Jujutsu]]. Distinguished tracked/shared content, local metadata, optional payload/version stores, and logical remote refs.
- Checked additional database/workspace paths against the previously pinned source snapshots. Marked examples, optional/configurable paths, packed refs, and alternative placements; no competitor runtime tests.

## [2026-11-06] correction | Remote branch trees versus local competitor storage
- Corrected all seven competitor profiles to lead with remote tracked branch contents, matching the intended scope of [[Device branch format]]. Relabeled the earlier trees as local filesystem layouts, not remote contents.
- Explicitly distinguished Syncthing's lack of Git branches, git-annex's separate metadata branch and payload store, LFS pointers, submodule gitlinks, and Jujutsu's conditional tracked conflict-tree entries from local `.git/` and `.jj/` directories.
- Added the distinction to [[Competitor storage approaches]]. Checked relevant paths against the pinned source snapshots; documentation-only.

## [2026-11-06] discovery | AI-managed synchronization and human decisions
- Created [[AI-managed synchronization and human decisions]] in `04_Discovery/` from Kai's proposal to hide Git mechanics with AI assistance and escalate ambiguous intent. Listed in [[index]].
- Added eight concrete scenarios, user-facing wording, engine/AI/human boundaries, six prompt templates, a synthetic Chat Completions input/response, stale-input and execution checks, open architecture questions, and an evaluation matrix.
- Marked all new behavior as exploratory and preserved current ADR constraints. Checked official API documentation and current summary code; parsed all example JSON, verified the nested response and request syntax. No AI provider called, no workspace content shared, and no implementation changed.

## [2026-11-06] discovery | Human-approved AI conflict proposals
- Updated [[AI-managed synchronization and human decisions]] and its [[index]] entry following Kai's clarification: the primary case is a technical text conflict whose meanings AI can combine, with mandatory human approval of the exact proposed text.
- Added a fuller meaning-level example and review/edit/reject flow. Revised prompts, action contract, synthetic API response and execution checks to request candidate approval rather than automatic application; checked JSON consistency.
- Kept contradictory intent (Monday/Tuesday) as a secondary case with unverified frequency. Moved general content-consistency checking and P3 outside initial scope; clean merges do not trigger AI editorial review. No implementation or live provider call.

## [2026-11-06] ticket | US00004 AI-proposed text conflict merges with human approval
- Created [[Story - US00004 - AI-proposed text conflict merges with human approval]] from case 2 of [[AI-managed synchronization and human decisions]] and Kai's explicit requirement for approval of the exact merged text. Backlog, medium priority, rank 3000, owner Kai; release, size and assignee unset.
- Duplicate check found no matching ticket. Recorded source/code context, compatible-intent example, initial scope/exclusions and 8 refinement questions. Acceptance criteria, implementation plan, test plan and review/test counters remain unfilled as required by create-ticket.
- Listed in [[index]] and linked back from discovery case 2. No tracker/chat integration configured; wiki is the only ticket record. Documentation only; no implementation or provider calls.

## [2026-11-06] discovery | Classical Git workflow for Mesh
- Created [[Classical Git workflow for Mesh]] in `04_Discovery/`, comparing ordinary shared commits, device branches with normal merges, and ordinary device trees with session-level publication. Listed in [[index]].
- Discussed benefits, nine disadvantages with concrete mitigations and remaining tradeoffs, remote branch trees, a collaboration scenario, AI/human/engine responsibilities, compatibility/migration questions and prototype criteria.
- Preserved human approval of AI-written resolutions and excluded general consistency review. Kept the architecture exploration independent of US00004 and current binding ADRs. Checked official Git documentation; no implementation, live test, migration or new ticket.

## [2026-11-06] refine | US00003 Exclude device-local files from sync
- Started refinement of [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]] and moved it to `Refinement`. Checked the context against `src/engine.rs` (`scan`, `install`, `ignored`) and `README.md`; the claims match.
- Added an eighth open question: unpublished drafts and open conflicts on a newly excluded path. No chat integration, so the questions are recorded in the ticket.

## [2026-11-06] create-ticket | Remaining 0.3 tray work
- Created [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] (5 open questions), [[Story - US00006 - Conflict list and resolution in the status window|US00006]] (5), [[Story - US00007 - Recent-save presence in the status window|US00007]] (4), [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] (5), [[Story - US00009 - Pause and resume synchronization|US00009]] (6) and [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]] (3), all Backlog, v0.3.0, ranks 4000–9000 in the suggested order. Listed in `index.md`.
- [[Dispatch Mesh milestone plan]]: 0.3 now lists its tickets in the suggested order.

## [2026-11-06] decision | ADR-0011 accepted
- [[ADR-0011 — Build the desktop host with Tauri 2]] accepted by Kai after US00001 was built and verified (CI green on Windows and Linux). Updated `index.md` and the milestone plan's decision table note.
- Ticket template: the `open_findings` guide now follows the one-review-per-story rule.
- Kai answered all eight questions: shared `.meshignore` only; `main` records an attributed removal while devices keep their files; incoming changes to excluded paths are dropped silently; `init` writes the Obsidian rules visibly; no CLI command; re-including uses the ordinary change path; the removal settles open conflicts; target v0.3.0. Rewrote the scope, wrote 14 acceptance criteria, set `open_questions: 0`. Refinement stays open until Kai ends it.

## [2026-11-06] plan | US00003 implementation plan
- Wrote the implementation plan in [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]]. The core rule: a computed tree never holds a path its own ignore policy excludes, and on disk an excluded path is left untouched. Nine steps, test matrix, risks, about 3–4 days. ADR extraction waits for sign-off.

## [2026-11-06] update | Mermaid tree layouts in competitor profiles
- Converted the remote-branch and local-filesystem ASCII trees in all seven competitor profiles to Mermaid treeView-beta, following [[Agentic Git Sync]]. Preserved hierarchy, paths, annotations, surrounding source citations, and research dates; removed the duplicate ASCII version of the existing reference diagram. Verified that surrounding prose is unchanged and no ASCII tree blocks remain in the profiles.

## [2026-11-06] decision | ADR-0013 Derive exclusion from each tree's own ignore policy
- Kai signed off the US00003 plan by invoking development. Created [[ADR-0013 — Derive exclusion from each tree's own ignore policy]] (`proposed`) and listed it in [[index]]. Moved US00003 to `In progress`.

## [2026-11-06] refine | US00005 Workspace detail with Finish session and recovery information
- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] moved to Refinement. Checked the engine (Status fields, saved copies, finish, sync retry) and recorded the facts. Kai answered all 5 questions: extend `Status` (amends ADR-0010), activity = unsettled + last 10 settled, saved copies with Show in folder, one Finish-session item per open session and no Quit prompt, no Retry now. 13 acceptance criteria drafted; `open_questions: 0`.

## [2026-11-06] develop | US00003 Exclude device-local files from sync
- Implemented on branch `us00003-exclude-device-local-files` (`25ca6c5`, `2ae22f6`): tree ignore policy (`src/policy.rs`), exclusion in scan/capture/render/publish/apply, Obsidian default in `init`, "Stopped synchronizing" wording, README and AGENTS.md. Gates green; CLI demonstration on two devices recorded. As-built notes in [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]]. Next: `/code-review US00003` in a fresh session.

## [2026-11-06] implementation-plan | US00005 Workspace detail with Finish session and recovery information
- Wrote the implementation plan in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]: additive `Status` fields (session times, title, purposes, `last_sync`, `saved_copies`) with an ADR-0010 amendment, tray model rules, host `finish`/saved-copy resolution, Tauri wiring with `tauri-plugin-opener`, window. Five decisions listed for review. Awaiting sign-off; ADR extraction follows it.

## [2026-11-06] develop | US00003 code review fixes
- Kai decided F2: ignore-rule files always synchronize (recorded in [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]] as an answered question and AC 15, and in [[ADR-0013 — Derive exclusion from each tree's own ignore policy|ADR-0013]]). Fixed F1 and F2 and added F3's provider test in `9b98471`; gates green. `open_findings` cleared to empty for the review's re-check.

## [2026-11-06] refine | US00006 Conflict list and resolution in the status window
- [[Story - US00006 - Conflict list and resolution in the status window|US00006]] moved Backlog → Refinement. Verified against the engine: `Status` carries only id/path/binary per conflict; the record attributes only the other version; `Workspace::resolve` refuses after later edits, contradicting the design frame's "reconciled separately". 8 questions recorded and answered by Kai: widen the same window, extend `Status` + on-demand detail (amends ADR-0010), derive current-version attribution, later-edit reconciliation as a separate engine story, image previews (PNG/JPEG/GIF/WebP up to 10 MB, no SVG) via a host command, 1 MB / UTF-8 compare limit, deletion placeholder, next conflict after resolving. 12 acceptance criteria drafted; `open_questions: 0`.

## [2026-11-06] ticket | US00010 Resolve a conflict after the file was edited again
- Created [[Story - US00010 - Resolve a conflict after the file was edited again|US00010]] (Backlog, v0.3.0, high) from [[Story - US00006 - Conflict list and resolution in the status window|US00006]] Q4: the engine reconciles a later local edit onto the chosen version instead of refusing with "File has newer edits". Engine facts verified against `src/engine.rs`; 6 open questions. Linked from US00006 (Q4 answer, Follow-ups) and listed in [[index]].

## [2026-11-06] code-review | US00003 fixes approved at 9b98471
- Re-checked the original review's fixes in [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]]. F1/F2 and F3's coverage gap are closed; `open_findings: 0`. Independently repeated the original CLI failures successfully and reran formatting, Clippy and all 115 tests on Windows. No source changes or status transition; `/test-plan US00003` is next.

## [2026-11-06] test-plan | US00003 manual test plan and freeze
- Wrote the manual test plan for [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]] against reviewed build `9b98471`: three checks (real Obsidian navigation, the live GitHub publication, two vaults in Obsidian). Set `open_tests: 3`, stamped `frozen: 2026-11-06`, moved the ticket to `Review`.

## [2026-11-06] test | US00003 manual tests passed
- Ran all three manual checks of [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]] with Kai on build `9b98471`; all passed. Set `open_tests: 0`. Recorded an observation: Obsidian rewriting a freshly seeded `app.json` on first open caused one ordinary, resolved conflict in the test vaults. The ticket stays in `Review` for Kai to move.

## [2026-11-06] merge | US00003 merged to main
- Fast-forwarded `main` to `9b98471` (three US00003 commits) and pushed. Recorded in [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]].

## [2026-11-06] develop | US00005 Workspace detail with Finish session and recovery information
- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] → In progress (assignee Kai) and implemented on branch `US00005-workspace-detail`: new `Status` fields, tray activity list, Finish session in the window and the tray menu, saved copies with Show in folder. As-built notes record the deviations (`has_remote` added, author name instead of "You").
- Amended [[ADR-0010 — Report workspace status and session stages separately]] with the workspace-detail fields, the coalescing of `last_sync` and saved-copy recording.

## [2026-11-06] ticket | US00011 Limit saved copies in the recovery folder
- Created [[Story - US00011 - Limit saved copies in the recovery folder]] (Backlog, no version target) from the saved-copy growth observed in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]; 5 open questions. Linked from US00005's Follow-ups and listed in [[index]].

## [2026-11-06] code-review | US00005 blocked at 4133e9c
- Recorded the independent review in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]: two blocking findings (phantom saved copies after interrupted displacement; AC 5 disagrees with planned/built activity ordering), no additional non-blocking findings. Set `open_findings: 2`, retained `In progress`, and left `open_tests` and the contract zone unchanged. Added a record-zone fix-round plan and retained disposable reproduction evidence in `target/us00005-review-evidence.json`. Formatting and Clippy passed independently on Windows; the full test result is recorded in the ticket when complete. No source changes or new wiki pages.
- Final gate result for this review: full `cargo test` passed on Windows, 132 tests and zero failures, including both workspace packages and doc-test targets. Updated the review entry with this result; the two blockers remain.

## [2026-11-06] implementation-plan | US00006 Conflict list and resolution in the status window
- Wrote the implementation plan in [[Story - US00006 - Conflict list and resolution in the status window|US00006]]: `open_waiting` lock helper, in-memory `ConflictStatus` side metadata plus on-demand `conflict_detail` (ADR-0010 amendment), host detail/preview/resolve, tray model with `similar` line marks, Tauri commands and window layout, window tabs and detail. Kai decided at planning that the list shows "Shared history" without a Git lookup (criterion 2 adjusted); corrected the refinement claim about non-UTF-8 text (criterion 5). Seven decisions listed for review. Awaiting sign-off; ADR extraction follows it.

## [2026-11-06] develop | US00005 code review fixes
- Fixed F1 in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] (phantom saved copy after an interrupted displacement) with five regression tests. Resolved F2 with Kai's decision to keep the planned activity policy: AC 5 and Scope amended (contract not yet frozen). Recorded in As-built notes; cleared `open_findings` for the re-check.

## [2026-11-06] code-review | US00005 fix re-check at 47f810e
- Independently re-checked F1/F2 in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] against `47f810e`. Repeated the original CLI reproduction successfully and retained evidence at `target/us00005-recheck-evidence.json`; verified the amended activity contract against the implementation and new regression. Formatting, Clippy and diff whitespace checks passed on Windows; final full-suite result and counter follow when complete. No source or contract changes; status and `open_tests` unchanged.
- Final re-check verdict: clear at `47f810e`; both original blockers resolved, no additional findings. Independently passed all mechanical gates on Windows, including all 138 tests (zero failures). Set `open_findings: 0`; `/test-plan US00005` is next. Status remains `In progress`, `open_tests` and freeze unchanged.

## [2026-11-06] test-plan | US00005 manual test plan and freeze
- Before writing the plan, verified AC 9, AC 10, AC 6 (restart) and AC 11 (Explorer selection) in the running tray. Found and fixed an AC 9 defect: the dependent-edits reason was masked as a capture fault or sync failure, so Finish session was never offered (`819d9b0`, two regression tests; as-built notes). Gates green after the fix.
- Wrote the manual test plan in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] (4 checks: tray menu, keyboard, Narrator, dark theme), set `open_tests: 4`, stamped `frozen: 2026-11-06`, moved the ticket to Review.

## [2026-11-06] test-plan | US00005 Narrator check removed
- At Kai's request, removed the Narrator check from the [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] test plan before testing (`open_tests` 4 → 3); noted in the plan that AC 13's screen-reader names are verified by code only.

## [2026-11-06] test | US00005 manual tests passed
- Kai ran the three remaining manual checks of [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] (tray menu, keyboard, dark theme) in a guided session; all passed. Ticked them and set `open_tests: 0`; results recorded in the test plan. Status stays Review.

## [2026-11-06] ticket | US00012 Status window layout follows the design frames
- Created [[Story - US00012 - Status window layout follows the design frames]] (Backlog, v0.3.0) from a comparison of the built US00005 window with design frames pp. 3, 4 and 13; 4 open questions. Noted that the PDF and HTML in `06_UX/` have no readable text. Linked from US00005's Follow-ups and listed in [[index]].

## [2026-11-06] merge | US00005 merged to main
- Rebased the five US00005 commits onto `51cf60e` and fast-forwarded `origin/main` to `676092d`. The local `main` branch is checked out in the worktree `dispatch-mesh-wiki-lf` and was left untouched (behind by 5). Recorded in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]].

## [2026-11-06] design | Readable design source in 06_UX
- Added [[Dispatch Mesh desktop app — design spec]], one PNG per frame in `06_UX/frames/` (from the PDF export) and the frame markup in `06_UX/frames/source/` (canvas state of 2026-11-06). The PDF and bundled HTML have no extractable text. Answered Q4 of [[Story - US00012 - Status window layout follows the design frames|US00012]] (`open_questions` 4 → 3) and pointed it at the new files. Listed in [[index]].

## [2026-11-06] ticket | BUG00001 AI summaries of large sessions never finish
- Created [[Bug - BUG00001 - AI summaries of large sessions never finish]] (Backlog, high) from Kai's report of a session stuck in Finishing up in `dispatch/wiki`, with the diagnostic-build evidence; explicit `/fix-bug` shortcut requested. Listed in [[index]].

## [2026-11-06] fix-bug | BUG00001 AI summaries of large sessions never finish
- Fixed under the explicit small-bug shortcut on branch `BUG00001-summary-lock-wait` (`403b165`): the summary step now waits up to 30 s for a busy workspace lock before and after the AI request. Two regression tests; gates green; verified live on `dispatch/wiki` (session published about a minute after finishing). Independent review omitted by request. [[Bug - BUG00001 - AI summaries of large sessions never finish|BUG00001]] → Done (v0.3.0), frozen and completed 2026-11-06.

## [2026-11-06] merge | BUG00001 merged to main
- Fast-forwarded `main` to `403b165` (the BUG00001 fix) and pushed. Rebuilt `target/release` from it. Recorded in [[Bug - BUG00001 - AI summaries of large sessions never finish|BUG00001]].

## [2026-11-07] implementation-plan | US00006 plan reworked for US00005 as built and the design spec
- Rewrote the plan in [[Story - US00006 - Conflict list and resolution in the status window|US00006]] against `main` at `403b165` and [[Dispatch Mesh desktop app — design spec]]: reuse US00005's 30 s lock wait via a host helper (no engine API change), `edited_after` in the on-demand detail, 1000 × 760 detail layout, and US00012's tokens and components. Kai decided four design points (author button suffix, an honest later-edit note with disabled buttons until US00010, a secondary "Review and choose", no "saved copy" for conflict sides). Criteria 7, 8 and 10 adjusted and 13 added (frame comparison). US00012 is developed first. Awaiting sign-off.

## [2026-11-07] design | Icons exported as SVG
- Exported the tray icons (5 states × light, dark, macOS template) to `06_UX/icons/tray/` and the status glyphs (5 states × light, dark) to `06_UX/icons/status/`, 25 files. Referenced from [[Dispatch Mesh desktop app — design spec]] (*Glyphs*).

## [2026-11-07] refine | US00012 Status window layout follows the design frames
- [[Story - US00012 - Status window layout follows the design frames|US00012]] moved Backlog → Refinement. Checked against `main` at `403b165`: sessions carry no device (no "You", no received segment), device ids are UUIDs with no readable name, the window is 440 × 520. Added Q5–Q9; Kai answered all 8: native title bar plus the design's top row, nav now with a Conflicts placeholder, bundled IBM Plex, `SessionStatus.device` (amends ADR-0010), no device names, 440 × 640, wording per state and reason, avatar colors from an email hash. Scope widened; 11 acceptance criteria; `open_questions: 0`.
- Aligned [[Story - US00006 - Conflict list and resolution in the status window|US00006]]: no device names on conflict sides, "You" via the session device, nav and placeholder from US00012.

## [2026-11-07] design | Frames and design spec follow the US00012 and US00006 refinement decisions
- Updated the design canvas, `06_UX/frames/source/` and [[Dispatch Mesh desktop app — design spec]] to the 2026-11-07 decisions in [[Story - US00012 - Status window layout follows the design frames|US00012]] (native title bar, tabs and Conflicts placeholder, bundled Plex, "You", no devices, 440 × 640, reason and reassurance lines, avatar colors) and [[Story - US00006 - Conflict list and resolution in the status window|US00006]] (author suffix on buttons, later-edit note with disabled buttons, secondary Review and choose, binary wording). New frame 3a2 for the Conflicts tab placeholder. The frame PNGs still show the PDF export of 2026-11-05.

## [2026-11-07] design | Frame PNGs re-rendered
- Re-rendered all 14 frame PNGs in `06_UX/frames/` from the current canvas source (2× scale), including the new `03a2-conflicts-tab.png`. They now show the 2026-11-06 and 2026-11-07 changes. IBM Plex wasn't available to the renderer, so the PNGs use Inter and DejaVu Sans Mono as stand-ins; noted in [[Dispatch Mesh desktop app — design spec]].

## [2026-11-07] implementation-plan | US00012 Status window layout follows the design frames
- Wrote the implementation plan in [[Story - US00012 - Status window layout follows the design frames|US00012]] against `main` at `403b165` and the design spec with its exported SVG glyphs: `SessionStatus.device` (ADR-0010 amendment), model fields for glyph, reason, reassurance, Finish placement, avatar, segments and stage text, verbatim status SVGs via `<picture>`, bundled IBM Plex, 440 × 640 window, restyled structure with the Activity/Conflicts nav, a CSS contrast test, and the design-spec update. Kai decided at planning that the meta line keeps the start time and a purpose line appears when recorded (frozen US00005 AC 1/3 stay met; criteria 3 and 5 adjusted). Seven decisions listed for review. Awaiting sign-off; ADR extraction follows it.

## [2026-11-07] create | BUG00002 One large text file makes the whole AI summary fall back
- Created [[Bug - BUG00002 - One large text file makes the whole AI summary fall back|BUG00002]] in Backlog from Kai's tray report: commit `d49570f` (session `c65c0d1e`) fell back to the local message because the deleted 1.36 MB `Dispatch Mesh desktop app.html` pushed the summary input over 512 KiB. Explicit `/fix-bug` shortcut requested. Listed in [[index]].
