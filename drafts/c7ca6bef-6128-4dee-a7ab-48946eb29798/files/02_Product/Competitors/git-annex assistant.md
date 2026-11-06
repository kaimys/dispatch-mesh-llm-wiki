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

## Storage architecture — source review (2026-11-06)

Inspected revision: [`de5ced7ec6db`](https://github.com/con/git-annex/commit/de5ced7ec6db8f4316a0090f37f006ac179f2c07). Static source inspection; no competitor was built or run. Existing feature claims and activity counts above retain their original research date.

**Source provenance.** Inspected the `upstream/master` snapshot in the explicitly unofficial `con/git-annex` mirror, not that repository's packaging/default branch. The canonical source browser was inaccessible to the web tool. Some documentation paths exceeded Windows checkout limits; the cited upstream source files were read directly from the fetched Git objects. This is a pinned source review, not a verified assistant deployment.

**Repository and local layout.** Annexed payloads live under `.git/annex/objects/`, addressed by annex keys; tracked links/pointers refer to them instead of putting all payload bytes into ordinary Git history. Locked and unlocked representations need different handling. A separate `git-annex` branch carries metadata; local journal changes are staged into its index before merging other metadata branches. Location logs associate content with repository UUIDs and presence changes. [Sources: object/link paths](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Annex/Locations.hs), [pointer representation](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Annex/Content/PointerFile.hs), [metadata branch](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Annex/Branch.hs), [location log](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Logs/Location.hs).

**Local index and recovery.** The SQLite keys database maintains key/file associations and content/inode information. It complements the object store and Git metadata. Automatic merge handling for conflicts involving annexed files creates key-derived variant names; it does not claim to semantically resolve all ordinary text conflicts. Content drop operations have explicit copy-safety machinery, including proofs whose validity can expire. [Sources: keys schema](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Database/Keys/SQL.hs), [automatic merge](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Annex/AutoMerge.hs), [drop logic](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Annex/Drop.hs), [proof expiry](https://github.com/con/git-annex/blob/de5ced7ec6db8f4316a0090f37f006ac179f2c07/Annex/SafeDropProof.hs).

**Pros (assessment).** Large content can have independent transfer and local-availability policies. Explicit availability metadata and guarded removal are useful patterns for bounded local storage. Deterministic variant identities aid convergence.

**Cons (assessment).** Git history alone does not guarantee payload availability. Objects, location records, local databases and remote storage add operational complexity. Variant files change the visible namespace. Mesh cannot adopt this representation without an explicit large-file protocol and migration design.

Comparison and selected Mesh adaptations: [[Competitor storage approaches]].

