---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://github.com/leweii/agentic-git-sync
researched: 2026-11-05
storage_reviewed: 2026-11-06
storage_source_revision: 2a82f02dbbdd118d21422b657b3230cf6dfdb20e
refresh: Before reuse in a product decision
---
# Agentic Git Sync

An Obsidian plugin aimed at people unfamiliar with Git, using an LLM agent for Git operations, messages, conflicts, and error diagnosis. It has the closest AI-oriented feature overlap. [README](https://github.com/leweii/agentic-git-sync#readme)

## Links and GitHub activity

| Field | Snapshot on 2026-11-05 |
|---|---|
| GitHub repo | [leweii/agentic-git-sync](https://github.com/leweii/agentic-git-sync) |
| Homepage | [Project README](https://github.com/leweii/agentic-git-sync#readme); no separate homepage declared |
| GitHub stars | 14 |
| GitHub forks | 0 |
| Posts in GitHub Discussions | 0 returned by GraphQL; Discussions disabled |

Counts: [REST metadata](https://api.github.com/repos/leweii/agentic-git-sync) and GraphQL. These measure repository attention, not reliability.

## Feature list

- Timed background pull/push and AI-drafted commit messages.
- AI conflict resolution with a Local / Remote / AI Suggestion resolver, reasoning, and confidence display.
- Git error diagnosis and empty-repository initialization.
- Setup wizard with GitHub App or token authentication.
- Submodules for team content alongside personal notes; synchronized non-secret configuration.

All are project claims, not independently verified behavior. [README](https://github.com/leweii/agentic-git-sync#readme)

## What distinguishes it from Mesh

Its agent operates Git inside Obsidian; Mesh uses a deterministic sync engine with optional AI summaries and factual fallback. Mesh already records path-scoped requested purpose independently of AI. AI conflict proposals and guided setup remain future Mesh features. Baseline: [[Competitor overview#Mesh comparison baseline]].

## What Dispatch could adapt

Recommendations: study the onboarding and side-by-side proposal UI. Show original versions and clearly label proposals; confidence is not validation. Keep AI advisory and opt-in, retain inputs, and bind summaries to frozen content. Consider separating portable configuration from device secrets. See [[Dispatch Mesh SWOT analysis]].

## Storage architecture — source review (2026-11-06)

Inspected revision: [`2a82f02dbbdd`](https://github.com/leweii/agentic-git-sync/commit/2a82f02dbbdd118d21422b657b3230cf6dfdb20e). Static source inspection; no competitor was built or run. Existing feature claims and activity counts above retain their original research date.

**Repository and local layout.** The vault and optional submodules are ordinary Git repositories. `GitManager` uses `simple-git`; its pull path requests merge rather than rebase, and pushes publish the configured branch. Agent tools operate that repository rather than a separate synchronized draft store. [Source: GitManager](https://github.com/leweii/agentic-git-sync/blob/2a82f02dbbdd118d21422b657b3230cf6dfdb20e/src/git/GitManager.ts).

**Three configuration scopes.** The versioned `.github-sync.json` carries structural settings such as remote, branch, sync policy, provider/model descriptions and submodules. Non-secret plugin state uses Obsidian plugin data. Credentials are written outside the vault under the OS user-configuration directory, in a per-vault JSON file named from a hash of its absolute path. That is separation from Git synchronization, not evidence of encrypted-at-rest secret storage. The secret-store implementation supersedes an outdated comment in `RepoConfig.ts` claiming secrets remain in plugin data. [Sources: portable schema](https://github.com/leweii/agentic-git-sync/blob/2a82f02dbbdd118d21422b657b3230cf6dfdb20e/src/config/RepoConfig.ts), [configuration I/O](https://github.com/leweii/agentic-git-sync/blob/2a82f02dbbdd118d21422b657b3230cf6dfdb20e/src/config/repoConfigIO.ts), [secret store](https://github.com/leweii/agentic-git-sync/blob/2a82f02dbbdd118d21422b657b3230cf6dfdb20e/src/config/secretStore.ts).

**Recovery and retention.** Diagnostic events go to daily JSONL files beneath the plugin's `event-log/` directory with seven-day cleanup. This is an observability log, not a transactional content snapshot. Recovery tools include Git commands and a repository-rebuild path with backup requirements; these are materially broader mutations than Mesh permits. [Sources: event log](https://github.com/leweii/agentic-git-sync/blob/2a82f02dbbdd118d21422b657b3230cf6dfdb20e/src/observability/EventLog.ts), [recovery tools](https://github.com/leweii/agentic-git-sync/blob/2a82f02dbbdd118d21422b657b3230cf6dfdb20e/src/git/recoveryTools.ts).

**Pros (assessment).** Explicit shared/local/secret boundaries help a newly cloned vault acquire useful configuration without inheriting credentials. Event logs aid diagnosis.

**Cons (assessment).** Multiple stores require coordinated migration and backup; path-keyed secrets need care when moving a vault. Event retention is not file retention. Agent-driven repair introduces a wider safety surface; copying those mutations would conflict with Mesh's immutable-history and recovery guarantees.

### Remote branch tree layout

This tree shows tracked entries at a remote branch tip, as seen in a repository file browser. The branch label is not a directory. Example user paths illustrate the layout; optional files appear only if committed. `.git/` is never a tracked subtree.

```mermaid
treeView-beta
	<configured vault branch>
		notes/
			example.md          ## ordinary committed user file
		.github-sync.json       ## portable config, when written and committed
		.gitmodules             ## when submodules are configured/committed
		<submodule-path>        ## gitlink: commit ID in another repository
```
A submodule entry records a commit reference; it does not embed that submodule's file tree in the parent branch. Its files belong to the submodule repository. Plugin `data.json` and event logs can also appear if tracked, but are not a required branch schema. External secret files and Git metadata are not part of this tree.

### Local filesystem layout — not remote branch contents

Representative desktop vault layout. `<config-dir>` is normally `.obsidian`; plugin data/logs lie inside the vault, so their synchronization depends on ignore/tracking policy.

```mermaid
treeView-beta
	vault/
		notes/example.md ## ordinary tracked content
		.github-sync.json ## portable versioned configuration
		.git/ ## Git objects, refs, index, config
		.gitmodules ## conditional: submodule definitions
		<submodule>/ ## conditional: separate Git history
		<config-dir>/plugins/agentic-git-sync/
			data.json ## non-secret plugin/device state
			event-log/
				YYYY-MM-DD.jsonl ## diagnostic events; seven-day cleanup
	<OS user-config>/agentic-git-sync/ ## outside the vault
		vault-<16-hex-path-hash>.json ## per-vault secrets
	Git remote(s) → configured branches ## vault and optional submodule commits
```

The user-config root is platform-specific: Windows uses `%APPDATA%`, macOS uses `~/Library/Application Support/`, and Linux uses `$XDG_CONFIG_HOME` or `~/.config/`. These are storage locations, not a claim of encryption or immunity from other OS backup/sync software. Sources are the configuration I/O, secret store and event-log modules linked above.

Comparison and selected Mesh adaptations: [[Competitor storage approaches]].

