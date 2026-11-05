---
id: US00002
type: story
status: Review
priority: high
rank: 2048
version_target: v0.2.5
size:
assignee: Kai
owner: "[[Kai]]"
open_questions: 0
open_tests: 3
open_findings: 0
discussion:
updated: 2026-11-05
completed:
frozen: 2026-11-05
---
Define, and record as an ADR, how a desktop host drives the Mesh engine for a workspace: how it reads state, triggers actions such as sync and finish session, and is stopped, while CLI intent and finish commands keep working. Then implement the minimum the 0.3 tray needs. This is the 0.2.x milestone item "Define the local engine/control interface that the tray host will use", and it settles open question 2 of [[Story - US00001 - Tray icon and status window host|US00001]].

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As the developer of the Mesh tray host, I want one defined way to observe and control a running workspace engine, so that the tray, the CLI and later agents all use the same engine operations and state, with no duplicate engines and no lost edits.

## Context

- Milestone source: [[Dispatch Mesh milestone plan]] → *0.2.x — engine stabilization*: "Define the local engine/control interface that the tray host will use. Keep CLI intent and finish actions usable while Mesh runs." The same plan lists "local control interface" among the decisions to close before implementing 0.3. *Product decisions* require the engine to stay independent of the desktop host.
- 0.3 depends on it: the tray must "use the same engine operations and state as the CLI" and "prevent duplicate engine instances for a workspace" ([[Dispatch Mesh milestone plan#0.3 — tray application|0.3]]).
- How processes coexist today (0.2.4):
  - `run` holds an exclusive `fs2` lock on `.mesh/runner.lock`, which already rules out a second runner.
  - Commands that change state take the workspace operation lock (`.mesh/lock`); the capture loop retries contention.
  - Network work takes `.mesh/network.lock`.
  - `status` reads a committed SQLite snapshot without the operation lock.

  So the CLI already works alongside a runner through shared on-disk state, with no IPC. Tests cover this: `runner_startup_survives_a_concurrent_workspace_operation` and `status_remains_readable_while_an_operation_holds_the_workspace_lock` (`tests/cli.rs`).
- What on-disk coexistence does not give a host:
  - **Change notification.** `status` must be polled.
  - **Waking the runner.** For example, to publish right after `session finish` instead of waiting for the next cycle.
  - **Stop and pause.** A way to tell a runner it does not own to stop or pause.
- The decision interacts with US00001's open questions on in-process embedding vs. a supervised `run` process, and on one binary vs. two (a `windows_subsystem = "windows"` binary has no console for the CLI).
- **Refinement findings (2026-11-05, from `src/main.rs` `watch`):**
  - **No wake channel needed.** The runner ticks every second, calls `close_idle` on every tick, and starts a new `sync()` whenever none is running, no capture is pending, and 3 s have passed since the last one. A CLI `session finish` is therefore picked up within seconds without any signal.
  - **Ctrl+C is the only shutdown path.** `watch` stops only on `tokio::signal::ctrl_c()`, which captures and returns. A host needs a shutdown trigger it can fire. Sending Ctrl+C to another process on Windows is unreliable: `GenerateConsoleCtrlEvent` requires a shared console. That favors an in-process engine or an explicit stop command.
  - **Runner failures are console-only.** Except for sync failures, they reach only the console. `report_failure` prints capture, sync and summary failures. Only sync failures are also persisted, as `state.attention = "Synchronization waiting: …"`. AI summary failures ("Summary deferred") are never written to state, so a host reading the status snapshot cannot see them. Offline appears as a sync-failure attention.
  - **The run loop is not in the library.** `watch`, `summarize_ready` and `show_status` live in `src/main.rs`; `src/lib.rs` exports only `ai`, `codec`, `engine`, `git`, `model`, `storage`, `transport`. Sharing them with a host requires moving them into the crate.
- Invariants that bind the design (`AGENTS.md`):
  - Never discard uncaptured edits.
  - Keep capture going while network or AI requests are pending.
  - Never expose credentials or response bodies in diagnostics.
  - Users never need Git knowledge.

## Scope

- An ADR in `07_Engineering/Decisions/` choosing the host↔engine model: in-process library, a supervised runner with local IPC, or on-disk state plus a small signal channel. It covers how state is read, how actions are invoked, how state changes are observed, how a host stops a runner it owns, and how the design extends to macOS and Linux (0.7/0.8).
- Library boundary: the operations a host calls (open, run loop, status snapshot, sync now, finish session, shutdown with capture) exposed from the crate so `main.rs` and a host share one implementation.
- Whatever the ADR selects as the minimum for US00001: state observation without unbounded polling cost, plus orderly shutdown that captures saved changes the way Ctrl+C does.
- Duplicate-engine prevention stays in force for any host, with a clear user-facing error.
- Regression tests: CLI `status`, `session finish` and intent commands work while a host-owned engine runs, and shutdown preserves captured work.

## Out of scope

- The tray UI itself ([[Story - US00001 - Tray icon and status window host|US00001]]).
- The richer status states ([[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]]); this ticket transports whatever the status model contains.
- Pause semantics, which are defined with the 0.3 pause/resume work.
- Remote or authenticated access (cloud MCP, 2.0).

## Open questions

- [x] **Model.** Should the host embed the engine in-process, supervise a `run` child process, or talk to an independently started runner? Embedding is the simplest; supervising isolates a UI crash from capture.
  **Answer (Kai, 2026-11-05):** in-process. The run loop moves into the library with a shutdown handle and a status/event channel. The tray calls it directly, and the CLI keeps working against the same workspace through the existing on-disk locks. Accepted trade-off: a UI crash stops capture until the host restarts, and captured work survives through the existing journals.
- [x] **Observation.** Can hosts poll the committed SQLite snapshot cheaply enough (and how often), or does the engine need to push changes (in-process channel, named pipe, file-change watch on the state DB)?
  **Answer (Kai, 2026-11-05):** the runner publishes. After each tick it computes the status snapshot (the same data `status` returns) and publishes it on a watch channel, but only when it differs from the last one. Changes made by the CLI in another process are included, because the runner re-reads workspace state on every tick. Latency is at most about one tick (1 s), and the host does no polling of its own.
- [x] **Waking and stopping a runner from another process.** Is a control channel needed in 0.3 at all (for example, CLI `session finish` waking the tray-owned runner to publish immediately), or is "the next sync cycle picks it up" acceptable?
  **Answer (follows from the in-process decision and the run-loop finding, 2026-11-05):** no cross-process channel in 0.3. The host stops its own in-process engine through the shutdown handle. The runner already picks up CLI `session finish` within its ~3 s sync cadence.
- [x] **Channel, if any.** Named pipe on Windows and a Unix socket elsewhere, or a localhost TCP port? Either way, how is access restricted to the current user?
  **Deferred (2026-11-05):** not needed with the in-process model. Revisit if a later milestone needs cross-process control (e.g. attaching to a CLI-started runner, or the 2.0 MCP host). The ADR records this as a known extension point.
- [x] **Library API shape.** Should the crate expose a blocking API, an async (tokio) API, or both? The current run loop is tokio-based and `main.rs`-internal.
  **Answer (Kai, 2026-11-05):** both, layered.
  - `Runner::start(root) -> RunnerHandle` runs the loop on a background thread with its own tokio runtime. The handle exposes `status()` (watch receiver of the status snapshot), `stop()` (capture saved changes, then end the loop, as Ctrl+C does today) and `join()`.
  - Underneath, a public async `run(root, shutdown, status_tx)` is what both the handle and the CLI's `run` command use.
  - UI toolkits keep the main thread, and UI code never needs tokio.
- [x] **Failure reporting to hosts.** Should the runner persist capture and AI-summary failures into engine state (as sync failures already are), or emit them as events, so that a host can show the cause of a failed operation? Today they are printed only.
  **Answer (Kai, 2026-11-05):** persist, and include in the snapshot.
  - The latest AI-summary failure is persisted in workspace state, as sync failures already are, so CLI `status` shows it too. It is cleared when a summary succeeds.
  - The runner's current capture, sync and summary errors are part of the status snapshot it publishes.
  - No separate event stream.
  - How these failures are classified and presented is part of the status model in [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]]. This ticket only guarantees they reach the snapshot and state.
  - Per the invariants, messages must not contain credentials or provider response bodies.
- [x] **Lock ownership on handover.** If a host starts while a CLI `run` holds `runner.lock`, does the host attach to that runner as an observer, or refuse to start and offer to take over?
  **Answer (Kai, 2026-11-05):** refuse, explain and retry. `Runner::start` fails with a distinct "already running" error when `runner.lock` is held, and the host never starts a second engine. The host shows: "Mesh is already running for this workspace from another window. Stop it there, then choose Retry." Retry attempts the lock again. There is no observer mode.

## Acceptance criteria

1. An ADR in `07_Engineering/Decisions/` records the in-process host↔engine model, the `RunnerHandle` API, snapshot publication, failure reporting and refuse-on-lock behavior. It names cross-process control (named pipe or Unix socket) as a deferred extension point.
   *Update 2026-11-05:* this is [[ADR-0009 — Embed the engine with a runner handle and published status]], recorded as `proposed`. When this ticket reaches `Done`, set ADR-0009 to `status: accepted`, and amend it first if the built API deviates.
2. The run loop, AI summarization and status-snapshot computation live in the library crate (not `src/main.rs`). The CLI `run` and `status` commands call those library functions, and their observable output is unchanged except for the failure fields in criterion 6.
3. `Runner::start(root)` returns a `RunnerHandle` without blocking the calling thread. The engine runs on a background thread with its own tokio runtime, so the caller needs no tokio runtime.
4. `RunnerHandle::stop()` captures saved changes and ends the loop without waiting for an in-flight HTTP or AI request, as Ctrl+C does for `run` today. After `join()`, a reopened workspace contains every change saved before `stop()`.
5. `RunnerHandle::status()` delivers a new snapshot within about one tick (≤ 2 s) after a change, and none while nothing changed. That includes a change made by a separate CLI process, such as `session finish` or an intent command.
6. The latest AI-summary failure is persisted in workspace state, cleared on the next successful summary, and shown by CLI `status`. The published snapshot carries the runner's current capture, sync and summary errors. No message contains a credential or a provider response body.
7. `Runner::start` on a workspace whose `runner.lock` is held (by CLI `run` or another handle) returns a distinct "already running" error and starts nothing. A later call succeeds once the lock is released.
8. Automated tests cover criteria 3–7, including CLI `status`, `session finish` and an intent command succeeding against a workspace while a `RunnerHandle` runs it. `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test` pass.

## Implementation plan

*Drafted 2026-11-05 against `main` at `3c2d513` (branch `task00001-stabilization-gaps`, with TASK00001 step 1 uncommitted in `src/git.rs`). Development started 2026-11-05 via `/develop US00002`, on `us00002-engine-control-interface` off `main` at `41a543b` (TASK00001 merged).*

### Ground truth that shapes the plan

- **TASK00001 builds half of this ticket.** Its plan, whose review points Kai approved on 2026-11-05, delivers:
  - `state.summary_failure` (step 2);
  - `ai::generate` returning the failure instead of printing it (step 3);
  - a lock-free `Workspace::status_snapshot()` in a new `src/status.rs`, which `show_status` delegates to (step 4).

  None of it is in the tree yet. This plan builds on those pieces and does not build them a second time.
- **AI failures never reach the runner today.** `ai::generate` (`src/ai.rs:250`) catches the provider error, `eprintln`s it and returns the fallback message. So the `summary_error` that `watch` tracks (`src/main.rs:344`, `:372`) only ever holds workspace I/O failures from `summarize_ready`. Criterion 6 therefore depends on TASK00001 step 3.
- **`status` scans the live folder.** `show_status` calls `workspace.scan()` (`src/main.rs:221`) to compute `local_changes_waiting` and `capture_error`. Doing that on every tick would rescan the whole tree once a second, the "unbounded polling cost" the scope rules out. The runner already knows the answer from its capture step: `candidate` holds a scan that differs from `observed` and is waiting to stabilize.
- **Stop is only checked between ticks.** In `tokio::select!`, once the tick branch is chosen its body runs to completion, including the `.await` on the capture `spawn_blocking`. Ctrl+C is only checked between ticks. An in-flight sync (`spawn_blocking`) or summary (`tokio::spawn`) keeps running, and the Ctrl+C path never awaits either. That matches criterion 4.
- **Shutdown capture can fail on a busy lock.** The Ctrl+C path calls `Workspace::open` once (`src/main.rs:354`). `open` fails at once with `WorkspaceBusy` when the operation lock is held, for example by a background sync's `network_call` while it reacquires the lock (`src/engine.rs:232`) or by a CLI command. Today, `run` then exits with an error and without the final capture. Criterion 4 needs a bounded retry.
- **Two ways to block on stop.** Dropping a tokio runtime waits for blocking tasks still running, here a sync stuck on Git. The CLI never notices, because the process exits. A `RunnerHandle` has to shut its runtime down with `shutdown_background()`, or `join()` would wait for the network.
- **The runner lock rejects a second handle in the same process.** `fs2` uses `flock` on Unix, which is per open file description, and `LockFileEx` on Windows, which is per handle. So two `Runner::start` calls within one process conflict exactly like two processes do.
- **Startup is cheap and synchronous.** Before the loop starts, `watch` only calls `Workspace::inspect`, takes the lock and creates the `notify` watcher (`src/main.rs:318`–`334`). The decided API returns a "distinct error and starts nothing" from `start`, so this part runs on the calling thread.

### Steps, in order

1. **Prerequisite: TASK00001 steps 2–4 are merged.**
   - *Why first:* criterion 2's "status-snapshot computation in the library" and criterion 6's persisted summary failure are that code. Starting before it lands means either building the status computation twice or rebasing a moved `watch` onto TASK00001's changes to `main.rs:413`.
   - One adjustment to TASK00001 step 4 is needed (see *Cross-ticket effects*). Split the snapshot into a scan-free core, `Workspace::status_from(&self, local: LocalObservation) -> Status`, where `LocalObservation { changes_waiting: bool, capture_error: Option<String> }`. `status_snapshot()` becomes "scan, then `status_from`". `Status` derives `Clone, PartialEq, Serialize`. If TASK00001 has already landed without this split, this step makes it as a pure refactor.
2. **Move the loop into the library** (new `src/runner.rs`; `src/lib.rs` adds `pub mod runner;`).
   - Move `watch`, `summarize_ready` and `report_failure` verbatim, then restructure:
     - `pub fn acquire(root: &Path) -> Result<RunnerLease>` covers inspect, opening `.mesh/runner.lock`, `try_lock_exclusive` and creating the watcher. When the lock is held, it returns a new typed `AlreadyRunning` error. Its `Display` stays "Mesh is already running for this workspace", so CLI output is unchanged. Callers use `downcast_ref`, as they do for `WorkspaceBusy`. `RunnerLease` owns the locked `File`, the watcher and its receiver, the canonical root and the branch.
     - `pub async fn run(lease: RunnerLease, shutdown: impl Future<Output = ()>, status: Option<watch::Sender<Option<RunnerSnapshot>>>) -> Result<()>` is the existing loop with `tokio::signal::ctrl_c()` replaced by `shutdown`.
     - `pub async fn summarize_ready(root)` stays public, because CLI `sync` calls it.
   - CLI-only text moves out of the library. `main.rs` calls `acquire`, prints the "Mesh is running for … Ctrl+C saves local work" banner, and `run(lease, ctrl_c, None)`. After `Ok`, it prints "Local changes saved. Mesh stopped." The order of lock, banner, loop and message, and the wording, stay as today (criterion 2).
   - The transition lines ("Synchronization resumed.", "Local capture resumed.", the `report_failure` stderr lines) stay in the loop. In a `windows_subsystem` host they go to a null handle, which Rust's stdio treats as a sink. The same information reaches the snapshot (step 4).
   - *Deviation from ADR-0009:* the ADR names `run(root, shutdown, status_tx)`. Splitting out `acquire` lets the CLI keep its banner order and lets `Runner::start` return the lock error synchronously. Amend ADR-0009 at `Done`, as criterion 1 already requires.
   - *Why this order:* a verbatim move with only the `shutdown` substitution keeps the diff reviewable, and the existing `tests/cli.rs` runner tests must pass unchanged at this step. Behavior changes come in steps 3–5.
3. **Orderly shutdown** (`src/runner.rs`).
   - The shutdown branch opens the workspace with a bounded retry on `WorkspaceBusy`: up to 10 s at 25 ms, mirroring `network_call`'s reacquire loop (`src/engine.rs:232`). Then it runs `capture_at(now())` and returns.
   - It never awaits `synchronization` or `summary`. Dropping the `JoinHandle`s detaches them.
   - *Why 10 s:* the operation lock is only held for local work. `network_call` releases it during Git. So a wait that long means something is wrong, and failing with the error beats hanging the host's Quit. Captured work is still on disk (ADR-0007).
   - This fixes the busy-lock gap for CLI `run` as well.
4. **Snapshot publication** (`src/runner.rs`).
   - `pub struct RunnerSnapshot { pub status: Status, pub capture_error: Option<String>, pub sync_error: Option<String>, pub summary_error: Option<String> }` derives `Clone, PartialEq, Serialize`. The three errors are the loop's current in-memory errors (message only, without the `Instant`).
   - In the capture `spawn_blocking`, after `close_idle`, the task computes `workspace.status_from(LocalObservation { changes_waiting: next.is_some(), capture_error: None })` from the workspace it already opened. That reuses the state it just loaded from SQLite, which also holds changes from separate CLI processes, so no extra open or scan is needed.
   - When capture fails (including `WorkspaceBusy`), the loop falls back to `Workspace::inspect(&root)?.status_from(…)` with `capture_error` set (busy is filtered, as in `report_failure`). A CLI command holding the lock therefore cannot delay a snapshot past one tick.
   - The tick then calls `status.send_if_modified(|current| …)` and replaces the value only when it differs, so subscribers wake only on change (criterion 5).
   - `presence` depends on the wall clock (300 s window). Its entries changing when a save ages out is a real change and is published. No other snapshot field is time-derived.
   - *Why `Option`:* the watch channel needs an initial value. A synchronous snapshot in `start` would need a full scan on the caller's thread. The first `interval` tick fires immediately, so `None` lasts milliseconds.
5. **Handle** (`src/runner.rs`).
   - `pub struct Runner;` with `pub fn start(root: &Path) -> Result<RunnerHandle>`:
     1. Call `acquire(root)?` on the calling thread, so criterion 7's error is synchronous and nothing is spawned on failure.
     2. Create `watch::channel(None)` for the status and `watch::channel(false)` for the stop flag.
     3. Spawn a `std::thread` named `mesh-runner` that builds a current-thread runtime with `enable_all()` and calls `block_on(run(lease, stop_rx.wait_for(|s| *s), Some(status_tx)))`. It then calls `runtime.shutdown_background()`, so the call does not wait for detached Git or HTTP work, and returns the result.
   - `RunnerHandle`:
     - `status(&self) -> watch::Receiver<Option<RunnerSnapshot>>` is a clone of the receiver. Without a runtime, the host uses `has_changed()` and `borrow_and_update()`, which are in-memory and do no I/O. A runtime is only needed for `changed().await`.
     - `stop(&self)` is idempotent and sends `true`.
     - `join(self) -> Result<()>` joins the thread. A panic becomes an error.
     - `Drop` calls `stop()` and then joins, so a host that exits without calling `stop` still gets the final capture.
   - The lease, and with it `runner.lock`, is dropped when `run` returns. Retry after release therefore works (criterion 7).
   - A detached sync thread may outlive `join()` for as long as the Git kill timeout (30 s, TASK00001 step 1). `network.lock` and the operation lock serialize it against a restarted runner.
6. **Tests** (new `tests/runner.rs`, library-level, using the CLI binary via `env!("CARGO_BIN_EXE_dispatch-mesh")` for the separate-process parts):
   - **Criterion 3:**
     - `start` returns within 1 s.
     - The test function is a plain `#[test]` with no tokio runtime, which proves the caller needs none.
     - A first snapshot arrives within 2 s.
   - **Criterion 4:**
     - Write a file, wait for the snapshot to show it captured, then write a second file and `stop()` before it stabilizes. After `join()`, `Workspace::open` shows both files in the active session.
     - **Stop during in-flight sync:** reuse the stalled loopback remote from `slow_remote_does_not_block_capture_or_overwrite_concurrent_saved_state` (`tests/cli.rs:254`). Once the server sees the request, the stalled request is in flight. `stop()` + `join()` must finish in under 5 s and still contain a file saved during the stall.
   - **Criterion 5:**
     - Wait for a snapshot, then let 3 s pass with no edits and assert `has_changed()` is false.
     - Run CLI `session finish` in a separate process and assert that a snapshot with that session `closed` arrives within 2 s.
     - Run CLI `intent begin`, then `intent end`, and CLI `status`, and assert each exits 0 while the handle runs (criterion 8).
   - **Criterion 6:**
     - Use the `serve_response` mock from `tests/provider.rs` returning 500 with a body containing a marker string and a `--api-key-env` key with a known value.
     - Finish a session and assert:
       - the snapshot's `status` carries the summary failure;
       - CLI `status` shows it;
       - neither contains the marker or the key.
     - Switch the mock to a valid response, finish a second session, and assert the failure is cleared.
     - Snapshot `sync_error` and `capture_error` are asserted with the unreachable-remote setup from TASK00001 step 7 and a held operation lock, where a busy lock must *not* appear as an error.
   - **Criterion 7:**
     - Start a handle, then call `Runner::start` on the same root and check `downcast_ref::<AlreadyRunning>()`.
     - CLI `run` against it exits non-zero with "already running".
     - After `stop`/`join`, `start` succeeds.
     - Reversed: a spawned CLI `run` makes `Runner::start` fail with `AlreadyRunning`.
   - **CLI shutdown busy-lock regression (step 3):** hold `Workspace::open` in the test while signalling the runner to stop, release the lock after 200 ms, and assert the capture happened. This is done through the handle, because Ctrl+C cannot be sent portably to a child process. The handle and the CLI share the code path.
   - **Existing tests:** `tests/cli.rs` runner tests stay unchanged and must pass. They prove criterion 2's "output unchanged".
   - **Timing:** tests use 2 s deadlines with 50 ms polling of `has_changed()`, not fixed sleeps, except the deliberate 3 s quiet window. Engine tests are slow on Windows, so the deadlines leave margin for CI.
7. **Docs.**
   - `README.md`: one paragraph, under the `status` section, saying that hosts embed the engine through `dispatch_mesh::runner::Runner`, plus the already-running message. The API is not yet a stable public crate surface; say so.
   - `AGENTS.md:11`: move "watcher, capture loop, and synchronization scheduling" from `src/main.rs` to a new `src/runner.rs` bullet.
8. **Gates:** `cargo fmt --check && cargo clippy --all-targets -- -D warnings && cargo test`.

### External changes and migration

- No change to the remote format.
- No local state change beyond TASK00001's `summary_failure`.
- No new dependency: `tokio::sync::watch` and the runtime builder are covered by the existing `rt-multi-thread` feature; `sync` is a default tokio module.
- CLI output is unchanged. TASK00001's new status values are that ticket's change, not this one's.

### Cross-ticket effects

- **[[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] step 4:** please build `status_snapshot()` as "scan + `status_from(LocalObservation)`", with `Status: Clone + PartialEq`. That is a small shape change, not new scope. Record it there when its plan is next touched.
- **[[Story - US00001 - Tray icon and status window host|US00001]]:**
  - The handle exposes a `watch::Receiver`. The decided API has no wakeup callback, so a tray event loop checks `has_changed()` on its own timer (an in-memory check, not a DB read) or wraps `changed()` in a small runtime. If the toolkit wants an event-loop proxy push, add `RunnerHandle::on_change(callback)` in US00001. It is additive.
  - The already-running text Kai specified is the host's to show. The library error stays CLI-worded.

### Wiki pages this makes wrong

- [[ADR-0009 — Embed the engine with a runner handle and published status]]: the `acquire`/`run(lease, …)` split, `Option<RunnerSnapshot>` and stop-on-`Drop`. Amend it and set it to `accepted` at `Done` (criterion 1).
- `AGENTS.md:11` says `src/main.rs` holds the "watcher, capture loop, and synchronization scheduling". Step 7 points that line at `src/runner.rs` in the same change.
- `README.md`: step 7 adds the host paragraph. Line 69 ("The run loop can capture while a summary request is pending") stays true.
- [[Dispatch Mesh requirements and architecture]]: it doesn't say where the loop lives, so no change is needed.

### Risks and rollback

- **Timing-sensitive tests on Windows CI.** Mitigations: deadlines instead of sleeps, and a 2 s bound that is twice the tick. If the quiet-window test flakes because of the 30-tick periodic rescan, that rescan does not change state unless files changed, so a flake points at a real spurious publication. Investigate it rather than widening the window.
- **Detached sync after `join()`.** The host process may keep a Git child for up to 30 s after Quit. That is acceptable for the tray, which exits anyway, and it is serialized by the locks. If it turns out to matter, a cancellable Git child is TASK00002's territory.
- **Library prints to stdio.** That is harmless in a GUI subsystem. Moving all console text behind a reporter is deferred until a host needs it.
- **Rollback:** revert the commit. No state or format change belongs to this ticket.

### Size

Roughly 2–3 focused days after TASK00001 steps 2–4 land: the move and handle about 40%, tests about 50% (mostly timing harness), docs 10%. This is an order-of-magnitude estimate, not a commitment.

### ADRs

- **Governing:** [[ADR-0009 — Embed the engine with a runner handle and published status]] (this design; amend at `Done`), [[ADR-0010 — Report workspace status and session stages separately]] (the `Status` carried), [[ADR-0007 — Persist state transactionally with recovery journals]] (lock-free fallback snapshot; captured work survives a failed shutdown), [[ADR-0005 — Make AI summaries configurable and opt-in]] (summary failure never blocks publication).
- **No new ADR candidates.** The API refinements in step 2, step 4 and step 5 elaborate ADR-0009 and belong in its amendment, not in a new record.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

Manual checks for the in-process runner and the CLI `run` rewiring on `us00002-engine-control-interface` at `0e4c9a6` (reviewed build). Build it with `cargo build` and use `target\debug\dispatch-mesh.exe` (Windows) or `target/debug/dispatch-mesh`. The automated suites already cover:

- the handle API: start without a runtime, quiet and changed snapshots, CLI changes within 2 s, stop with and without a held lock or a stalled remote;
- published capture, sync and summary errors, and their recovery;
- secrets: no response body or API key in any output;
- refusal of a second runner in every combination.

These checks are not repeated below.

- [ ] **Kai:** *Ctrl+C in a real terminal still saves the last edit.*
  1. In PowerShell, create a scratch workspace with `dispatch-mesh.exe --workspace C:\MeshManual init --name Kai --email kai@example.test`, then start `dispatch-mesh.exe --workspace C:\MeshManual run`.
  2. In an editor, create `C:\MeshManual\note.md` containing `hello`, save it, and **within one second** press Ctrl+C in the `run` window.

  Expected:
  - The window prints `Local changes saved. Mesh stopped.` and returns to the prompt.
  - `$LASTEXITCODE` is `0`.
  - `dispatch-mesh.exe --workspace C:\MeshManual status` lists a session whose `files` include `note.md`.

  Automation cannot send Ctrl+C to a console process on Windows; the tests drive the same shutdown path through `RunnerHandle::stop()`.
- [ ] **Kai:** *Ctrl+C during a real network outage does not hang.*
  1. Use a workspace connected to a real GitHub test repository (`init --remote https://github.com/<you>/<test-repo>.git`), with working credentials.
  2. Start `run`, edit and save a file, and wait until `status` shows that session at `draft_uploaded`.
  3. Disconnect the network (Wi-Fi off or cable out). Edit and save a second file, wait about 5 s, then press Ctrl+C.

  Expected:
  - `run` stops within about 5 s with `Local changes saved. Mesh stopped.`
  - No credential prompt appears.
  - After reconnecting, `status` lists the second file in a session, and starting `run` again uploads it (the session returns to `draft_uploaded`).

  Automation covers this against a loopback mock, not real Git with the Windows credential helper and a real network drop.
- [ ] **Kai:** *Linux CI is green for this branch.*
  1. Push `us00002-engine-control-interface`.
  2. Open the `ci` workflow run for it on GitHub.

  Expected: both the `windows-latest` and `ubuntu-latest` jobs pass, including `tests/runner.rs` (10 tests).

  On Linux the runner lock is a `flock` per open file description, so it is the only place where refusing a second same-process `Runner::start` is verified. This machine has neither WSL nor Docker, and the branch has not been pushed yet.

Deliberately omitted:
- A GUI-subsystem host (no console for stdio) does not exist until [[Story - US00001 - Tray icon and status window host|US00001]]; its tests cover that.
- A real AI provider: the provider failure path and credential suppression are asserted against loopback mocks with a child-process sentinel key, and real-provider checks belong to [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]].

## As-built notes

### 2026-11-05 — Implemented on `us00002-engine-control-interface` (`f2b3e1c`)

Built as planned: `src/runner.rs` (`acquire`, `run`, `summarize_ready`, `Runner`, `RunnerHandle`, `RunnerSnapshot`, `AlreadyRunning`), `Workspace::status_from(LocalObservation)` in `src/status.rs`, and a thin `src/main.rs`. Tests are in `tests/runner.rs` (6 tests). Gates are green: `cargo fmt --check`, clippy `-D warnings`, `cargo test` (75 tests). The existing `tests/cli.rs` runner tests pass unchanged.

Deviations from the plan:

- **Tokio `sync` feature added** to `Cargo.toml`. The plan said `tokio::sync` was available by default, but tokio has no default features. This adds no new crate.
- **Step 1 was done here.** TASK00001 merged without the scan-free split, so it was applied as a pure refactor: `status_snapshot()` = scan + `status_from`.
- **The criterion 6 runner test asserts response-body absence with a marker string, not an API-key value.** Setting a process environment variable in parallel tests is unsafe and racy. The CLI-level credential and body checks remain in `tests/provider.rs` (TASK00001).
- **A mutation check of the shutdown retry was done by hand, not committed.** With `SHUTDOWN_LOCK_WAIT = 0`, `shutdown_capture_rides_out_a_briefly_held_operation_lock` fails with `WorkspaceBusy`; with the 10 s wait, it passes.
- **Criterion 1 is still open by design.** ADR-0009 is to be amended (the `acquire`/`run(lease, …)` split, `Option<RunnerSnapshot>`, stop-on-`Drop`) and set to `accepted` when this ticket reaches `Done`.

> ⚠️ Correction 2026-11-05 (review R3): the bullet above on the criterion 6 runner test overstated the evidence. `tests/provider.rs` configures `api_key_env: None`, and its secret-looking strings are response-body markers. Its missing-variable test checks a variable *name*, not that an actual key value is suppressed, so nothing covered suppression of a real key until `0e4c9a6` (below).

### 2026-11-05 — Review fixes (`0e4c9a6`)

Fixes for the 2026-11-05 review of `f2b3e1c`. Only test code changed; `src/` is untouched.

- **R1:** the `mesh_ok` helper in `tests/runner.rs` retries the documented `WorkspaceBusy` ("retry shortly") outcome for up to 15 s, with any other CLI failure still failing at once. The runner can hold the operation lock past the engine's 500 ms settle time while finalizing, which is the existing CLI contract and not a product change. The summary-recovery assertions are unchanged.
- **R2:** three new tests cover the host error fields and busy-lock filtering:
  - `capture_errors_are_published_and_clear_once_the_folder_is_readable`: an oversized file under a 16-byte `max_file_bytes` produces a capture error that clears once the file shrinks.
  - `sync_errors_are_published_with_durable_state_and_clear_after_recovery`: a missing bare remote produces a sync error alongside a durable attention or offline state, and both clear once the remote is created.
  - `a_held_operation_lock_still_publishes_and_is_not_an_error`: with the lock held across ticks, no busy error is published, and a committed change made under the held lock reaches the snapshot through the lock-free fallback within 2 s.

  Each test also asserts that the other error fields stay clear. Mutation checks were done by hand, not committed: publishing `sync_error: None` fails the sync test, and dropping the busy filter in `report_failure` fails the held-lock test.
- **R3:** `a_failing_provider_never_exposes_the_api_key_in_status` sets a sentinel key only in a CLI `run` child's environment, so this test process's environment is never touched. It checks that the mock received the key, and that neither `status` nor the runner's stderr contains it.
- **Remaining deviations, recorded per R3:**
  - **File-capture waits are 10 s, not the plan's general 2 s.** Capture needs two identical scans and runs on a Windows file watcher. The 2 s bound is kept where criterion 5 sets it: for a change made by a separate CLI process.
  - **`Runner::start` builds the Tokio runtime on the calling thread** and moves it into the engine thread, instead of building it inside that thread. A failure to build the runtime is then returned synchronously from `start`, like the lock refusal. Building a current-thread runtime is cheap and does not block on the loop.
  - **Shutdown during a stalled AI request has no separate test.** The summary task is never awaited on shutdown and is dropped with the runtime. The stalled-network test covers the analogous Git path.
- **Gates:** `cargo fmt --check` and `cargo clippy --all-targets -- -D warnings` pass. Full `cargo test` passed twice in a row (79 tests), and the runner suite passed three more times on its own (10/10 each). `open_findings` is cleared to empty until the next review.

### 2026-11-05 — Code-review hand-off blocked at `f2b3e1c`

Findings R1 and R2 below leave criterion 8 incomplete: `cargo test` fails in the new summary-recovery test under runner lock contention, and the new runner error fields and busy-lock filtering have no automated assertions. Keep this ticket `In progress` and unfrozen; do not run `/test-plan` until a fresh `/develop US00002` session repairs the failing test, adds the missing coverage and a fresh review clears it. `open_tests` is unchanged. No implementation code was changed in this review.

## Code review

### 2026-11-05 — branch us00002-engine-control-interface at f2b3e1c

**Scope and independence.** Reviewed `main...HEAD` (`41a543b...f2b3e1c`): one US00002 commit, eight changed files, including both new files in full. This session authored none of the implementation. Read the complete ticket and ADR-0005, ADR-0007, ADR-0009 and ADR-0010. No open PR exists for this branch; no PR comment was posted.

**Gates.** Independently ran `cargo fmt --check` and `cargo clippy --all-targets -- -D warnings`: both passed. **Full `cargo test` failed**: `summary_failures_reach_the_snapshot_without_the_response_body` panicked at `tests/runner.rs:43` when CLI `session finish` returned `Mesh: Another Mesh operation is using this workspace; retry shortly`. Library (5), CLI (9), engine (27) and provider (20) tests passed; runner tests finished 5 passed / 1 failed. A focused rerun of the failing test passed (1/1), but rerunning the runner suite reproduced the same failure (5/6 passed), establishing timing dependence rather than a consistently green gate. Storage/transport was skipped after the failure; a separate `cargo test --test storage_transport` passed all 8 tests. Doc-tests were not reached by the failed full run. `git diff --check main...HEAD` passed. Verification is local Windows only, using temporary workspaces and local mock services; no live GitHub or AI-provider test was run.

**Verdict: changes required — 2 blocking findings, 1 non-blocking finding.** The ticket cannot leave review or hand off to `/test-plan` as it stands. The blockers are a reproduced mechanical-gate failure and an explicit acceptance-test obligation the suite does not exercise; no additional functional implementation defect was established.

**Acceptance criteria, one by one.**

1. ADR-0009 records the agreed model, handle methods, publication, failure reporting and refusal behavior. Its amendment and acceptance at `Done` remain the expressly scheduled completion action, not an unrecorded pivot. Include the final async signature, optional initial snapshot, stop-on-Drop and named-pipe/Unix-socket extension point in that amendment.
2. Satisfied: loop and summarization are in `src/runner.rs`, status computation in `src/status.rs`, and CLI commands delegate to them. The old transition messages and startup/shutdown text are preserved; the existing CLI tests are unchanged.
3. Satisfied by inspection and the plain synchronous runner test: the handle runs the engine on its own thread/runtime and startup is bounded in the test.
4. Satisfied by inspection and tests for immediate saved edits, a held operation lock and a stalled network request. Shutdown does not join the network or summary jobs; `shutdown_background()` avoids runtime teardown waiting for blocking Git work. A stalled-AI-request shutdown is not separately tested.
5. Publication uses `send_if_modified` and reloaded durable state, with lock-free fallback. The tests assert an initial snapshot, a quiet interval and a CLI finish update within 2 s. File-capture waits use 10 s rather than the plan's general 2 s test deadline; the CLI-change bound itself is retained.
6. Summary failure persistence, sanitized response-body handling and subsequent clearing are exercised through a running handle, but the recovery test is timing-sensitive (R1); CLI persistence/recovery is covered separately in `tests/provider.rs:718`. The three runner error fields are wired by inspection but their publication/filtering/recovery is unasserted (R2). Actual injected API-key-value suppression is not covered by the cited provider tests (R3).
7. Satisfied: typed refusal is tested handle-to-handle, handle-to-CLI and CLI-to-handle, plus restart after release.
8. Failed: the full test gate is red (R1), and tests omit the explicitly planned snapshot-error assertions under criterion 6 (R2).

**Findings, in severity order.**

1. **R1 — Blocking, P2: make the summary-recovery scenario survive normal runner contention.** Location: `tests/runner.rs:326` (CLI invocation) and `tests/runner.rs:43` (unconditional one-attempt success assertion). After the first failed provider request, the test writes the second file, sees it captured, then issues `session finish` while the runner can be finalizing/synchronizing. The operation lock can remain occupied beyond the engine's 500 ms settle retry, and the CLI returns the documented busy error. This failed both the full suite and a subsequent runner-suite rerun; the isolated test passed. Consequently criterion 8's green mechanical gate is not met. **Fix:** make this test's CLI interaction handle the explicitly retryable busy outcome with a bounded retry (and fail immediately on any other error), or arrange deterministic lock availability without bypassing the separate CLI process. Preserve the assertions that a later successful summary clears the persisted failure. Run the runner suite and full gates after repair; one isolated pass is insufficient evidence. This is an observed test/coordination failure, not evidence of lost edits.
2. **R2 — Blocking, P2: exercise the host error fields and busy-lock filtering.** Location: `tests/runner.rs:252`–`331`, especially the snapshot assertions at lines 308–315; production wiring is `src/runner.rs:290`–`292`. None of the six runner tests asserts `RunnerSnapshot.capture_error`, `sync_error` or `summary_error`. The stalled-sync test stops before the remote failure is consumed, and the held-lock test checks only final capture. Thus publishing `None` in all three fields, or exposing `WorkspaceBusy` as a capture error, would escape the new suite. Criterion 8 requires coverage of criterion 6, and plan step 6 explicitly promises sync/capture-error and busy-lock assertions. **Fix:** add host-level regression cases that (a) cause a real scan failure, for example exceeding a small configured file-size limit, assert capture error/attention publication, then restore the file and assert clearing; (b) fail a loopback remote, assert `sync_error` and durable offline/attention agree, then restore synchronization and assert clearing; and (c) hold the operation lock across a tick, assert the fallback snapshot remains usable and no busy error is published. Check the other error fields remain appropriately clear in these cases. These are coverage failures, not claims that the current wiring is broken.
3. **R3 — Non-blocking, P3: correct the claimed credential-test evidence and record remaining plan deviations.** Location: this ticket's implemented as-built entry, the bullet beginning “The criterion 6 runner test”; supporting code is `tests/runner.rs:293` and `tests/provider.rs:740`. Both provider configurations use `api_key_env: None`; the secret-looking strings are response-body markers, not injected credential values. The missing-variable test verifies a variable name, not suppression of an actual key value. The note therefore overstates which part of the original plan is covered. The as-built entry also does not record the omitted error-field assertions, the 10 s file-capture waits, or building the runtime on the caller thread rather than inside the engine thread (`src/runner.rs:327`). **Fix:** append a dated correction distinguishing actual response-body coverage from credential-value coverage, record the remaining deviations and their rationale, and either add the intended credential sentinel test safely (for example, in an isolated child process) or accurately state that it was omitted. No acceptance-contract rewrite is needed to correct the record.

**What should stay.** Keep the single shared run loop and typed synchronous lock refusal: they avoid divergent CLI/host behavior and accidental duplicate engines. Keep the scan-free `status_from` split and lock-free fallback: publication reuses existing capture work and can observe committed state under contention. Keep the bounded shutdown retry and `shutdown_background()` together: each addresses a different way shutdown could miss saved edits or wait for remote work. The stalled-network and cross-process finish tests verify useful observable behavior rather than implementation details.

**Repair plan / before this leaves review.**

- A fresh `/develop US00002` session repairs R1's failing scenario, adds R2's regression coverage, appends R3's as-built correction and reruns the mechanical gates. Do not fix implementation in this review session.
- Request a fresh `/code-review US00002` against the resulting commit; only that review recounts `open_findings`.
- No `/update-ticket` contract change or product decision is required for these findings. Preserve the scheduled ADR-0009 amendment/acceptance at `Done`.
- `/test-plan` remains blocked until `open_findings: 0`; it will stamp the freeze against the reviewed implementation. Leave status, `frozen`, `completed` and `open_tests` unchanged here.

### 2026-11-05 — branch us00002-engine-control-interface at 0e4c9a6

**Verdict: review clear — 0 blocking and 0 non-blocking findings.** R1, R2 and R3 from the review of `f2b3e1c` are resolved. The old review and blocker entry above describe that earlier build; this entry and `open_findings: 0` describe `0e4c9a6`. The review gate for `/test-plan` is clear; this review does not move the card or complete the ticket.

**Scope and independence.** Rechecked the full branch scope `main...0e4c9a6` against the ticket, plan and governing ADRs, with detailed review of the complete fix diff `f2b3e1c..0e4c9a6`. The fix commit changes only `tests/runner.rs` (228 additions, 37 deletions); production code is unchanged from the independently reviewed implementation. This session authored neither implementation nor fixes. No open PR exists on the branch; no PR comment was posted.

**Independent verification.** `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and full `cargo test` all passed locally on Windows: 79 tests (5 library, 9 CLI, 27 engine, 20 provider, 10 runner, 8 storage/transport), with doc-tests completing successfully (0 tests). A separate `cargo test --test runner` also passed all 10 tests, so the previously failing scenario passed both separately and in the full run. `git diff --check main...HEAD` passed. These checks used disposable local workspaces, bare remotes and loopback mock services; no live GitHub or AI-provider request or separate-platform verification was performed. The author's repeated-run and mutation-check claims remain author evidence, not checks repeated by this reviewer.

**Resolution of the previous findings.**

1. **R1 resolved.** `mesh_ok` retries only a failed command containing the documented `WorkspaceBusy` message, for a bounded 15 s; other failures still fail immediately. The summary-failure/recovery assertions and separate CLI process are retained. The runner suite and full gate both passed independently, including the formerly timing-sensitive case.
2. **R2 resolved.** The oversized-file test checks published capture failure, matching workspace attention and subsequent clearing. The missing-remote/recovery test checks published sync failure together with durable attention/offline and their clearing after creating the remote. The held-lock test observes multiple busy ticks without capture/sync errors, then verifies a committed marker reaches the snapshot within 2 s while the lock remains held. Other error fields are checked as clear in the failure/recovery cases. These exercise the exact missing publication and busy-filtering scenarios requested by the prior review.
3. **R3 resolved.** A dated correction explicitly retracts the original credential-coverage claim and records the remaining deviations. The new sentinel test sets the key only in a CLI runner child's environment, verifies the mock received it, and checks both CLI status and runner stderr for absence of that key. No shared test-process environment is modified. Existing snapshot response-body checks remain intact.

**Acceptance criteria and plan.** Criteria 2–7 retain the previously reviewed implementation and now have the requested additional regression evidence; criterion 8's coverage and mechanical gate are satisfied. Criterion 1 retains its explicitly scheduled ADR-0009 amendment/acceptance at `Done`: record the final async signature, optional initial snapshot, stop-on-Drop and deferred named-pipe/Unix-socket extension there. The runtime-construction location, 10 s capture waits and lack of a separate stalled-AI shutdown test are now recorded deviations; they introduce no new product decision or scope. No contract-zone rewrite or `/update-ticket` is needed. Both commits belong to US00002.

**What should stay.** Keep busy-only bounded retries rather than retrying arbitrary CLI failures. Keep both the error-to-recovery assertions and the held-lock marker test: the former verifies host-visible outcomes, while the latter proves the lock-free fallback actually publishes fresh committed data. Keep credential injection scoped to the child process so the test cannot race with other provider tests.

**Before this leaves review.** `/test-plan US00002` is next and should stamp the freeze against reviewed commit `0e4c9a6`. Retain the ADR-0009 amendment/acceptance at completion. Status remains `In progress`; `open_tests`, `frozen` and `completed` are unchanged. No implementation was edited in this re-review.

## Follow-ups

### 2026-11-05 — Architecture decisions recorded

Kai requested ADR extraction. [[ADR-0009 — Embed the engine with a runner handle and published status]] records the answers in this ticket as an agreed, not-yet-built design (`proposed` under the ADR template). [[ADR-0010 — Report workspace status and session stages separately]] governs the status model the runner transports. Recording these decisions does not complete implementation, review or testing and does not change ticket gates.

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[Dispatch Mesh milestone plan]] — 0.2.x engine stabilization; 0.3 tray application; Decisions to close during delivery
- [[Story - US00001 - Tray icon and status window host|US00001]] — consumer of this interface
- [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] — status model this interface transports
- Code: `src/main.rs` (`run`, `.mesh/runner.lock`, `show_status`), `src/engine.rs`, `src/lib.rs`; tests `tests/cli.rs`

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
