---
type: briefing
status: draft
owner: "[[Kai]]"
updated: 2026-11-05
---
# Claude Design briefing — Dispatch Mesh desktop app

A brief for Claude Design. It asks for the visual and interaction design of the Dispatch Mesh desktop app: the tray icon, the tray menu, the status window, and the conflict, presence and session views that follow it. Everything here comes from approved wiki sources. Where a decision is still open, the brief says so and asks the design to explore options instead of assuming one.

Sources, in precedence order: [[ADR-0010 — Report workspace status and session stages separately|ADR-0010]], [[ADR-0009 — Embed the engine with a runner handle and published status|ADR-0009]], [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions|ADR-0003]], [[Dispatch Mesh requirements and architecture]], [[Dispatch Mesh milestone plan]], [[Story - US00001 - Tray icon and status window host|US00001]], [[Dispatch Mesh concept]], and the repository `README.md` (status table and recovery procedures).

## 1. The product in one paragraph

Dispatch Mesh synchronizes a folder between people, devices and AI agents, as easily as Google Drive, and keeps Git-backed history of **who changed what and why**. Users never see Git. Saved edits sync continuously as *drafts*. Related edits across several files are grouped into a *session*, which closes after five minutes without local edits, or earlier when the user chooses **Finish session**. A closed session becomes one permanent entry in shared history, with an AI-written summary if AI is enabled, or a factual local summary if it is not. When an agent records the purpose of its edits ("intent"), that purpose travels with exactly those changes. Conflicts are never silently resolved. Both versions are kept, the already accepted version stays visible, and unrelated files keep syncing.

Tagline from [[Dispatch Mesh naming recommendations]]: *File synchronization with history and intent. For teams and agents.* The name is not final; keep the wordmark replaceable.

## 2. Who it is for

- **Primary:** people on small teams who maintain shared documents, such as an LLM wiki or an Obsidian vault, and who **do not know Git**. They want to trust that their work is saved and shared, and to know when something needs them.
- **Secondary:** people who run AI agents that edit the same folder. They want to see which changes came from an agent and what purpose it was given.
- **Not the audience:** Git experts who want branches, commits and diffs exposed. Technical details may appear only on request, for diagnosis.

## 3. Design principles

1. **Quiet by default, clear when needed.** Mesh lives in the background. The icon and window must make "everything is fine" boring and "you need to act" impossible to miss. Don't repeat unchanged warnings on every sync cycle.
2. **Work is never lost, so say so.** The strongest message the UI can give is that saved edits are safe, even offline, even with a conflict, even after Quit. Offline is calm, not alarming.
3. **Familiar words, not Git words.** Use *workspace, folder, file, saved, shared, history, version*. Avoid *commit, branch, push, merge, ref, main*. See §5.
4. **Recorded intent is a request, not proof.** When showing an agent's purpose, label it as the *requested purpose*. Never present it as what was achieved. Inferred reasons must be labeled *inferred*.
5. **Advisory, never blocking.** Presence ("Alice recently changed this file") informs but does not lock. Conflicts don't stop unrelated files.
6. **Every problem has a next step.** An attention state always shows its reason and one useful action. Never show credentials, raw Git output or provider response bodies.

## 4. The status model the UI renders

The engine publishes one **workspace summary** and a **stage per session** ([[ADR-0010 — Report workspace status and session stages separately|ADR-0010]]). These are fixed. The design decides how they look.

### Workspace summary (highest precedence wins)

| Engine value | Suggested user label | Meaning for the user | Tone |
|---|---|---|---|
| `needs_attention` | Needs attention | Something needs you: a conflict, a refused sign-in, an unreadable file, shared history changed outside Mesh, or an AI summary failure. A `reason` is always supplied. | Alert |
| `offline` | Offline — changes saved on this device | The remote can't be reached. Capture continues, and everything uploads on the next successful sync. A `reason` is supplied. | Calm, informational |
| `saved_locally` | Saved on this device | Edits are captured but not yet shared as a draft. Usually lasts seconds. | Working |
| `draft_uploaded` | Shared as draft | The open session is visible to collaborators but not yet in permanent history. | Working |
| `waiting_for_finalization` | Finishing up | A closed session is waiting for summary, conflict resolution, earlier sessions, or publication. | Working |
| `synced` | Up to date | Everything is published and nothing is waiting. | Idle |

The higher states hide lower-priority activity, which stays visible in the session detail.

### Session stages

`saved_locally` → `draft_uploaded` → `waiting_for_finalization` → `published`, or `discarded`. A session ends as `discarded` when its edits cancelled out, so no empty history entry is created. Show this as a progress track or timeline per session. Sessions received from other devices start at "uploaded".

### Attention reasons to design for (from the README recovery procedures)

| Reason (as the engine words it) | What the user should do | Action in UI |
|---|---|---|
| "N unresolved conflicts" | Review and choose a version | Open conflicts |
| "Remote refused access" | Fix sign-in or repository permission | Sign in again / Retry |
| "Remote rejected the update" | Usually clears itself on the next cycle | Retry now |
| "Shared history was rewritten outside Mesh" / "The shared branch was deleted outside Mesh" | Someone changed shared history with Git directly. Sync is paused and local work is kept. | Show details / Retry |
| "Local capture waiting: …" | A named file can't be read (symlink, case collision, newly ignored path) | Show file |
| "Edits depend on a received draft …" | Run Finish session | Finish session |
| "AI summary failed: …" | Fix the key, quota or endpoint. Publication continues with a local summary. | Open AI settings |

Offline reasons (DNS, connection refused, timeout, TLS) come with the reassurance *"Keep working — your changes upload when the connection returns."*

## 5. Vocabulary

| Concept (engine) | Say in the UI | Avoid |
|---|---|---|
| Workspace / managed folder | Workspace, folder | Repository, repo |
| Session | Editing session, or just "your recent changes" | Commit, draft commit |
| Draft upload | Shared as draft | Pushed to device branch |
| Publication to `main` | Added to shared history | Committed, merged, pushed to main |
| Conflict, accepted / incoming | Conflict: *current version* / *other version* | Ours/theirs, HEAD |
| Recorded intent | Requested purpose (by agent or person) | Prompt, commit body |
| Recovery record | Saved copy | Backup blob |
| Recent-save presence | "Alice recently changed this file · 12 s ago" | Lock, editing now |

Presence comes from saved changes only. Never word it as "is editing" or "has the file open". That signal is reserved for future editor integrations and must look different.

## 6. Surfaces to design

Scope follows the milestone plan. **Design 0.3 in full. Sketch 0.5 at flow level.** Later items are listed only so the system can grow into them.

### 6.1 Tray icon (0.3, [[Story - US00001 - Tray icon and status window host|US00001]])

- A set of icon states for the Windows notification area (16 and 32 px, light and dark taskbar), then the macOS menu bar (template image, monochrome) and Linux trays.
- At minimum: **Up to date**, **Working** (saved locally, draft uploaded, finishing up), **Offline**, **Needs attention**.
- **Open question for the design:** do *saved locally*, *draft uploaded* and *finishing up* get distinct icons, or share one "working" icon? Do we reserve a **Paused** state now? Pause is not yet in the engine model. Please propose an answer with a rationale.
- Tooltip names the state, for example "Dispatch Mesh — Up to date" or "Dispatch Mesh — Needs attention: 2 unresolved conflicts".
- State must not depend on color alone (accessibility, monochrome menu bars).

### 6.2 Tray menu (0.3)

- Left-click opens the status window.
- Menu: **Open status window**, **Finish session** (later 0.3 ticket), **Pause / Resume** (later 0.3 ticket; what pause suspends is not yet defined), **Quit**.
- **Quit** stops Mesh after capturing saved edits. It doesn't wait for the network, and nothing is lost. **Open question:** if a session is open, should Quit first *offer* "Finish session"? Please mock both variants.
- Closing the status window only hides it. Mesh keeps running. Make that obvious the first time.

### 6.3 Status window (0.3)

The central surface. It needs these views or sections:

1. **Header / overview.** Workspace name and folder path, the summary state with its reason, the last successful sync time, and the primary action when attention is needed.
2. **Activity.** Recent sessions with their stage track: files touched, author (you, a collaborator, or an agent), requested purpose if recorded, and the resulting history entry with its summary title once published. Incoming changes from others appear as theirs, never as yours.
3. **Conflicts.** The list of unresolved conflicts: path, competing authors, timestamps, and which version is currently shown. A badge count drives the tray state.
4. **Conflict detail.** Side-by-side or inline text comparison between *current version* and *other version*, with the common base available. Authors, sessions and any requested purpose appear per side. Actions: **Keep current version** / **Use other version**. For binary files, show previews or metadata and offer whole-file choice only, with no merge. State clearly that both versions are kept after resolution. Show what happens to edits made since the conflict was recorded: they are protected and reconciled separately.
5. **Presence.** Per file: who saved recently, and when. Entries expire after five minutes. Show it in the activity list and as a non-blocking notice on the file.
6. **Already running.** If another Mesh (CLI or second tray) owns the workspace: "Mesh is already running for this workspace" with **Retry**.
7. **Empty / first-run state** for a workspace that has just been initialized.

**Open question:** one window per workspace, or one window for several workspaces from the start? The milestone speaks of "each workspace's state". Please design for **several workspaces**, with a one-workspace layout that doesn't feel empty, and show how the tray aggregates their states.

### 6.4 Notifications (0.3)

Native toasts for meaningful changes only: a new conflict, an attention state that starts, and an attention state that clears. Never one per sync cycle. Please supply wording and a frequency rule.

### 6.5 Guided setup (0.5, flow-level sketch)

Three entry paths with no Git knowledge required:

- **Create a workspace.** Choose the destination account or organization, the name and the visibility *before* anything is created.
- **Join an existing workspace.** Pick from accessible workspaces, choose a local folder, and Mesh downloads and syncs.
- **Connect an existing folder.** Mesh inspects the folder and preserves its existing files and history. Ambiguous cases go to a review step and are never silently overwritten.

Also: account sign-in, recoverable states (cancelled sign-in, access denied, interrupted download, partial setup that resumes without duplicates), and **AI provider setup**. AI setup covers a provider or local model, a synthetic connection test, and explicit consent to send text and requested purposes to that provider. AI is opt-in, and the consent screen must make clear what leaves the device.

### 6.6 Later, don't design yet

Login startup setting (0.6), macOS/Linux platform adaptations (0.7/0.8), editor-integration presence, written text-conflict resolution and AI proposals, and cloud/MCP access (2.0).

## 7. Constraints

- **Platform:** Windows 11 first, then macOS menu bar, then Linux trays. Respect each platform's tray conventions and light/dark themes.
- **Toolkit not chosen yet** (open in [[Story - US00001 - Tray icon and status window host|US00001]]: webview, Tauri, egui or native). Keep the design implementable as a small, compact desktop window, not a web-app shell. Avoid effects that only a browser engine can render cheaply.
- **Compact footprint.** A utility window of about 420–480 px wide for the overview, expandable for conflict comparison.
- **Accessibility:** WCAG AA contrast, keyboard operation, screen-reader labels for icon states, and state never shown by color alone.
- **Privacy in UI:** never display credentials, API keys, raw Git errors or provider response bodies, even in "details".
- **Honesty:** don't show features as available before their milestone. Disabled or "coming later" placeholders are fine in exploration frames but must be labeled.

## 8. Realistic sample data for mockups

- Workspace "Team Wiki" at `C:\Users\alex\Documents\Team Wiki`. Collaborators: Alice (laptop), Bob (desktop), and "Research agent" (an agent working on Bob's device).
- Session, published: 3 files (`onboarding/setup.md`, `onboarding/troubleshooting.md`, `index.md`). Title "Clarify setup steps and add troubleshooting examples". Requested purpose (by Research agent): "Clarify onboarding for new colleagues". Applies to the two onboarding files only.
- Session, draft uploaded: Alice, `meetings/2026-11-04.md`, last saved 40 s ago.
- Text conflict: `policies/retention.md`. Current version "retention: 60 days" (Alice). Other version "retention: 90 days" (Bob). Base "retention: 30 days".
- Binary conflict: `assets/logo.png`. Two complete versions from Alice and Bob.
- Presence: "Alice recently changed this file. Last saved 12 seconds ago."
- Offline reason: "Could not reach the remote (connection timed out)".

## 9. Deliverables requested

1. Tray icon set: all states, Windows sizes, light/dark, plus a macOS template-image variant.
2. Tray menu, with both Quit variants.
3. Status window: overview, activity, conflicts list, text conflict detail, binary conflict detail, presence, already-running, first-run. One frame for each summary state (§4), and the multi-workspace variant.
4. Notification set with wording.
5. Guided-setup flow sketch (§6.5), including the AI consent screen.
6. A short component and token sheet (color, type, spacing, status glyphs) so the result can be built in whichever toolkit is chosen.
7. A note answering the open questions: icon granularity, a reserved Paused state, the Quit prompt, workspaces per window, and showing the attention reason in the 0.3 shell.

## 10. Open product questions the design should inform

From [[Story - US00001 - Tray icon and status window host|US00001]] and the milestone's *Decisions to close during delivery*:

- Which of the six summary states get distinct icons?
- Is a *Paused* state reserved now? Pause semantics aren't defined, and adding it means extending ADR-0010.
- Should Quit with an open session offer *Finish session* first?
- One host for one workspace, or for several?
- Should the 0.3 window shell already show the attention `reason`? The engine supplies it. The recommendation is yes.
- What notification frequency stays helpful without nagging?

Design answers are proposals. Decisions are recorded in the tickets or ADRs.
