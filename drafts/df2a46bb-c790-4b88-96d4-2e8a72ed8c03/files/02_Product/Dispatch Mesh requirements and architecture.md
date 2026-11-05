---
copilot_chat: obsidian://copilot-chat?vault=Dispatch-Wiki&id=epoch%3A1793099431555
---

# Dispatch Mesh requirements and architecture

Status: Product discovery; requirements and implementation recommendation, not an implementation ticket.

Architecture decisions recorded on 2026-11-05: [[ADR-0001 — Synchronize drafts separately from permanent history]], [[ADR-0002 — Group files into sessions with an inactivity boundary]], [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]], [[ADR-0004 — Bind agent intent to attributable changes]], [[ADR-0005 — Make AI summaries configurable and opt-in]], [[ADR-0006 — Let Mesh own Git operations in managed folders]], [[ADR-0007 — Persist state transactionally with recovery journals]], [[ADR-0008 — Store content as blobs referenced by metadata]], [[ADR-0009 — Embed the engine with a runner handle and published status]], and [[ADR-0010 — Report workspace status and session stages separately]]. These govern the corresponding recommendations below. The last two are agreed designs awaiting implementation, not completed preview features.

Based on [[Dispatch Mesh concept]] and the subsequent discussion about desktop operation, autonomous agents, and cloud access through MCP.

## Product objective

Provide reliable Git-backed synchronization for Dispatch on Windows, macOS, and Linux. Collect related saved changes into editing sessions, produce meaningful history with AI-generated commit messages, and integrate changes automatically while preserving recoverable versions.

The first release targets Windows. The same synchronization engine should later support desktop clients on other platforms and a cloud-hosted Dispatch service that users can access without a local setup.

Delivery priority: complete the Windows, macOS, and Linux desktop product before the authenticated cloud MCP upgrade. The roadmap targets macOS beta in 0.7, Linux beta in 0.8, cross-platform desktop beta in 0.9, and a stable desktop release in 1.0. Cloud MCP previews and release belong to the later 2.0 cycle. Cloud requirements below describe that deferred scope; retain the engine boundaries needed for it without making cloud hosting a desktop prerequisite. See [[Dispatch Mesh milestone plan]].

## Confirmed requirements

### Desktop operation

- Support Windows, macOS, and Linux, starting with Windows.
- Support users unfamiliar with Git through guided setup; do not require Git CLI commands or manual repository configuration.
- On Windows and macOS, run only while the relevant user is logged in. Synchronization while logged out is not required.
- Watch a configured folder for saved file changes and collect related changes into editing sessions.
- Avoid creating a permanent commit for every individual file change.
- Synchronize saved drafts continuously while a session remains open. A session can cover related changes across multiple files and closes after five minutes without local edits; switching files does not close it.
- After an editing pause, send the accumulated diff to a remote AI API to generate a meaningful commit message.
- Keep AI integration provider-independent and easy to reconfigure, including support for locally hosted models instead of a remote service.
- Automatically merge compatible text changes; preserve unresolved competing versions for review while unrelated files continue syncing.
- Focus on text files while also synchronizing and committing binary files. Resolve binary conflicts only by choosing one complete conflicting version; do not merge binary contents automatically or through AI.
- Provide editing presence so collaborators can notice activity in the same file without locking it.
- Provide a tray icon opening a status window with an unresolved-conflicts list and access to conflict details and resolution actions.

“Only while logged in” is interpreted as allowing quiet background operation during the login session, including while the screen is locked. The desktop process should stop at logout or explicit quit. This interpretation, and the Linux desktop lifecycle, remain to be confirmed.

### Cloud operation

- Run independently of an interactive OS login.
- Support an existing role/service account for an autonomous agent performing recurring work, housekeeping, and support tasks.
- Support an associated GitHub role account for that agent.
- Expose Dispatch capabilities through a remote MCP server so users can work without installing Dispatch or cloning a repository locally.
- Support multiple connecting users and select the appropriate authorized GitHub account for each user's operations.
- Preserve the initiating identity for queued work that continues after an MCP connection closes.

## Recommended technology

Use **Rust with Tokio** for the shared synchronization engine. This is a recommendation, not a confirmed product requirement.

The workload consists of long-running filesystem observation, timers, subprocesses, network requests, and durable coordination. Rust is a good fit for explicit lifecycle control and a modest resource footprint. Its main cost is implementation complexity and the learning curve, particularly around asynchronous code.

| Component | Recommendation | Responsibility |
|---|---|---|
| Shared engine | Rust | Session management, snapshots, reconciliation, integration, conflict policies |
| Async runtime | Tokio | Timers, task coordination, network I/O, subprocess management |
| File watching | `notify` plus reconciliation scans | Detect changes and recover from missed events |
| Durable local state | SQLite | Session metadata, pending operations, recovery records |
| Git operations | Packaged Git executable | Diff, fetch, merge, commit, push |
| AI API client | `reqwest` and `serde` | HTTP requests and structured responses |
| Diagnostics | `tracing` | Structured operational logs |

Keep Git integration behind an interface so packaging or implementation choices can change. Initially, prefer the Git executable for its standard behavior and support for operations such as `git merge-tree` without modifying the live working tree.

### Configurable AI providers

Users must be able to switch AI providers or use a locally hosted model without changing the synchronization engine. References to a remote AI API describe one deployment option, not a requirement to transmit content to a cloud service.

Use an internal provider interface that accepts a frozen diff, recorded intent and provenance, and snapshot identifiers, and returns a validated commit title and description tied to that snapshot. Keep wire protocols and provider-specific parameters in adapters.

The recommended first adapter is OpenAI-compatible Chat Completions with a configurable base URL, model, and credentials where required. This provides a practical starting point for hosted services and compatible local servers. OpenAI Responses and Anthropic Messages are other protocols that can be supported through separate adapters; they are not required for the initial adapter.

Configure supported capabilities rather than assuming every compatible endpoint implements the same parameters, context limits, or structured output features. Prefer schema-constrained responses where supported; otherwise request JSON and validate it locally. Use bounded retries and a factual fallback message when summarization fails. Synchronization and local capture must continue independently of model availability.

Provide guided provider settings and a connection test in the desktop UI. Store API credentials securely. A local-only configuration must not silently fall back to a remote provider. Binary changes can be described using file-level change metadata; binary payloads are not needed for commit-message generation.

### Agent intent capture

Use the Mesh CLI as the local integration point for agents to attach the user's requested purpose to their changes. The console preview implements `intent begin --purpose ... --paths ...` and `intent end <id>`; see [[ADR-0004 — Bind agent intent to attributable changes]] and the versioned README for command details. Remote agents should eventually use equivalent authenticated MCP operations through the same application layer.

Intent records should contain the relevant requested purpose, its provenance, the initiating identity, and the agent/task context. Bind them to the snapshots or changes they explain and retain that association through draft synchronization, integration, and finalization. Do not assign one agent's intent to unrelated direct edits or another agent's work merely because they share a file or editing session. The precise mechanism for attributing changes made by concurrent local writers remains a technical design task; a filesystem watcher alone cannot establish attribution.

Summarization must distinguish the user's requested purpose from the changes actually achieved. Summarize relevant intent rather than storing entire conversations, apply privacy and secret filtering, and allow ordinary direct edits without any intent record. Explicitly finishing a session remains optional and separate from recording intent.

C# with modern .NET is a credible alternative if the implementing team is significantly more experienced with C#. The correctness of synchronization and recovery matters more than the language choice.

## Recommended architecture

Keep the synchronization engine independent of desktop UI, interactive login, and platform startup mechanisms.

| Layer | Desktop host | Cloud host |
|---|---|---|
| Lifecycle | User login session | Independently supervised process or container |
| Interface | Tray/menu-bar controls and status | MCP tools, administrative CLI, health/status endpoint |
| Identity | Configured desktop user | Per-request user or explicitly configured autonomous agent |
| Credentials | User credential provider | Securely stored, explicitly selected credentials |
| Storage | Local persistent state | Persistent workspace storage and durable job state |
| Recovery | Reconcile on startup and resume | Recover after restart and retry interrupted jobs |

### Desktop hosting

For Windows, start with a normal application launched at login. A Windows Service is unnecessary for the agreed desktop lifecycle. If Task Scheduler is used, select its interactive-token mode, which requires an existing logged-in session.

For macOS, use a login item or LaunchAgent. Add Linux desktop startup integration separately once its lifecycle requirements are settled.

A small desktop interface should expose saved locally, waiting to sync, synced, and needs attention, plus pause, resume, quit, and recovery actions. Closing a status window should have clearly defined behavior distinct from quitting synchronization.

The tray icon should indicate when conflicts need attention. Clicking it opens the status window, where users can browse unresolved conflicts and inspect the competing versions, authors, and available resolution actions. This is a Mesh interface backed by synchronized conflict records; GitHub does not automatically provide that interface.

Offer an optional "Finish session" action in the tray menu and status window. Users do not need to invoke it: five-minute inactivity closes sessions automatically. Finishing freezes the current session for finalization; further edits enter a new session without waiting for AI summarization or publication.

### Guided workspace setup

Target users include people unfamiliar with Git. Provide setup through the desktop interface using familiar terms such as workspace, local folder, and shared history. Mesh should perform repository operations and credential configuration on the user's behalf, with actionable status and error messages.

Support these setup paths:

- Create a new workspace, select or create its local folder, and create and connect the backing repository through a guided flow.
- Join an existing workspace by selecting an accessible repository and a local destination; Mesh clones it and starts synchronization.
- Connect an existing local folder, whether it already contains a Git repository or needs initialization. Guide remote selection or creation and preserve existing files and history. If local and remote content already exist, reconcile them without silently overwriting either side.

Guide account sign-in and repository access through the UI. Do not require users to run Git commands, manually install Git, edit Git configuration, or construct credential-bearing remote URLs. Technical details can remain available for troubleshooting without being prerequisites for setup.

Repository creation must clearly present the destination account or organization and visibility before creation. Exact screens, default visibility, handling of existing repository configurations, and whether workspace invitations supplement repository selection remain to be designed.

### Editing presence

Editing presence helps people and agents notice concurrent activity before it becomes a conflict. It is advisory: collaborators can continue editing the same file, and synchronization and conflict preservation remain responsible for protecting their changes.

- Initially derive presence from saved local changes observed by Mesh. Show the collaborator, affected file, and last observed save, for example: "Alice recently changed this file. Last saved 12 seconds ago."
- A filesystem watcher cannot determine which file is open, detect unsaved typing, or identify the human behind an arbitrary write. Attribute activity to the configured session identity and distinguish users, agents, and devices where available.
- Keep presence per file, even when an editing session covers multiple files. Expire a file's recent-save presence after five minutes without further local edits to that file. Edits elsewhere in the session must not keep an old file marked active.
- Do not generate presence from applying incoming synchronized changes; that would incorrectly attribute remote edits to the receiving user and create activity loops.
- Exchange presence independently of permanent commits and AI summarization, so collaborators can see activity during an open session. Presence is temporary operational state, separate from durable conflict records and Git history.
- Display a non-blocking notice when another collaborator has recent activity in the same file. Make file-level presence accessible from the desktop status window opened through the tray icon; editor integrations can also display it beside the affected file.
- Optional editor integrations, such as an Obsidian integration, can report active editing more precisely. Label that signal separately from recent saves; do not claim live typing based on filesystem activity alone. An integration must refresh and expire its signal so stale activity does not persist after disconnects or crashes.
- Share presence only with authorized collaborators in the same workspace. When disconnected, let stale signals expire; absence of presence is not proof that nobody else is editing.

The initial feature uses recent-save presence. Exact presence transport, update cadence, and editor integration contracts remain implementation decisions.

### Cloud hosting and MCP

The cloud host should provide:

- An authenticated remote MCP endpoint exposing Dispatch operations.
- An application layer enforcing workspace access and coordinating changes with the synchronization engine.
- Isolated workspaces and durable background jobs.
- GitHub account linking and credential selection.
- A separate execution path for explicitly configured autonomous work.

MCP tools should expose product actions such as reading notes, editing content, managing tasks, and finishing an editing session. They should use the same application layer that controls snapshots and synchronization.

A connecting user should authenticate to Dispatch, link and authorize a GitHub account, and then access only the workspaces and repositories permitted to that identity. Account linking must not be inferred from a supplied username or email alone.

## Identity and authorization

Model these identities separately:

| Identity | Meaning |
|---|---|
| Execution account | OS/container identity running the process and owning files |
| Authenticated Dispatch user | Caller whose workspace permissions apply |
| GitHub authentication identity | User or application authorizing remote GitHub operations |
| Git author/committer | Metadata recorded in a commit |
| Autonomous agent identity | Explicit principal for scheduled or delegated agent work |

The cloud process can run under one service account while serving many GitHub identities. Changing commit metadata does not change remote authorization.

### Per-operation account selection

Never switch a global GitHub login, global Git configuration, or shared credential state when a user connects. Concurrent requests must select their own authorized credentials and commit metadata.

Each operation and durable job should carry:

- Authenticated principal and workspace identifier.
- Repository identifier and linked GitHub account reference.
- Whether the work is user-directed or autonomous.
- Author/committer policy and session/job identifier.
- References to required authorization, rather than embedding raw tokens in job records.

Recheck authorization when executing queued work. If credentials expire, access is revoked, or an account is unlinked, pause or fail with a recoverable status. Never silently fall back to a more privileged automation account.

### MCP and GitHub authentication

Use separate authorization flows and tokens for Dispatch MCP access and GitHub access. The MCP token must be intended for the Dispatch MCP resource; the server must not forward that incoming token to GitHub.

Prefer a **GitHub App with user authorization** for user-directed operations. GitHub App user access tokens constrain access by both the user's access and the app's permissions and installation scope. Each user must authorize the app.

For independent automation, a GitHub App installation token is an option when app-attributed activity is appropriate. If work must use the existing GitHub role account, explicitly provision authorization for that account instead. Keep autonomous authorization distinct from authorization for interactive users.

Store credentials securely, handle expiration and refresh, and supply credentials to individual Git subprocesses without persisting them in repository URLs or logs.

## Repository isolation and concurrency

### Initial managed-folder workflow

Mesh owns Git operations in a managed folder for the initial release. Supporting users who manually commit, switch branches, pull, merge, rebase, or otherwise mutate repository state alongside Mesh is outside the initial scope. Users need no Git workflow knowledge; existing repositories can still be connected through guided setup.

This is a support boundary, not a reason to overwrite unexpected changes. If Mesh detects external repository mutations that invalidate its recorded state, preserve captured work and pause affected synchronization with an actionable needs-attention status. Do not silently reset the repository or discard history. Advanced interoperability with manual Git workflows can be considered later.

Initially use isolated working directories and indexes for each user/agent context. Do not share one mutable checkout between concurrent identities.

- Coordinate operations that mutate the same workspace.
- Serialize local publication attempts for a shared branch and detect remote races from other clients.
- Preserve each operation's authorship and credential context.
- Recompute integration when the remote branch advances.
- Keep durable state and draft content across cloud restarts; an ephemeral container filesystem is insufficient.

SQLite is recommended for desktop and individual worker state. A horizontally scaled cloud control plane may need a shared database and queue; do not assume one local SQLite database coordinates multiple hosts.

## Session, synchronization, and recovery recommendations

These extend the design hypotheses in [[Dispatch Mesh concept]] and still require validation.

- Extend only the current unpublished editing session. Once published into shared history, treat its commit as permanent.
- Use a stable session ID independent of changing draft commit hashes.
- Combine filesystem notifications with startup, resume, error-recovery, and periodic scans.
- Debounce writes before capture and persist recoverable snapshots before network operations.
- Freeze a snapshot for merging and AI summarization while subsequent edits enter the next draft.
- Associate AI responses with a session ID and content hash so a delayed response cannot label a different diff.
- Continue local capture if GitHub or the AI endpoint is unavailable.
- Perform integration away from the live editing directory and check for newer edits before applying incoming content.
- Provide an explicit finish-session operation for users and autonomous agents.
- On desktop logout, prioritize persisting local state; do not depend on a final network synchronization completing.

The agreed inactivity boundary is five minutes without local edits across the session. Sessions can span multiple files; changing files does not end a session. Incoming synchronized changes and presence updates do not reset the local editing timer. The 1–2 second write-settling interval remains a hypothesis. A thirty-minute maximum session duration was previously proposed but is not confirmed.

Continuous draft synchronization is the selected product model. Keep draft revisions separate from permanent shared history; do not repeatedly amend and force-push the shared branch. The implemented transport uses `mesh/devices/<device-id>` refs and stable session IDs, with native content blobs and reference-based metadata, as recorded in [[ADR-0001 — Synchronize drafts separately from permanent history]] and [[ADR-0008 — Store content as blobs referenced by metadata]].

Keep recurring agent execution separate from synchronization. The autonomous agent performs housekeeping or support work; git-sync captures and integrates resulting repository changes. The agent can explicitly finish a session at the end of a logical task.

Expose the optional finish-session operation through the local CLI and authenticated remote MCP interface as well as the desktop controls. Proposed interfaces are `dispatch-mesh session finish --workspace <id>` and an MCP `finish_session` tool accepting a workspace and optionally a session ID. By default, scope the action to the caller's own session. Return finalization status or an operation ID, since summarization and publication are asynchronous. Agents may rely on the same inactivity timeout instead of explicitly finishing; exact command and tool schemas remain to be specified.

## Conflict handling recommendation

Automatic handling must preserve recoverability even when intent cannot be inferred.

1. Apply ordinary three-way merging where changes merge cleanly.
2. Use format-aware rules and validation for supported structured content.
3. Allow AI-proposed resolutions within a defined scope, preserving both inputs and validating where possible.
4. Preserve both versions and flag unresolved cases when meaning is ambiguous or validation fails. For binary conflicts, preserve both complete files and require choosing one of the conflicting versions; text merging, format-aware merging, and AI resolution do not apply to binary contents.

A clean textual merge is not proof of semantic correctness. For unresolved conflicts, keep the version already accepted into shared state visible while preserving the incoming alternative. This is a temporary display policy, not an automatic resolution; unrelated files continue syncing.

### Durable conflict records

Maintain one durable record per conflict and an index of unresolved records. Each record references the workspace and file path, common base and both competing snapshots, session identities and authors, timestamps, recorded intent where available, currently accepted version, and resolution status. Use stable conflict identifiers to avoid duplicates. Preserve exact binary snapshots rather than relying on a textual diff.

Synchronize records and their referenced content in Mesh-managed storage, separate from ordinary documents. They must survive restarts and remain recoverable after resolution; a local SQLite index alone is insufficient. The tray-accessible status window presents the index and details. Text conflicts can show diffs and offer selection or a written resolution; binary conflicts offer selection of one complete competing file. Cloud access should expose the same conflict list and resolution operations through authenticated MCP tools.

Record a resolution as a new traceable change, retaining the competing inputs. Before applying it, check for edits made since the conflict was recorded so newer work is not silently overwritten. The preview records contributing input revisions so selections survive publication order changes, using SQLite and reachable Git blobs; see ADR-0003, ADR-0007 and ADR-0008 above. Retention periods and concurrent conflicting resolution policy remain open.

## Delivery sequence

1. Build a Windows console executable with durable snapshots, session management, Git integration, and recovery.
2. Add login startup and desktop status controls around that engine.
3. Validate offline operation, restart recovery, concurrent edits, and conflict preservation.
4. Add macOS beta in 0.7 and Linux beta in 0.8 without changing the core session model, then validate cross-platform desktop operation in 0.9 and release the stable desktop product in 1.0.
5. In the 2.0 upgrade cycle, build the cloud host with persistent workspaces, identity isolation, and durable jobs.
6. In that same 2.0 cycle, expose Dispatch operations through authenticated MCP tools and per-user GitHub authorization, with private-alpha and beta gates before release.
7. Add recurring autonomous work under its explicitly configured identity.

Design the engine's identity and storage boundaries for cloud use from the start, while implementing the Windows experience first.

## Open decisions

- The current draft transport and content representation are settled by ADR-0001 and ADR-0008 above. Future extensions must preserve their invariants; Git CLI versus library backend is a separate open decision in [[Task - TASK00002 - Choose the Git backend before bundling Git]].
- Are session boundaries beyond five-minute inactivity and explicit finish needed, including a maximum duration?
- How should text/binary classification, file-size limits, and large-binary storage be configured?
- Which editor integrations should provide precise editing presence after the initial recent-save feature?
- Should Linux desktop execution follow the same login-session rule?
- Should desktop synchronization continue while the screen is locked, as currently assumed?
- Can one Dispatch user link several GitHub accounts, and how is the account selected for a workspace?
- Which autonomous operations must use the existing role account, and which may use a GitHub App identity?
- What author/committer attribution should represent user-directed AI edits and autonomous edits?
- What are the cloud retention, workspace-sharing, and deployment-scale requirements?
- Which Dispatch operations should the first MCP interface expose?

## References

- [[Dispatch Mesh concept]] — original discovery discussion, session model, and synchronization precedents.
- [Tokio tutorial](https://tokio.rs/tokio/tutorial) — asynchronous runtime.
- [notify documentation](https://docs.rs/notify/latest/notify/) — filesystem notifications, polling fallback, and platform limitations.
- [Git merge-tree documentation](https://git-scm.com/docs/git-merge-tree) — merge computation without modifying the working tree or index.
- [Microsoft Task Scheduler logon types](https://learn.microsoft.com/en-us/windows/win32/taskschd/principal-logontype) — interactive-session execution.
- [Apple Service Management](https://developer.apple.com/documentation/servicemanagement) — login items and launch agents.
- [MCP authorization specification, 2025-11-25](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-11-25/basic/authorization.mdx) — resource-bound authentication and separate upstream tokens.
- [GitHub App authentication on behalf of a user](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/authenticating-with-a-github-app-on-behalf-of-a-user) — per-user authorization and access boundaries.
- [GitHub App best practices](https://github.com/github/docs/blob/main/content/apps/creating-github-apps/about-creating-github-apps/best-practices-for-creating-a-github-app.md) — token selection and credential handling.
