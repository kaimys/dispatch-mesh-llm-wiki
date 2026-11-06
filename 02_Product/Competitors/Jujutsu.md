---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://github.com/jj-vcs/jj
  - https://docs.jj-vcs.dev/latest/working-copy/
  - https://docs.jj-vcs.dev/latest/operation-log/
researched: 2026-11-05
storage_reviewed: 2026-11-06
storage_source_revision: 2f76430997be91fa1557ca7f5fa323e3ed004dcc
refresh: Before reuse in a product decision
---
# Jujutsu

A Git-compatible version control system for software development. Included because the concept names it as an architectural reference; it is not a background folder-sync competitor. [Project README](https://github.com/jj-vcs/jj#readme)

## Links and GitHub activity

| Field | Snapshot on 2026-11-05 |
|---|---|
| GitHub repo | [jj-vcs/jj](https://github.com/jj-vcs/jj) |
| Homepage | [jj-vcs.dev](https://jj-vcs.dev/) |
| GitHub stars | 32,366 |
| GitHub forks | 1,277 |
| Posts in GitHub Discussions | [738 discussion threads](https://github.com/jj-vcs/jj/discussions); enabled |

Counts: [REST metadata](https://api.github.com/repos/jj-vcs/jj) and GraphQL. See [[Competitor overview#Research method and maintenance]].

## Feature list

- Git-backed content storage and Git forge interoperability. [README](https://github.com/jj-vcs/jj#readme)
- A working-copy commit updated automatically by most commands, implicit file tracking, and persistent conflict state with incremental resolution. This is command-triggered snapshotting, not continuous background synchronization. [Working-copy design](https://docs.jj-vcs.dev/latest/working-copy/)
- An operation log recording repository views, with undo, operation revert, and restore commands. [Operation log](https://docs.jj-vcs.dev/latest/operation-log/)

## What distinguishes it from Mesh

Jujutsu is an explicit developer VCS workflow. Mesh captures saved edits autonomously, exchanges open-session drafts, and publishes immutable shared session commits. Its console preview still requires Git installation/authentication; hiding setup is roadmap work. See [[Competitor overview#Mesh comparison baseline]].

## What Dispatch could adapt

Recommendations: study stable change identity, inspectable operation history, and recovery UX. Mesh could explain how a session evolved and preview recovery actions. Preserve immutable shared commits; adopting a recovery concept does not require adopting history-rewriting workflows. See [[Dispatch Mesh SWOT analysis]].

## Storage architecture — source review (2026-11-06)

Inspected revision: [`2f76430997be`](https://github.com/jj-vcs/jj/commit/2f76430997be91fa1557ca7f5fa323e3ed004dcc). Static source inspection; no competitor was built or run. Existing feature claims and activity counts above retain their original research date.

**Repository and local layout.** The Git backend stores file/tree/commit objects in Git, either with its managed backend repository or a colocated Git repository. Jujutsu adds local metadata under `.jj/`: extra backend metadata, repository views, operation records and working-copy state. The backend uses `refs/jj/keep/` references to retain needed Git commits; it also encodes change identity/conflict information with Jujutsu-aware logic. A Git-only view is therefore not the complete Jujutsu model. [Source: Git backend](https://github.com/jj-vcs/jj/blob/2f76430997be91fa1557ca7f5fa323e3ed004dcc/git-backend/src/git_backend.rs).

**Operation history.** The simple operation store writes separate content-addressed protobuf files under `views/` and `operations/`. Operations link parent operations and repository views; working-copy state separately persists `tree_state` and `checkout`. This allows repository-state recovery beyond merely remembering the latest commit. Operation storage has garbage-collection logic, so retention is an explicit concern rather than unlimited by definition. These local records are not automatically transported by an ordinary Git push. [Sources: operation store](https://github.com/jj-vcs/jj/blob/2f76430997be91fa1557ca7f5fa323e3ed004dcc/lib/src/simple_op_store.rs), [operation schema](https://github.com/jj-vcs/jj/blob/2f76430997be91fa1557ca7f5fa323e3ed004dcc/lib/src/protos/simple_op_store.proto), [working-copy persistence](https://github.com/jj-vcs/jj/blob/2f76430997be91fa1557ca7f5fa323e3ed004dcc/lib/src/local_working_copy.rs).

**Pros (assessment).** Separating content, logical change identity, repository view and operations makes earlier states explainable and recoverable. Explicit keep references address Git object reachability.

**Cons (assessment).** More stores, indexes and metadata formats must remain consistent. Git interoperability does not transport the entire local operation history. Jujutsu's rewrite-oriented developer workflow cannot be copied wholesale into Mesh's immutable shared history. The useful adaptation is inspectable local operations and recovery references, not replacing Mesh sessions with mutable shared commits.

Comparison and selected Mesh adaptations: [[Competitor storage approaches]].

