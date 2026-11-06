---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://github.com/Vinzent03/obsidian-git
researched: 2026-11-05
storage_reviewed: 2026-11-06
storage_source_revision: 8d1144aef2d41ffca6245a60199656e7a0c5ab17
refresh: Before reuse in a product decision
---
# Obsidian Git

A community plugin that brings Git versioning and automated synchronization into Obsidian. It directly overlaps with Mesh's shared-wiki use case. [README](https://github.com/Vinzent03/obsidian-git#readme)

## Links and GitHub activity

| Field | Snapshot on 2026-11-05 |
|---|---|
| GitHub repo | [Vinzent03/obsidian-git](https://github.com/Vinzent03/obsidian-git) |
| Homepage | [Official documentation](https://publish.obsidian.md/git-doc/) |
| GitHub stars | 12,229 |
| GitHub forks | 667 |
| Posts in GitHub Discussions | [129 discussion threads](https://github.com/Vinzent03/obsidian-git/discussions); enabled |

Counts: [REST metadata](https://api.github.com/repos/Vinzent03/obsidian-git) and GraphQL. See [[Competitor overview#Research method and maintenance]].

## Feature list

- Scheduled commit/pull/push and startup pull.
- Source control, history, and side-by-side diff views; selective staging.
- In-editor conflict actions to retain either side, both sides, or the ancestor when available.
- Desktop line-change indicators and optional submodules.
- Mobile implementation, explicitly described as highly unstable by the project.

Documented capabilities only. The current README has richer conflict handling than the concept's brief description suggests. [README](https://github.com/Vinzent03/obsidian-git#readme)

## What distinguishes it from Mesh

The plugin integrates directly with Obsidian and exposes normal Git controls. Mesh runs independently of the editor and shares open-session drafts separately from finalized history. Mesh currently lacks the plugin's visual history/diff/conflict interface. The reviewed plugin documentation does not establish Mesh-equivalent scoped agent intent. Baseline: [[Competitor overview#Mesh comparison baseline]].

## What Dispatch could adapt

Recommendations: prioritize understandable file history, diffs, and conflict actions. An optional Obsidian adapter could expose Mesh status and explicit active-editing signals, while preserving the independent engine. Test platform-specific behavior and label limitations as clearly as this plugin does. See [[Dispatch Mesh SWOT analysis]].

## Storage architecture — source review (2026-11-06)

Inspected revision: [`8d1144aef2d4`](https://github.com/Vinzent03/obsidian-git/commit/8d1144aef2d41ffca6245a60199656e7a0c5ab17). Static source inspection; no competitor was built or run. Existing feature claims and activity counts above retain their original research date.

**Repository and local layout.** The vault/repository uses ordinary tracked files, Git objects, refs and an index. Desktop operations run through `simple-git`, including staging, commits and configured merge/rebase integration. A separate `isomorphic-git` implementation uses the Obsidian filesystem adapter and implements Git merge-state files such as `MERGE_HEAD` and `MERGE_MSG`. These are two implementations of the Git workflow, not Mesh-style device manifests. [Sources: desktop backend](https://github.com/Vinzent03/obsidian-git/blob/8d1144aef2d41ffca6245a60199656e7a0c5ab17/src/gitManager/simpleGit.ts), [JavaScript backend](https://github.com/Vinzent03/obsidian-git/blob/8d1144aef2d41ffca6245a60199656e7a0c5ab17/src/gitManager/isomorphicGit.ts).

**Application state.** Device-specific values such as hostname, Git executable path and last automatic operation times use Obsidian local storage. Plugin settings are also loaded/saved through Obsidian's plugin-data API. Thus the Git history and editor/plugin state have different persistence paths; a repository copy alone is not the complete device setup. [Sources: local settings](https://github.com/Vinzent03/obsidian-git/blob/8d1144aef2d41ffca6245a60199656e7a0c5ab17/src/setting/localStorageSettings.ts), [plugin lifecycle](https://github.com/Vinzent03/obsidian-git/blob/8d1144aef2d41ffca6245a60199656e7a0c5ab17/src/main.ts).

**Recovery and retention.** Pending conflicts use Git merge/index state and working-file resolution. Committed versions remain Git history; the reviewed backend does not add a separate durable all-save archive or Mesh-like conflict manifest. Retention is consequently tied to Git history and local Git recovery facilities rather than a plugin content-pruning policy.

**Pros (assessment).** Direct history/diff/staging access, ordinary Git interoperability, and separation of several device preferences from shared content.

**Cons (assessment).** Frequent automatic commits can accumulate permanent history. Desktop and JavaScript backends require separate compatibility testing. Plugin/local-storage state does not provide Mesh's transactionally linked capture, apply-recovery and conflict-input records.

Comparison and selected Mesh adaptations: [[Competitor storage approaches]].

