---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://github.com/Vinzent03/obsidian-git
researched: 2026-11-05
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
