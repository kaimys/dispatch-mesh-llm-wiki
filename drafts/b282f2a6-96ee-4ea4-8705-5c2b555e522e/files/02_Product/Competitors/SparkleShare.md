---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://github.com/hbons/SparkleShare
  - https://github.com/hbons/SparkleShare/issues/2006
researched: 2026-11-05
storage_reviewed: 2026-11-06
storage_source_revision: 9b10379c7537967fc50b590a1ba30db5170b64ea
refresh: Before reuse in a product decision
---
# SparkleShare

A desktop folder-sharing application built around Git repositories, with a Dropbox-like experience and self-hosting. It is the closest general product precedent in the concept. [README](https://github.com/hbons/SparkleShare#readme)

## Links and GitHub activity

| Field | Snapshot on 2026-11-05 |
|---|---|
| GitHub repo | [hbons/SparkleShare](https://github.com/hbons/SparkleShare) |
| Homepage | [sparkleshare.org](https://sparkleshare.org/) — declared by the repository; homepage could not be retrieved during this research |
| GitHub stars | 4,998 |
| GitHub forks | 569 |
| Posts in GitHub Discussions | 0 returned by GraphQL; Discussions disabled |

Counts and enabled status: [REST metadata](https://api.github.com/repos/hbons/SparkleShare) and GraphQL. Issue conversations are not counted as Discussions.

## Feature list

- Automatically synchronizes project folders when files are added, edited, or removed.
- Uses Git and Git LFS, including self-hosted repositories.
- Desktop/status-icon workflow; Linux and macOS installation guidance and historical Windows support.

These describe the established application, not a verified replacement release. [README](https://github.com/hbons/SparkleShare#readme)

## What distinguishes it from Mesh

SparkleShare already established the folder-sync desktop workflow that Mesh still plans to ship. Mesh's implemented distinctions are separate draft transport, inactivity-based sessions, scoped recorded intent, and synchronized conflict inputs/selections. See [[Competitor overview#Mesh comparison baseline]].

Platform readiness needs care: the maintainer's 2024 retirement/build concerns were followed by a Rust port update in 2025, a 2026 rewrite notice, and an August 2026 funding announcement. The API reports the repository is **not archived**. Funding and a reboot are not evidence of a finished cross-platform release. [Maintainer status thread](https://github.com/hbons/SparkleShare/issues/2006), [repository metadata](https://api.github.com/repos/hbons/SparkleShare)

## What Dispatch could adapt

Recommendations: use familiar project/folder concepts, clear status, and guided host connection. Evaluate an explicit large-file backend later. Treat platform packaging and long-term maintenance as core product work. Monitor the funded reboot as both a learning source and a renewed competitor. See [[Dispatch Mesh SWOT analysis]].

## Storage architecture — source review (2026-11-06)

Inspected revision: [`9b10379c7537`](https://github.com/hbons/SparkleShare/commit/9b10379c7537967fc50b590a1ba30db5170b64ea). Static source inspection; no competitor was built or run. Existing feature claims and activity counts above retain their original research date.

**Scope.** This revision contains the established C# implementation. It does not establish the storage design or readiness of the announced rewrite.

**Repository and local layout.** Each shared folder is a Git working copy. The backend stages/commits local changes, fetches, and merges `FETCH_HEAD`. It persists local bookkeeping in the Git directory, including `.git/has_unsynced_changes`, `.git/info/size`, `.git/info/history_size`, and an LFS-behind marker. These local files are not shared branch content. [Source: Git repository backend](https://github.com/hbons/SparkleShare/blob/9b10379c7537967fc50b590a1ba30db5170b64ea/Sparkles/Git/Git.Repository.cs).

**Payloads and conflicts.** Ordinary mode stores file bytes in Git. The LFS setup path installs clean/smudge filters and local attributes (`.git/info/attributes`); LFS mode stores pointers in Git and transfers payloads through LFS. The fetcher recognizes its LFS convention and configures it explicitly. Conflict handling can materialize both sides as renamed files with identifying clues, stage them, and finish the merge; special metadata files have separate handling. [Source: fetcher/filter setup](https://github.com/hbons/SparkleShare/blob/9b10379c7537967fc50b590a1ba30db5170b64ea/Sparkles/Git/Git.Fetcher.cs), [conflict implementation](https://github.com/hbons/SparkleShare/blob/9b10379c7537967fc50b590a1ba30db5170b64ea/Sparkles/Git/Git.Repository.cs).

**Pros (assessment).** Normal Git repositories support familiar hosting and history. LFS provides a distinct payload path. Renamed conflict files can remain visible and recoverable while synchronization continues.

**Cons (assessment).** Git and LFS availability must both be considered; a Git clone alone does not prove every payload is locally available. Conflict copies alter the user's file namespace and do not encode Mesh-style retained input selections. The reviewed paths expose local flags and Git history, rather than a single transactional session/apply database. No equivalent application retention policy was established from these paths.

### Remote branch tree layout

This tree shows tracked entries at a remote branch tip, as seen in a repository file browser. The branch label is not a directory. Example user paths illustrate the layout; optional files appear only if committed. `.git/` is never a tracked subtree.

```mermaid
treeView-beta
	<shared remote branch>
		notes/
			example.md ## ordinary bytes, or an LFS pointer in LFS mode
		.sparkleshare ## project identifier when created/tracked
		<empty-directory>/
			.empty ## placeholder preserving an otherwise empty folder
		<renamed conflict files> ## competing versions when committed by resolution
```

LFS payload bytes are fetched separately; the tracked file at its original path contains the pointer in that mode. Local attributes, hooks, size caches and unsynced flags do not appear in this branch tree. Identifier/placeholder creation is in the fetcher and repository backend linked above.

### Local filesystem layout — not remote branch contents

Representative shared folder in the inspected C# implementation. LFS entries are conditional; normal mode keeps payloads in Git objects. Ref names below are logical, not a promise of loose files.

```mermaid
treeView-beta
	shared-folder/
		notes/example.md ## working bytes; Git blob or LFS-backed content
		<renamed conflict copies> ## materialized competing versions, when needed
		.git/
			objects/ ## ordinary Git history; LFS pointers in LFS mode
			refs/ ## local and remote-tracking refs
			index
			config
			has_unsynced_changes ## local backend flag
			info/
				size ## cached working-content size
				history_size ## cached Git-directory size
				attributes ## local filter rules, including LFS when enabled
			hooks/pre-push ## LFS hook when configured
			lfs/ ## conditional LFS local storage/state
				is_behind ## backend's pending-LFS-download marker
	Git remote → branch history ## files or LFS pointers
	LFS endpoint → payloads ## separate content path in LFS mode
```

Local flags and `.git/info/` files are not shared tracked files. Filter setup and these paths are defined in the repository backend/fetcher linked above; this tree does not describe the announced rewrite.

Comparison and selected Mesh adaptations: [[Competitor storage approaches]].

