---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://syncthing.net/
  - https://docs.syncthing.net/users/syncing.html
researched: 2026-11-05
storage_reviewed: 2026-11-06
storage_source_revision: b34734376128492fa4e1b01412193de033e20613
refresh: Before reuse in a product decision
---
# Syncthing

A continuous device-to-device folder synchronizer with encrypted connections and explicit device authorization. It substitutes for Mesh when moving files matters more than Git history. [Homepage](https://syncthing.net/)

## Links and GitHub activity

| Field | Snapshot on 2026-11-05 |
|---|---|
| GitHub repo | [syncthing/syncthing](https://github.com/syncthing/syncthing) |
| Homepage | [syncthing.net](https://syncthing.net/) |
| GitHub stars | 90,113 |
| GitHub forks | 5,578 |
| Posts in GitHub Discussions | 0 returned by GraphQL; Discussions disabled |

Counts: [REST metadata](https://api.github.com/repos/syncthing/syncthing) and GraphQL. Community conversation is on its [own forum](https://forum.syncthing.net/), so disabled GitHub Discussions does not imply no community.

## Feature list

- Continuous synchronization, TLS, authenticated device IDs, and a browser management interface across desktop platforms. [Homepage](https://syncthing.net/)
- Watchers plus periodic scans, hash-verified block transfers, and temporary-file application.
- Propagated conflict copies rather than text merging. [Synchronization design](https://docs.syncthing.net/users/syncing.html)
- Optional per-folder, per-device versioning. It defaults off and archives replaced/deleted content received from peers, not earlier local edits. [Versioning](https://docs.syncthing.net/users/versioning.html)

## What distinguishes it from Mesh

Syncthing transfers files without a Git-hosted shared history, session summaries, or scoped recorded intent. Mesh uses a repository as transport and audit history; a peer need not be online at the same time once its draft is uploaded. Syncthing offers an existing device-management UI that Mesh's preview lacks. Baseline: [[Competitor overview#Mesh comparison baseline]].

## What Dispatch could adapt

Recommendations: benchmark watcher recovery, interrupted transfer handling, large-folder resource use, and clear device/folder status. Mesh already combines watching and scans; the opportunity is verification and polish. Define retention and restore behavior explicitly before adding cleanup. See [[Dispatch Mesh SWOT analysis]].

## Storage architecture — source review (2026-11-06)

Inspected revision: [`b34734376128`](https://github.com/syncthing/syncthing/commit/b34734376128492fa4e1b01412193de033e20613). Static source inspection; no competitor was built or run. Existing feature claims and activity counts above retain their original research date.

**Repository and local layout.** Syncthing does not use Git for synchronized user data. At this revision it keeps synchronization metadata in SQLite: the schema has per-device file records, separately interned names and versions, serialized FileInfo records, deduplicated block lists, and local block lookup rows. The block rows contain hashes/offsets/sizes, not an archive of every file's payload. Working files, temporary transfers and optional archived versions live on the filesystem. This describes current source, not older database implementations. [Sources: file schema](https://github.com/syncthing/syncthing/blob/b34734376128492fa4e1b01412193de033e20613/internal/db/sqlite/sql/schema/folder/20-files.sql), [block schema](https://github.com/syncthing/syncthing/blob/b34734376128492fa4e1b01412193de033e20613/internal/db/sqlite/sql/schema/folder/50-blocks.sql), [database setup](https://github.com/syncthing/syncthing/blob/b34734376128492fa4e1b01412193de033e20613/internal/db/sqlite/db_folderdb.go).

**Transfer and recovery.** The receive path reconstructs files through temporary paths, reuses available blocks, verifies incoming block hashes, and installs completed files. Conflicts can become timestamp/device-tagged `.sync-conflict-...` files. An enabled versioner archives replaced/deleted files in the receive path. The staggered implementation thins retained versions over increasing age intervals and exposes restore operations. [Sources: receive pipeline](https://github.com/syncthing/syncthing/blob/b34734376128492fa4e1b01412193de033e20613/lib/model/folder_sendrecv.go), [staggered retention](https://github.com/syncthing/syncthing/blob/b34734376128492fa4e1b01412193de033e20613/lib/versioner/staggered.go), [version storage helpers](https://github.com/syncthing/syncthing/blob/b34734376128492fa4e1b01412193de033e20613/lib/versioner/util.go).

**Pros (assessment).** Indexed metadata and block reuse support incremental work without serializing a growing full application state for each change. Explicit versioning/restore policies make the storage tradeoff visible.

**Cons (assessment).** It supplies no immutable Git session history or author-intent record. Optional receive-side versioning is not an all-local-save backup. Conflict copies do not express a durable semantic selection. Adopting its retention schedule alone would be unsafe for Mesh, whose blobs can remain essential to unresolved sessions and conflict recovery.

Comparison and selected Mesh adaptations: [[Competitor storage approaches]].

