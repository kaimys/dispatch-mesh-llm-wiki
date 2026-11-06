---
type: engineering-reference
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "[[gitwatch]]"
  - "[[SparkleShare]]"
  - "[[Obsidian Git]]"
  - "[[Agentic Git Sync]]"
  - "[[git-annex assistant]]"
  - "[[Syncthing]]"
  - "[[Jujutsu]]"
researched: 2026-11-06
refresh: Recheck pinned source paths before design adoption or a storage-related implementation plan.
---

# Competitor storage approaches

Static source review of all seven profiles in `02_Product/Competitors/`, dated 2026-11-06. Each profile now contains an implementation section with immutable source links, repository/local layout, recovery and retention behavior, and assessed pros and cons. No competitor was installed, built, benchmarked or exercised against user data. The conclusions concern the inspected code paths; they are not reliability rankings or exhaustive security audits.

Mesh baseline: [[Device branch format]] and [[SQLite database format]]. Mesh separates device draft snapshots from immutable shared session commits, stores exact content in Git and deduplicated SQLite blobs, and retains conflict inputs and recovery state. Current retention is indefinite. The three recommendations below are proposals, not accepted architecture changes or implemented features.

## Comparison

The profiles' **Remote branch tree layout** sections show tracked branch contents, matching the scope of Mesh's device-branch tree. Their separate **Local filesystem layout** sections show on-disk internals. `.git/` and `.jj/` metadata directories are not tracked branch subtrees; Syncthing has no Git branch at all. Git-annex's metadata branch and Jujutsu's conditional conflict-tree entries are explicitly distinguished from local storage.

| Project / model | What goes into Git or across peers | Local data beyond working files | Recovery and retention | Main tradeoff for Mesh |
|---|---|---|---|---|
| [[gitwatch]] — commit watched changes | Ordinary file blobs and commits; optional pull/rebase/push | Git objects, index and refs; watcher/timer process state | Ordinary Git history and merge-state checks; no separate application snapshot retention found in the script | Simple and portable, but short save batches become permanent commits. |
| [[SparkleShare]] — Git-backed shared folder | Ordinary commits and merges; LFS pointers in its LFS mode | Git directory, unsynced/size/LFS flags and filter configuration; LFS payload handling | Renamed competing versions and Git history | Familiar folder sync; LFS adds a second availability dependency and conflict copies alter file names. Reviewed C# implementation only. |
| [[Obsidian Git]] — editor-integrated Git | Ordinary commits through desktop or JavaScript Git backends | Git/index/merge state, plugin settings, device-local Obsidian storage | Git conflict handling and history; no Mesh-like separate draft journal found in reviewed paths | Strong inspectability; two backend paths and editor state add compatibility work. |
| [[Agentic Git Sync]] — agent-assisted Git | Ordinary branches/submodules plus portable `.github-sync.json` | Git data, plugin settings, external per-vault secret files, daily JSONL events | Git repair/backup tools; seven-day diagnostic log retention | Useful configuration boundaries; diagnostic logs are not recoverable content snapshots. |
| [[git-annex assistant]] — Git metadata plus external content | Tracked links/pointers; `git-annex` metadata branch and availability logs; payload transfer is separate | Key-addressed annex objects, metadata journal and SQLite key associations | Key-derived conflict variants and guarded content drop | Explicit placement/availability policy; considerably more machinery than native Git blobs. |
| [[Syncthing]] — direct peer synchronization | File metadata and blocks; no Git repository for synchronized user content | SQLite file/device/version/block indexes, temporary files, optional archived versions | Hash-checked receive/install, conflict copies, configurable version cleanup/restore | Efficient incremental indexing and transfer; no shared authored session history. |
| [[Jujutsu]] — Git objects plus operation history | Git-backed content/history; ordinary Git push does not carry the entire local operation store | Extra backend metadata, content-addressed protobuf views/operations, working-copy state under `.jj/` | Operation graph and keep refs, with garbage collection | Recoverable repository views; more formats and a developer workflow Mesh should not copy wholesale. |

Source evidence for each row is in its linked profile; exact revisions are registered below. Pros and cons are architectural assessments inferred from those implementations.

## What the approaches teach us

**Content, synchronization state and recovery history are different stores.** Gitwatch relies almost entirely on normal Git. SparkleShare and the Obsidian plugins add application bookkeeping around it. Git-annex separates payload availability from metadata; Syncthing separates file bytes from an indexed synchronization model; Jujutsu separately records operations and views. Mesh already distinguishes local state from transport, but its single current JSON row is not a chronological operation history.

**A reference does not prove recoverability.** An annex key or LFS pointer can exist without a local payload. Conversely, a Git object can exist locally but be lost later if no reachable reference keeps it. Jujutsu's keep refs and git-annex's guarded drop logic highlight separate requirements: preserve references to needed content, and prove availability before removing copies. Mesh currently protects remote referenced blobs through its device trees; proposed cleanup must preserve that invariant.

**Retention must name the artifact being expired.** Agentic Git Sync expires diagnostics, Syncthing thins optional archived file versions, and Jujutsu collects operation-store objects. These policies do not establish that Mesh may delete conflict evidence, pending capture, or unpublished sessions. Ordinary Git-backed tools also continue accumulating reachable committed content even if their local logs are bounded.

## Three adaptations for Mesh

### 1. Add an inspectable local operation history

**Learn from:** [[Jujutsu]]'s separate views and operation records; [[Agentic Git Sync]]'s diagnostic timeline is useful supporting evidence, but is not a substitute for recoverable state.

**Proposed Mesh design:** record meaningful durable transitions—capture checkpoint, incoming apply, session close, conflict selection, publication—with operation ID, parent/previous operation, affected session IDs, timestamp, factual reason, and references to recoverable before/after state. Keep content in existing blob storage. Write the operation record and state change in the same SQLite transaction where possible, and explicitly represent filesystem apply progress rather than claiming SQLite makes file installation atomic.

Start with read-only inspection and recovery preview. A restore should capture the recovered bytes as a new local edit/session; it must not rewind or amend a shared commit. Do not snapshot the entire state for every watcher event without measuring size and write cost.

**Benefit:** users and support can explain what happened between two states, including after a crash, and locate the exact recoverable versions.

**Cost / validation:** introduces schema migration and more retention roots. Test interrupted transitions, migrated workspaces, repeated recovery, and restore after peers have advanced. Define which operations remain recoverable versus merely logged before adding expiry.

### 2. Introduce retention based on reachability and recovery guarantees

**Learn from:** [[git-annex assistant]]'s guarded removal and availability records, [[Jujutsu]]'s explicit reachability roots, and [[Syncthing]]'s configurable archive thinning and restore interface.

**Proposed Mesh design:** first report storage by category and implement a read-only cleanup preview. Enumerate roots from active and waiting sessions, retained settled sessions, unresolved and retained resolved conflict inputs/selections, pending capture, apply journals, migration backups, and retained operation history. A reference's age alone must never make it eligible. Classify local historical bytes with no retained reference separately from recovery data required by the product contract.

Any later deletion needs a versioned policy, atomic coordination with capture, and a demonstrated restore path. Do not infer replica availability from a successful push alone. Local database reclamation and remote-history growth are separate problems: removing an entry from a future device tree does not remove older reachable Git commits. Reducing that remote history would need an explicitly agreed archival/protocol design; this proposal does not authorize rewriting shared history.

**Benefit:** makes growth understandable and creates a defensible path to bounded local storage.

**Cost / validation:** highest durability risk of the three. Test concurrent capture, interrupted cleanup, offline devices, unresolved conflicts, restored backups and schema upgrades. Start with accounting and dry-run reports; retain the current no-pruning behavior until guarantees are approved.

### 3. Separate shared policy from device configuration explicitly

**Learn from:** [[Agentic Git Sync]]'s portable repository configuration versus external secrets and [[Obsidian Git]]'s device-local settings.

**Proposed Mesh design:** define a small versioned shared-policy document and a distinct device configuration. Candidates for shared policy are supported ignore rules and workspace conventions. Device UUID, absolute paths, upload bookkeeping, transport-specific credentials and active recovery state remain local. Preserve current environment/credential-manager handling of secrets; copying the competitor's plaintext secret-file mechanism is unnecessary.

Import shared settings through validation and explicit precedence rules. Shared configuration must never silently enable AI text sharing, switch providers, or bypass the existing restriction on changing ignore rules for already synchronized paths. A peer-written policy document is input, not permission to execute commands or weaken local settings.

**Benefit:** new devices can inherit intended workspace conventions while keeping identity and consent independent.

**Cost / validation:** requires conflict/precedence rules and migration behavior. Test two devices with different paths and identities, malformed/unsupported policy versions, policy conflicts, tracked-file exclusion changes, and unchanged AI consent after a clone/import.

**Suggested order:** scope configuration boundaries with guided setup; prototype operation inspection as recovery work; deliver storage accounting early, but gate actual pruning on a separate retention design and adversarial recovery tests. These are engineering recommendations, not ticket state changes.

## Source register and limitations

| Profile | Inspected commit | Provenance |
|---|---|---|
| [[gitwatch]] | [`04735cfa8973`](https://github.com/gitwatch/gitwatch/commit/04735cfa8973a5ba425ec4befb25ecb04074465c) | Fetched default branch snapshot |
| [[SparkleShare]] | [`9b10379c7537`](https://github.com/hbons/SparkleShare/commit/9b10379c7537967fc50b590a1ba30db5170b64ea) | Fetched default branch snapshot |
| [[Obsidian Git]] | [`8d1144aef2d4`](https://github.com/Vinzent03/obsidian-git/commit/8d1144aef2d41ffca6245a60199656e7a0c5ab17) | Fetched default branch snapshot |
| [[Agentic Git Sync]] | [`2a82f02dbbdd`](https://github.com/leweii/agentic-git-sync/commit/2a82f02dbbdd118d21422b657b3230cf6dfdb20e) | Fetched default branch snapshot |
| [[git-annex assistant]] | [`de5ced7ec6db`](https://github.com/con/git-annex/commit/de5ced7ec6db8f4316a0090f37f006ac179f2c07) | Unofficial mirror, `upstream/master` |
| [[Syncthing]] | [`b34734376128`](https://github.com/syncthing/syncthing/commit/b34734376128492fa4e1b01412193de033e20613) | Fetched default branch snapshot |
| [[Jujutsu]] | [`2f76430997be`](https://github.com/jj-vcs/jj/commit/2f76430997be91fa1557ca7f5fa323e3ed004dcc) | Fetched default branch snapshot |

Source checkouts/fetched objects are retained locally under the source repository's ignored `target/competitor-source-review/` for review. They are evidence caches, not product dependencies. git-annex's canonical source browser was unavailable; its upstream snapshot was read from the explicitly unofficial mirror. Windows long-path failures affected its documentation checkout, so relevant source was read using `git show` from fetched objects. No failed checkout was treated as an executable/test result.

Source files were followed through their storage and synchronization paths; no claim of comprehensive coverage is made. The existing profiles' popularity counts and other dated feature claims were not refreshed. Some defaults can differ across releases and configuration; use the pinned sources rather than assuming these descriptions apply to every installed version.

Maintenance owner: [[Kai]]. Refresh this overview and the corresponding profile sections together before adopting a design, and when a cited storage implementation changes. This manual refresh is registered in [[index]]; no recurring automation was created. Related Mesh constraints: [[ADR-0001 — Synchronize drafts separately from permanent history]], [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]], [[ADR-0007 — Persist state transactionally with recovery journals]], and [[ADR-0008 — Store content as blobs referenced by metadata]].

