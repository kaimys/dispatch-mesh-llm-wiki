---
type: competitor-index
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - "[[gitwatch]]"
  - "[[SparkleShare]]"
  - "[[Obsidian Git]]"
  - "[[Agentic Git Sync]]"
  - "[[git-annex assistant]]"
  - "[[Syncthing]]"
  - "[[Jujutsu]]"
researched: 2026-11-05
refresh: Before reuse in a product decision
---
# Competitor overview

```dispatch
label: Update competitors and SWOT
intent: refresh-competitor-research
repo: dispatch-mesh
prompt: |-
  Refresh the competitor research in dispatch/wiki/{{file}} and the SWOT analysis in dispatch/wiki/02_Product/Dispatch Mesh SWOT analysis.md.

  Read AGENTS.md, dispatch/invariants.md, README.md, the Mesh concept, current requirements, milestone plan, and relevant ADRs. Read this overview, all linked competitor profiles in dispatch/wiki/02_Product/Competitors, and the existing SWOT before editing.

  Research every named competitor and architectural reference using current primary sources: official repositories, documentation, releases, and maintainer statements. Refresh each profile's brief description, GitHub repository, homepage, exact GitHub stars and forks, top-level GitHub Discussion thread count and enabled state, feature list, differences from Mesh, and concrete ideas Dispatch could adapt. Use live GitHub REST metadata and GraphQL discussions.totalCount; exclude replies, issues, and pull requests. Distinguish disabled Discussions, unavailable data, and true zero counts. Label unofficial mirrors and keep their metrics separate from canonical project metrics. Never invent unavailable figures or treat an unsuccessful lookup as zero.

  Update the overview's comparison table, Mesh baseline, qualifications, and source links from those profiles. Reassess the SWOT and recommended actions from the refreshed evidence. Distinguish implemented Mesh behavior, work in progress, and roadmap features; distinguish documented competitor claims from independently verified behavior and analyst inferences. Do not infer active usage or reliability from stars, forks, or discussion counts. Preserve useful context and this inline prompt.

  Stamp actual research dates, retain source links near factual claims, and record retrieval failures or unresolved uncertainties. Maintain owner, maintained_by, derived_from, and refresh metadata. Update the wiki index if the research set changes and append a dated entry to dispatch/wiki/00_Start-Here/log.md. Verify profile completeness, table/count consistency, and internal links. Finish with a concise summary of material changes and remaining evidence gaps. This is a documentation refresh; do not change engine code, create recurring automation, or publish anything.
```

Research snapshot: **2026-11-05**. Scope: the six open-source competitors/references in [[Dispatch Mesh concept]], plus Jujutsu, which the concept explicitly calls an architectural reference. Google Drive is a positioning analogy in the concept, not one of its named open-source references; it is outside this research set.

## Profiles and activity

| Profile | Role | Stars | Forks | GitHub Discussion posts |
|---|---|---:|---:|---|
| [[gitwatch]] | Minimal auto-commit baseline | 1,746 | 221 | 6 threads |
| [[SparkleShare]] | General Git-backed desktop sync | 4,998 | 569 | Disabled; API returns 0 |
| [[Obsidian Git]] | Direct wiki workflow alternative | 12,229 | 667 | 129 threads |
| [[Agentic Git Sync]] | Direct AI-assisted wiki alternative | 14 | 0 | Disabled; API returns 0 |
| [[git-annex assistant]] | Large-file sync and conflict reference | N/A canonical; 30 unofficial mirror | N/A canonical; 5 unofficial mirror | Canonical N/A; mirror disabled, API returns 0 |
| [[Syncthing]] | General sync substitute, without Git history | 90,113 | 5,578 | Disabled; API returns 0 |
| [[Jujutsu]] | Architectural reference, not unattended sync | 32,366 | 1,277 | 738 threads |

Each profile links its repository/API sources. The git-annex mirror is `con/git-annex`, not an official GitHub project or a project-wide popularity measure. Analysis: [[Dispatch Mesh SWOT analysis]].

## Mesh comparison baseline

The repository's `README.md`, read on 2026-11-05, is the implemented-behavior baseline. [[Dispatch Mesh requirements and architecture]] and [[Dispatch Mesh milestone plan]] describe the intended product; the concept includes historical alternatives and is not proof of implementation.

**Implemented console preview:** continuous draft exchange on device branches; multi-file sessions closed by five minutes without local edits; immutable permanent shared history; exact-path recorded requested purpose; optional configurable Chat Completions summaries with factual fallback; clean three-way text merging; synchronized conflict inputs and selections; SQLite transactions and recovery journals; JSON status including recent-save presence.

**Limits today:** Windows verification, Rust/Git prerequisites, externally configured Git authentication, default 16 MiB file limit, unsupported filesystem cases, no retention/pruning, experimental distributed conflict display, and no automatic local-history import from an existing checkout.

**Planned:** tray/status/conflict windows, guided account/repository setup, bundled Git, installer/login startup, macOS and Linux delivery, optional editor presence integration, and AI conflict proposals. Authenticated cloud MCP is deferred to 2.0. Do not score these as shipped strengths. Recent-save presence is not observation of unsaved typing.

## Research method and maintenance

- Primary sources only: project repositories, official documentation, and maintainer status statements. Profiles describe documented capabilities; no competitor installation, interoperability test, security audit, or performance benchmark was performed.
- Exact stars and forks came from live GitHub REST `GET /repos/{owner}/{repo}` using `stargazers_count` and `forks_count`. These are date-specific observations; they may differ from cached or rounded web pages.
- **Posts in GitHub Discussions means top-level discussion threads**, across all categories, using authenticated GitHub GraphQL `repository.discussions.totalCount`. Replies, comments, issues, and pull requests are excluded. `hasDiscussionsEnabled` was checked separately. Disabled is not evidence of an inactive community.
- Reproduction query (substitute the owner/name from each profile):

```graphql
query {
  repository(owner: "gitwatch", name: "gitwatch") {
    nameWithOwner
    hasDiscussionsEnabled
    discussions { totalCount }
  }
}
```

- Feature absence means “not established in the reviewed documentation,” unless explicitly contrasted with an implementation mechanism. No claim of market-wide uniqueness is made. Recommendations and SWOT conclusions are analyst inferences.
- **Maintenance register:** Kai owns this dated research set. Before its next use in a product decision, refresh all seven profiles' metrics and upstream status, then regenerate this overview and the SWOT together; record the refresh date and changes in [[log]]. This is a manually refreshed snapshot; no recurring automation was created or implied.

## Important research qualifications

- SparkleShare's old maintenance warning is followed by a Rust reboot and an August 2026 funding announcement in its [maintainer thread](https://github.com/hbons/SparkleShare/issues/2006). A completed reboot was not verified. Its declared homepage could not be retrieved.
- Obsidian Git now documents visual conflict resolution; characterizing it simply as stopping for manual Git repair would understate its current documented UX. [README](https://github.com/Vinzent03/obsidian-git#readme)
- Agentic Git Sync documents guided GitHub connection and AI conflict UX, but its small GitHub audience provides little adoption evidence. [README](https://github.com/leweii/agentic-git-sync#readme)
- git-annex's [official download page](https://git-annex.branchable.com/download/) identifies a non-GitHub canonical repository. Do not compare its unofficial mirror's counts with official project counts as adoption rankings.
