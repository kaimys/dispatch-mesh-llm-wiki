---
type: index
owner: "[[Kai]]"
updated: 2026-11-04
---
# Dispatch Mesh wiki — index

Entry point for people and agents. Read this first, then follow the links. Every page is listed here; every change gets an entry in [[log]].

## Conventions

- A page is authoritative only with `status: approved` and `source_of_truth: true`. Everything else is working material.
- Precedence when documents conflict: approved ADRs and proposed ADRs (both bind; only superseded ones may be ignored) → [[Dispatch Mesh requirements and architecture]] → [[Dispatch Mesh milestone plan]] → tickets → discovery notes.
- The wiki is the source of truth for ticket status. There is no external tracker.
- `01_Sources/` is immutable: add raw artifacts as `YYYY-MM-DD - <origin> - <title>`, never edit or delete them; write new wiki pages from them and log the ingest.
- Every page carries `owner:` — a person in `00_Start-Here/Team/`, never a team.
- Workflow logic lives in the code repository under `dispatch/workflow/`, not here. Shared rules: `dispatch/invariants.md` in the repo.

## Start here

- [[log]] — append-only change ledger
- [[Kai]] — team

## Product

- [[Dispatch Mesh requirements and architecture]] — current requirements, session model, editing presence
- [[Dispatch Mesh milestone plan]] — 0.2.x → 2.0 roadmap and release gates
- [[Dispatch Mesh naming recommendations]]

## Discovery

- [[04_Discovery/git-sync/README|git-sync discovery]] — concept updates and the original discovery discussion

## Requirements

- `05_Requirements/Tickets/` — tickets (the Dispatch board)

## Engineering

- `07_Engineering/Decisions/` — ADRs

## Delivery and QA

- `08_Delivery-and-QA/Releases/` — release notes (`version` + `date`, feeds the Release Plan)

## Meetings

- `09_Meetings/` — meeting notes (feeds the Meetings tab)

## Templates

- `00_Start-Here/Templates/` — ticket-story, ticket-bug, adr, release-note, meeting. Parsed by the board; do not improvise notes without them.
