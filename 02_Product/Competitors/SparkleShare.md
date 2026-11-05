---
type: competitor-profile
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[Dispatch Mesh concept]]"
  - https://github.com/hbons/SparkleShare
  - https://github.com/hbons/SparkleShare/issues/2006
researched: 2026-11-05
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
