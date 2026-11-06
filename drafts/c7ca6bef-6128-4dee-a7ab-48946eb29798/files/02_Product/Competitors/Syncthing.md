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
