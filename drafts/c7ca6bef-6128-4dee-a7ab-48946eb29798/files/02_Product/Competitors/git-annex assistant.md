---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://git-annex.branchable.com/assistant/
  - https://git-annex.branchable.com/automatic_conflict_resolution/
  - https://github.com/con/git-annex
researched: 2026-11-05
storage_reviewed: 2026-11-06
storage_source_revision: de5ced7ec6db8f4316a0090f37f006ac179f2c07
refresh: Before reuse in a product decision
---
# git-annex assistant

The background folder-synchronization component of git-annex, which manages large-file content separately from Git history and supports multiple storage destinations. [Project homepage](https://git-annex.branchable.com/), [assistant](https://git-annex.branchable.com/assistant/)

## Links and GitHub activity

| Field | Snapshot on 2026-11-05 |
|---|---|
| GitHub repo | No canonical GitHub repository identified; [con/git-annex](https://github.com/con/git-annex) is an explicitly **unofficial** testing/distribution repository with an upstream mirror branch |
| Canonical source | [Official source browser](https://source.git-annex.branchable.com/); the [download page](https://git-annex.branchable.com/download/) gives `git://git-annex.branchable.com/` |
| Homepage | [git-annex assistant](https://git-annex.branchable.com/assistant/) |
| GitHub stars | N/A for canonical project; **30 for con/git-annex only** |
| GitHub forks | N/A for canonical project; **5 for con/git-annex only** |
| Posts in GitHub Discussions | N/A for canonical project; unofficial repo returns 0 and has Discussions disabled |

Mirror counts: [REST metadata](https://api.github.com/repos/con/git-annex) and GraphQL. They are not git-annex popularity totals. The project has its own [forum](https://git-annex.branchable.com/forum/).

## Feature list

- Background folder synchronization through the assistant.
- Large-file content storage outside Git; offline/online backup and archive use cases.
- Multiple storage destinations, checksums, and encryption capabilities in git-annex. [Homepage](https://git-annex.branchable.com/), [assistant](https://git-annex.branchable.com/assistant/)
- Conflicts become deterministic `.variant-*` files so synchronization can continue; users decide whether to combine or retain them. [Conflict design](https://git-annex.branchable.com/automatic_conflict_resolution/)

## What distinguishes it from Mesh

Its storage architecture targets distributed large content; Mesh currently stores content as native Git blobs with preview size limits. Variant files provide operational conflict handling, rather than a semantic decision. Mesh preserves conflict inputs in durable synchronized records and keeps the accepted file visible. See [[Competitor overview#Mesh comparison baseline]].

## What Dispatch could adapt

Recommendations: use deterministic conflict identities and test convergence across independent resolution/publication orders. Offer understandable export/recovery without replacing Mesh's durable records. Evaluate content availability and replication policies if large-file storage becomes a priority. See [[Dispatch Mesh SWOT analysis]].
