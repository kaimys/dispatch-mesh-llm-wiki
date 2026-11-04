# Dispatch Mesh milestone plan

Date: 2026-11-04  
Status: Proposed delivery plan. Version numbers describe intended scope; they are not delivery-date commitments.

Related: [[git-sync requirements and architecture]], [[git-sync]].

## Objective

Move from the current console preview to a Windows application that people can set up and use without knowing Git, then expose the same workspace operations through an authenticated cloud MCP service.

The five requested capabilities are the main delivery milestones. Bundled Git precedes guided setup so that the setup flow can be tested on a computer with no Git installation.

| Version | Milestone                             | Result for the user                                                         | Dependencies                                                  |
| ------- | ------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------- |
| 0.2.x   | Stabilize the engine preview          | Reliable capture, synchronization, recovery, and understandable diagnostics | Current implementation                                        |
| 0.3     | Tray application                      | See activity, finish a session, and review conflicts from the desktop       | Engine stabilization                                          |
| 0.4     | Bundled Git and Windows packaging     | Install and run Mesh without installing Git separately                      | Tray host and a tested Git backend                            |
| 0.5     | Guided account and repository setup   | Create or join a workspace without Git commands                             | Tray UI, bundled Git, and credential integration              |
| 0.6     | Login startup and desktop beta        | Mesh starts with the user's login and recovers automatically                | Packaged application and guided setup                         |
| 0.7     | macOS beta                            | C                | Packaged application and guided setup                         |
| 0.8     | Authenticated cloud MCP private alpha | Use Dispatch workspaces through a remote assistant without a local clone    | Stable engine contracts, durable jobs, and identity isolation |
| 0.9     | Cloud beta and operational readiness  | Predictable hosted operation, recovery, and account lifecycle handling      | Cloud private-alpha acceptance                                |

## Product decisions carried into every milestone

- Synchronize saved drafts continuously. GitHub's `main` branch contains finalized sessions; it is not the live draft view.
- Keep published shared history immutable. Draft revisions have stable session identities and recoverable versions.
- Let a session cover related changes across multiple files. Close it after five minutes without local edits; switching files does not close it.
- Make **Finish session** optional. Offer it in the tray menu and status window, through the local CLI, and eventually through authenticated MCP tools.
- Preserve recorded user purpose with its associated changes, including when AI message generation is unavailable. Distinguish requested purpose from achieved changes.
- Focus on text while also committing binary files. Binary conflicts are resolved by choosing one complete competing version.
- Preserve unresolved conflicts and their inputs. Keep the accepted shared version visible while unrelated files continue syncing.
- Provide advisory, per-file recent-save presence. Editor integrations can later provide a more precise active-editing signal.
- Keep manual Git operations in managed folders outside the initial supported workflow.
- Keep the engine independent of the desktop host, startup mechanism, AI provider, and cloud authentication layer.

## 0.2.x — engine stabilization

Before building the desktop experience, close the gaps that could turn routine operation into confusing or unsafe behavior.

Work:

- Complete regression coverage for two-device collaboration, interrupted publication, restart recovery, deletions, renames, binary conflicts, and edits arriving during incoming changes.
- Keep file contents in native Git blobs and local binary storage, with metadata references and tested migrations.
- Make saved locally, draft uploaded, waiting for finalization, published, offline, and needs attention distinguishable in the application status model.
- Surface actionable AI failures. Distinguish missing credentials, bad endpoint settings, invalid response formats, temporary rate limits, and provider quota or access failures where the provider supplies that information.
- Retain useful commit messages and recorded intent during AI outages. Keep provider selection and text-sharing permissions explicit.
- Document the supported file/path limits, ignore-policy behavior, storage growth, and recovery procedures. Make unsupported cases visible without discarding work.
- Define the local engine/control interface that the tray host will use. Keep CLI intent and finish actions usable while Mesh runs.

Exit criteria:

- Automated engine and integration checks pass on supported test environments.
- Two clients receive drafts before a session closes, then converge on finalized history without attributing incoming edits to the receiving user.
- Restart and network-failure tests preserve captured work and conflict inputs.
- A user can distinguish a draft upload from publication on `main` and can find the cause of a failed operation.

## 0.3 — tray application

Deliver the first Windows desktop interface around the existing engine.

Work:

- Add a tray icon with meaningful state changes and a status window. Closing the window leaves Mesh running; **Quit** stops the host.
- Show each workspace's capture, draft-sync, finalization, publication, and attention state.
- Add pause/resume, optional **Finish session**, and access to recovery information. Define and label exactly what pause suspends before implementing it.
- List unresolved conflicts. Show the path, competing authors and versions, and the currently displayed version. Support text comparison and version selection; binary resolution offers whole-file selection.
- Show recent-save presence per file, including the collaborator and last observed save. Expire stale signals and distinguish them from actual editor activity.
- Use the same engine operations and state as the CLI. Prevent duplicate engine instances for a workspace.
- Notify on meaningful failures or conflicts without repeating unchanged warnings on every synchronization cycle.

Exit criteria:

- A user can inspect a workspace, finish a session, and resolve a conflict through the desktop interface.
- CLI intent commands continue working while the tray application runs.
- Two-device tests cover presence, conflict notification, resolution propagation, and recovery after restarting the tray host.
- New edits remain safe while a prior session is being summarized or published.

## 0.4 — bundled Git and Windows packaging

Remove the prerequisite to install or configure a separate Git executable.

Work:

- Package a pinned, tested Git distribution and the helpers needed by the supported authentication methods.
- Resolve the packaged executable explicitly. Keep Git configuration and credential selection scoped to Mesh operations.
- Package the application for a normal Windows user account, including a clear install, upgrade, and uninstall experience.
- Review distribution requirements and include the required third-party notices. Record the Git version in diagnostics and maintain an update procedure.
- Test paths containing spaces and non-ASCII characters, locked files, offline installation, and application upgrades with existing workspaces.
- Keep managed documents and recovery data safe across upgrades and uninstall operations. Define any explicit data-removal action separately.

Exit criteria:

- Mesh starts and synchronizes on a clean Windows VM with no Git on `PATH` and no developer tools installed.
- The packaged application uses its intended Git version without changing a user's global Git setup.
- An upgrade preserves workspace identity, pending drafts, intent records, conflicts, and saved credentials.
- Release artifacts have a repeatable build and verification procedure.

## 0.5 — guided account and repository setup

Make the initial experience usable without Git knowledge.

Work:

- Add guided account sign-in and secure credential storage using the selected desktop credential mechanism.
- Support **Create a workspace**, **Join an existing workspace**, and **Connect an existing local folder**.
- For repository creation, present the destination account or organization, repository name, and visibility before creation.
- For joining, show accessible repositories and let the user choose a local destination. Mesh performs the clone and initial synchronization.
- For an existing folder, inspect its files, repository history, and remote configuration. Preserve local and remote content while reconciling differences, and route ambiguous cases into review.
- Handle access denial, expired credentials, cancelled sign-in, interrupted cloning, and partially completed setup with recoverable states.
- Guide AI provider configuration, including local models, a synthetic connection test, and explicit consent for sending text and intent to that provider.
- Use familiar workspace and folder language in the main flow. Provide technical details when they help diagnose a problem.

Exit criteria:

- A first-time user on a clean Windows VM can create or join a workspace and see a file arrive on a second device without using a CLI.
- Existing-folder setup does not silently overwrite documents or discard Git history.
- Repeating or resuming an interrupted setup does not create duplicate repositories or lose captured work.
- Sign-in and provider tests explain the next useful action without exposing credentials.

## 0.6 — login startup and Windows desktop beta

Make daily operation reliable across login, logout, sleep, and application updates.

Work:

- Add a user-controlled **Start Mesh when I sign in** setting and register the packaged application for the user's login session.
- Maintain one desktop host per user context and one engine owner per workspace.
- Prioritize durable local capture during shutdown and logout. Recover pending operations at the next start without depending on a final network request.
- Reconcile after sleep/resume and network changes. Keep startup status visible while recovery is running.
- Respect pause and startup preferences across restarts. Define the behavior of explicit quit versus the next login.
- Exercise the installer and startup registration together, including upgrades and uninstall cleanup.

Exit criteria:

- Repeated login/logout and sleep/resume tests preserve saved edits, drafts, and unresolved conflicts.
- Startup works without administrator privileges or an interactive terminal.
- Startup registration does not leave duplicate processes or stale application entries after upgrades.
- Confirm the remaining lifecycle choices, including screen-lock behavior, before declaring the Windows desktop beta complete.

## 0.8 — authenticated cloud MCP private alpha

Host the engine behind a remote MCP interface with explicit user and workspace authorization.

Work:

- Implement authentication for the Dispatch MCP resource and workspace-level authorization for every operation. Verify the implementation against the current MCP authorization specification.
- Provide a hosted account-linking flow for GitHub. Keep MCP access tokens separate from GitHub credentials and select authorized GitHub credentials per operation.
- Isolate mutable working directories, indexes, and job state by user/agent context. Coordinate concurrent publication to a shared branch.
- Persist workspaces, snapshots, conflict records, and background jobs across service restarts.
- Preserve the initiating principal, linked account reference, workspace, and recorded intent in queued work. Recheck authorization when executing it.
- Model the autonomous agent as a separate, explicitly provisioned principal. Preserve the boundary between its role account and interactive users' authorization.
- Expose an initial tool set through the same application layer used by the desktop engine.

Proposed initial MCP operations:

| Area | Operations |
|---|---|
| Workspaces | List authorized workspaces and inspect synchronization status |
| Content | Read files and apply scoped edits with concurrency checks |
| Intent | Begin and end a purpose scope associated with the caller's changes |
| Sessions | Inspect and optionally finish the caller's session |
| Conflicts | List, inspect, and resolve authorized conflicts; select one whole version for binaries |
| Jobs | Inspect queued work and recoverable failures after a connection closes |

Exit criteria:

- Two authenticated users can work in an authorized shared workspace, and access to another workspace is denied.
- Concurrent operations cannot leak credentials, use another caller's identity, or silently fall back to an autonomous account.
- Work continuing after an MCP disconnect retains its initiating identity and remains inspectable.
- Token expiry, revocation, account unlinking, worker restart, and publication races have tested recovery behavior.
- A remote agent can edit a file, attach purpose, optionally finish its session, and observe synchronization without a local clone.

## 0.9 — cloud beta and operational readiness

Expand beyond the private alpha after the identity and recovery boundaries are validated.

Work:

- Define and implement workspace-sharing rules, retention, backup/restore, deletion, quotas, and deployment capacity for the intended beta audience.
- Add health checks, operational metrics, audit events, and administrative recovery tools with credential-safe logs.
- Test concurrent workers and use an appropriate shared coordination mechanism if deployment extends beyond one host.
- Exercise disaster recovery and restore a workspace together with its drafts, conflict records, authorization references, and pending jobs.
- Define upgrade and protocol-compatibility policies for desktop and cloud clients.

Exit criteria:

- Backup restoration and interrupted-job recovery are demonstrated in a production-like environment.
- Operational limits and retention behavior are documented and visible to administrators and users where relevant.
- Security and authorization review findings are resolved before widening access.

## Decisions to close during delivery

| Decision | Needed before |
|---|---|
| Tray UI toolkit, local control interface, pause semantics, and notification behavior | Implementing 0.3 |
| Git distribution, credential-helper packaging, installer format, and distribution review | Shipping 0.4 |
| Desktop GitHub authorization flow, repository visibility default, and existing-folder reconciliation UX | Shipping 0.5 |
| Startup preference defaults, screen-lock behavior, quit semantics, and supported Windows versions | Shipping 0.6 |
| Cloud hosting, identity provider, workspace sharing, multi-account selection, and first tool schemas | Shipping 0.7 |
| Retention, capacity, quotas, backup objectives, and the boundary between user-directed and autonomous jobs | Widening cloud access in 0.8 |

These decisions do not change the confirmed synchronization and session behavior above.

## Parallel and later tracks

- Keep portability checks running as the shared engine evolves. Schedule macOS and Linux desktop hosts and their startup integration as separate platform releases; prioritize them explicitly alongside the cloud work.
- Add editor integrations for precise editing presence after the recent-save feature is established.
- Add richer text conflict editing, format-aware validation, and optional AI conflict proposals after preservation and manual review are dependable.
- Keep recurring autonomous-agent execution separate from synchronization. Schedule it once the required operations and identity policy are defined.

## Release discipline

For each milestone, turn the work into reviewable implementation tickets and retain the acceptance criteria as release gates. Run engine checks, targeted integration tests, and the platform-specific validation appropriate to the change. New storage or transport formats require migration and compatibility tests.

Use Mesh intent scopes before agent edits, associate purpose with exact changed paths, and end the scope after capture. An agent may optionally finish the session at the end of a logical task. Commit messages should state the delivered behavior and preserve the requested purpose without attributing it to unrelated edits.
