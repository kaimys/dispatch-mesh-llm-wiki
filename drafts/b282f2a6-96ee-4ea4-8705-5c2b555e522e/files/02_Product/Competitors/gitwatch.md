---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://github.com/gitwatch/gitwatch
researched: 2026-11-05
storage_reviewed: 2026-11-06
storage_source_revision: 04735cfa8973a5ba425ec4befb25ecb04074465c
refresh: Before reuse in a product decision
---
# gitwatch

A small Bash utility that watches a file or directory and automatically commits saved changes to an existing Git repository. It is a baseline alternative for people who primarily need automatic versioning. [Project README](https://github.com/gitwatch/gitwatch#readme)

## Links and GitHub activity

| Field | Snapshot on 2026-11-05 |
|---|---|
| GitHub repo | [gitwatch/gitwatch](https://github.com/gitwatch/gitwatch) |
| Homepage | [Project README](https://github.com/gitwatch/gitwatch#readme); no separate homepage in repository metadata |
| GitHub stars | 1,746 |
| GitHub forks | 221 |
| Posts in GitHub Discussions | [6 discussion threads](https://github.com/gitwatch/gitwatch/discussions); enabled |

Counts: [GitHub REST metadata](https://api.github.com/repos/gitwatch/gitwatch) and GraphQL `discussions.totalCount`. See [[Competitor overview#Research method and maintenance]].

## Feature list

- Recursive filesystem watching and a short settling delay before committing.
- Optional remote push and branch selection; optional pull/rebase before push.
- Configurable messages, exclusions, and startup capture/merge guards.
- Linux service, Docker, Nix, and macOS installation instructions; Git and watcher dependencies.

These are documented features, not independently tested here. [README](https://github.com/gitwatch/gitwatch#readme)

## What distinguishes it from Mesh

Its documented model creates ordinary commits after brief event batching. Mesh separates continuously shared drafts from permanent commits after a five-minute multi-file session. The reviewed documentation does not describe Mesh-style intent scoping or durable synchronized conflict selections. Mesh's extra state also makes its engine more complex. Mesh baseline: [[Competitor overview#Mesh comparison baseline]].

## What Dispatch could adapt

Recommendations: keep the unattended runner easy to deploy; provide straightforward startup and exclusion settings. Preserve the distinction between settling writes and ending an editing session rather than copying its short commit cadence. See [[Dispatch Mesh SWOT analysis]].

## Storage architecture — source review (2026-11-06)

Inspected revision: [`04735cfa8973`](https://github.com/gitwatch/gitwatch/commit/04735cfa8973a5ba425ec4befb25ecb04074465c). Static source inspection; no competitor was built or run. Existing feature claims and activity counts above retain their original research date.

**Repository and local layout.** The watched folder is a normal Git working tree. `perform_commit` stages changes and creates a normal commit; configured pull/rebase and push commands exchange that history. Files are ordinary Git blobs, trees and commits in the repository's Git directory. The reviewed script adds no separate draft branch, content store, session database or durable application journal. Watcher events and a settling timer batch writes; uncommitted saves remain in the working tree until capture. [Source: commit/watch loop](https://github.com/gitwatch/gitwatch/blob/04735cfa8973a5ba425ec4befb25ecb04074465c/gitwatch.sh).

**Recovery and retention.** Merge-state checks can defer automatic commits; optional pull uses rebase. Recovery and historical content therefore depend on ordinary Git commits and repository state. The script does not implement a policy for expiring captured file versions or a synchronized conflict-input registry.

**Pros (assessment).** Very small storage model, directly inspectable history, ordinary Git hosting, and few application-specific formats to migrate.

**Cons (assessment).** Event batches become permanent commits, so frequent saves can create noisy history. Uncaptured changes have no independent application checkpoint. Optional rebase and unresolved Git states need a recovery workflow; there is no equivalent of Mesh's continuously shared open session. This conclusion applies to the inspected script, not every possible wrapper or deployment.

### Remote branch tree layout

This tree shows tracked entries at a remote branch tip, as seen in a repository file browser. The branch label is not a directory. Example user paths illustrate the layout; optional files appear only if committed. `.git/` is never a tracked subtree.

```mermaid
treeView-beta
	<configured remote branch>
		notes/
			example.md ## user file, stored as an ordinary Git blob
		.gitignore ## optional tracked ignore rules
```

There is no gitwatch-specific branch directory schema in the reviewed script: the branch contains the captured user tree. It does not add Mesh-style `state.json` or `drafts/`.

### Local filesystem layout — not remote branch contents

Representative watched directory; `notes/example.md` is an example user file. Git internals are local; pushing transfers reachable objects and updates remote refs, not the literal `.git/` directory.

```mermaid
treeView-beta
	watched-folder/
		notes/example.md ## tracked user content → ordinary Git history
		.gitignore ## optional shared ignore rules
		.git/ ## local repository metadata
			objects/ ## blobs, trees, commits (loose or packed)
			refs/ ## logical branch/remote refs; may be packed
			index ## staged snapshot
			config ## local Git configuration/remotes
	remote Git repository
		<configured branch> ## normal commits of tracked files
```

The reviewed script adds no dedicated on-disk gitwatch state directory. Watcher/timer state lives in the running process; see the commit/watch-loop source above.

Comparison and selected Mesh adaptations: [[Competitor storage approaches]].

