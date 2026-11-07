---
type: index
owner: "[[Kai]]"
updated: 2026-11-06
---
# Dispatch Mesh wiki — index

Entry point for people and agents. Read this first, then follow the links. Every page is listed here; every change gets an entry in [[log]].

## Conventions

- ADRs use `accepted`, `proposed` (agreed, not yet built), and `superseded`, following the ADR template. Accepted and proposed ADRs bind; other pages use `status: approved` and `source_of_truth: true` to mark approved source material.
- Precedence when documents conflict: accepted and proposed ADRs (both bind; only superseded ones may be ignored) → [[Dispatch Mesh requirements and architecture]] → [[Dispatch Mesh milestone plan]] → tickets → discovery notes.
- The wiki is the source of truth for ticket status. There is no external tracker.
- `01_Sources/` is immutable: add raw artifacts as `YYYY-MM-DD - <origin> - <title>`, never edit or delete them; write new wiki pages from them and log the ingest.
- Every page carries `owner:` — a person in `00_Start-Here/Team/`, never a team.
- Workflow logic lives in the code repository under `dispatch/workflow/`, not here. Shared rules: `dispatch/invariants.md` in the repo.

## Start here

- [[log]] — append-only change ledger
- [[Kai]] — team

## Product

- [[Dispatch Mesh concept]] — concept updates and the original discovery discussion
- [[Dispatch Mesh requirements and architecture]] — current requirements, session model, editing presence
- [[Dispatch Mesh milestone plan]] — 0.2.x → 2.0 roadmap and release gates
- [[Dispatch Mesh naming recommendations]]
- [[Competitor overview]] — dated research, metric definitions, and maintenance register
- [[Dispatch Mesh SWOT analysis]] — competitive assessment and recommended product actions
- Competitor profiles: [[gitwatch]], [[SparkleShare]], [[Obsidian Git]], [[Agentic Git Sync]], [[git-annex assistant]], [[Syncthing]], [[Jujutsu]] (architectural reference)

## Discovery

- [[Classical Git workflow for Mesh]] — ordinary remote file trees and Git workflow alternatives; benefits, drawbacks, mitigations, residual tradeoffs and prototype decision criteria. Owner: [[Kai]]; review before an architecture decision.
- [[AI-managed synchronization and human decisions]] — AI proposes meaning-preserving text for technical conflicts; humans approve the exact result. Includes scenarios, prompts and a synthetic Chat Completions exchange; general consistency review is deferred. Owner: [[Kai]]; review before refinement.

## Requirements

- `05_Requirements/Tickets/` — tickets (the Dispatch board)
  - [[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]] — combine compatible text edits with an AI proposal and human approval of the exact result (unscheduled)
  - [[Story - US00001 - Tray icon and status window host|US00001]] — tray icon and status window host (v0.3.0)
  - [[Story - US00002 - Local engine control interface for the tray host|US00002]] — local engine/control interface for the tray host (v0.2.5)
  - [[Task - TASK00001 - Close remaining 0.2.x stabilization gaps|TASK00001]] — close remaining 0.2.x stabilization gaps (v0.2.5)
  - [[Task - TASK00002 - Choose the Git backend before bundling Git|TASK00002]] — choose the Git backend before bundling Git (v0.4.0)
  - [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]] — exclude device-local files such as `.obsidian/workspace.json` from sync, including already-synchronized ones
  - [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] — workspace detail: session stages, Finish session, recovery information (v0.3.0)
  - [[Story - US00006 - Conflict list and resolution in the status window|US00006]] — conflict list, comparison and resolution in the status window (v0.3.0)
  - [[Story - US00007 - Recent-save presence in the status window|US00007]] — recent-save presence per file in the status window (v0.3.0)
  - [[Story - US00008 - Desktop notifications for conflicts and attention|US00008]] — notifications for new conflicts and attention starting or clearing (v0.3.0)
  - [[Story - US00009 - Pause and resume synchronization|US00009]] — pause and resume, after deciding what pause suspends (v0.3.0)
  - [[Story - US00010 - Resolve a conflict after the file was edited again|US00010]] — resolving a conflict after the file was edited again reconciles that edit onto the chosen version (v0.3.0)
  - [[Story - US00011 - Limit saved copies in the recovery folder|US00011]] — keep only useful saved copies in `.mesh/recovery`, so the folder, status and window list stop growing without bound
  - [[Story - US00012 - Status window layout follows the design frames|US00012]] — restyle the status window header, state card and activity list to the design frames (v0.3.0)
  - [[Bug - BUG00001 - AI summaries of large sessions never finish|BUG00001]] — the summary step gives up on a busy workspace lock after 0.5 s, so large sessions stay in Finishing up
  - [[Bug - BUG00002 - One large text file makes the whole AI summary fall back|BUG00002]] — one text file over the 512 KiB input cap costs the AI summary for the whole session
  - [[Task - TASK00003 - Two-device tests for the 0.3 exit criteria|TASK00003]] — two-device tests for the 0.3 exit criteria (v0.3.0)

## UX

- `06_UX/`
  - [[Claude Design briefing — desktop app]] — design brief for the tray icon, status window, conflicts, presence and guided setup
  - [[Dispatch Mesh desktop app — design spec]] — the design frames as text: tokens, glyph paths, components and per-view rules; frame PNGs in `06_UX/frames/`, frame markup in `06_UX/frames/source/`

## Engineering

- [[Competitor storage approaches]] — source-backed comparison of all seven competitor storage models and three proposed Mesh adaptations; refresh with linked profiles before design adoption (owner: [[Kai]]).
- [[SQLite database format]] — tables and columns, JSON state properties, logical ER diagram, blob references, transactions, recovery, migration, and retention. Maintenance: [[Kai]] reviews on schema, serialization, migration, or recovery changes and before reuse with a newer release.
- [[Device branch format]] — version-two device branch tree, complete `state.json` field reference, session lifecycle, `base/` versus `files/`, and indefinite preview retention. Maintenance: [[Kai]] reviews on transport/schema, session lifecycle, or retention changes and before reuse with a newer release; source revision recorded on the page.
- `07_Engineering/Decisions/` — ADRs

| ADR | Decision | Status |
|---|---|---|
| [[ADR-0001 — Synchronize drafts separately from permanent history\|ADR-0001]] | Synchronize drafts separately from permanent history | accepted |
| [[ADR-0002 — Group files into sessions with an inactivity boundary\|ADR-0002]] | Group files into sessions with an inactivity boundary | accepted |
| [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions\|ADR-0003]] | Preserve synchronized conflict inputs and traceable resolutions | accepted |
| [[ADR-0004 — Bind agent intent to attributable changes\|ADR-0004]] | Bind agent intent to attributable changes | accepted |
| [[ADR-0005 — Make AI summaries configurable and opt-in\|ADR-0005]] | Make AI summaries configurable and opt-in | accepted |
| [[ADR-0006 — Let Mesh own Git operations in managed folders\|ADR-0006]] | Let Mesh own Git operations in managed folders | accepted |
| [[ADR-0007 — Persist state transactionally with recovery journals\|ADR-0007]] | Persist state transactionally with recovery journals | accepted |
| [[ADR-0008 — Store content as blobs referenced by metadata\|ADR-0008]] | Store content as blobs referenced by metadata | accepted |
| [[ADR-0009 — Embed the engine with a runner handle and published status\|ADR-0009]] | Embed the engine with a runner handle and published status | accepted |
| [[ADR-0010 — Report workspace status and session stages separately\|ADR-0010]] | Report workspace status and session stages separately | proposed |
| [[ADR-0011 — Build the desktop host with Tauri 2\|ADR-0011]] | Build the desktop host with Tauri 2 | accepted |
| [[ADR-0012 — Ship the desktop host as a separate executable serving several workspaces\|ADR-0012]] | Ship the desktop host as a separate executable serving several workspaces | proposed |
| [[ADR-0013 — Derive exclusion from each tree's own ignore policy\|ADR-0013]] | Derive exclusion from each tree's own ignore policy | proposed |

ADR-0001–0008 record implemented console-preview decisions. Their original individual decision dates are unavailable; each records its provenance and documentation date. ADR-0009 and ADR-0010 record Kai's designs dated 2026-11-05. ADR-0009 was accepted on 2026-11-05 after US00002 was built and verified; ADR-0010's implementation and verification remain in its linked ticket. ADR-0011 and ADR-0012 record Kai's US00001 decisions dated 2026-11-05; they are built under that ticket. ADR-0011 was accepted on 2026-11-06 after US00001 was built and verified. ADR-0013 records Kai's US00003 decision of 2026-11-06 and is built under that ticket. TASK00002's Git backend decision remains separate and open.

## Delivery and QA

- `08_Delivery-and-QA/Releases/` — release notes (`version` + `date`, feeds the Release Plan)
  - [[v0.2.5]] — engine stabilization: status model, AI failures, in-process engine interface (2026-11-05)
    - [[v0.2.5 GitHub release]] — paste-ready GitHub release text, links to the public wiki repo (`GitHub/`)
- `08_Delivery-and-QA/Reports/` — verification reports
  - [[github-workflow-test-2026-11-04]] — live GitHub workflow test of 0.2.4, per-case results
  - [[github-workflow-findings-2026-11-04]] — findings and known limitations from that run

## Meetings

- `09_Meetings/` — meeting notes (feeds the Meetings tab)

## Templates

- `00_Start-Here/Templates/` — ticket-story, ticket-bug, adr, release-note, meeting. Parsed by the board; do not improvise notes without them.
