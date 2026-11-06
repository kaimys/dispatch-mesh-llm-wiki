---
type: product-discovery
status: exploration
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
created: 2026-11-06
updated: 2026-11-06
derived_from:
  - "Kai's discussion in this chat on 2026-11-06"
  - "[[Competitor storage approaches]]"
  - "[[AI-managed synchronization and human decisions]]"
  - "[[Device branch format]]"
  - "[[SQLite database format]]"
refresh: Review before refinement or an architecture decision; verify Git feature compatibility during prototyping.
---

# Classical Git workflow for Mesh

## Product opportunity and status

Kai's proposal is to make greater use of the classical Git workflow: shared branches contain ordinary user files, Git performs mechanical integration, and Mesh hides its operations from end users. AI can help propose text when a technical conflict prevents a clean merge; a human approves the exact proposed result. Developers and support staff can inspect ordinary branch contents, diffs and history using existing tools.

This discovery evaluates that direction, including disadvantages and possible remedies. It is not a decision to replace the current protocol or support arbitrary manual Git operations. [[ADR-0001 — Synchronize drafts separately from permanent history]], [[ADR-0002 — Group files into sessions with an inactivity boundary]], [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]] and the other accepted ADRs remain binding. A change requires an explicit design and compatibility plan.

The central question is **whether ordinary Git trees on device branches can make Mesh easier to understand and maintain while preserving continuous drafts, meaningful sessions, recoverable conflicts and immutable shared history.**

The AI feature in [[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]] is independent: human-approved proposals can be added to either the current transport or a future classical Git workflow. Neither change is a prerequisite for the other by definition.

## What “classical Git” means here

Three choices should be considered separately:

1. **Representation:** branch trees contain workspace files at their normal paths.
2. **History/publication:** when checkpoints become commits, how branches integrate, and what reaches shared permanent history.
3. **Operation ownership:** Mesh executes Git for the user; optional inspection through other tools does not automatically authorize those tools to modify Mesh-managed state.

Mesh already uses native Git blobs and ordinary commits internally. The proposed representation change is from a device branch containing a manifest plus session-specific folders to a device branch whose root is a normal workspace snapshot. It is not replacing a non-Git store with Git for the first time.

## Remote branch trees

Illustrative current device branch:

```text
mesh/devices/<device-id>              # branch label, not a directory
├── state.json                       # sessions and conflict metadata
├── drafts/<session-id>/
│   ├── base/<workspace-path>        # before versions
│   └── files/<workspace-path>       # session result versions
├── conflicts/<conflict-id>/...
└── resolutions/<conflict-id>/...
```

Candidate ordinary-tree device branch:

```text
<device draft branch>                # exact namespace to be decided
├── notes/
│   └── onboarding.md               # current draft bytes at the normal path
├── assets/
│   └── logo.png
└── .meshignore                      # only when tracked
```

Shared finalized branch:

```text
main                                # or the configured shared branch
├── notes/
│   └── onboarding.md               # finalized integrated bytes
├── assets/
│   └── logo.png
└── .meshignore                      # only when tracked
```

These trees show **tracked remote branch contents**. Neither `.git/` nor Mesh's local SQLite database is checked in as a subtree. In the candidate, successive checkpoint commits retain earlier file versions. A branch with a conflicting version can still be an ordinary, internally consistent snapshot; it need not contain conflict markers.

Session IDs, approval records and pending conflicts still need representation. They might use commit metadata plus explicitly synchronized auxiliary metadata, rather than `drafts/` directories. That placement is an open design question, not a claim that file trees alone contain every Mesh concept.

## Workflow alternatives

| Option | Workflow | Benefit | Remaining cost / compatibility |
|---|---|---|---|
| A — one shared stream | Capture batches, commit, fetch/integrate, push directly to the shared branch. | Smallest conceptual model; browser shows newest successful integration. | Rapid sharing means many permanent commits; waiting for session completion delays sharing. Conflicts still need preserved local/remote inputs. This conflicts with the current separation of drafts and permanent sessions. |
| B — device branches and normal merges | Append checkpoints to device branches; integrate them into shared history with ordinary ancestry-preserving merges. | Normal file trees and merge graphs; Git ancestry records what was incorporated. | Checkpoint ancestors remain reachable from shared history even if the UI emphasizes one session merge. A first-parent view can reduce visual noise but does not remove those commits. Session/history expectations need explicit review. |
| C — device branches and session-level publication | Share ordinary checkpoint trees; publish each finalized session as a new integrated session commit on the shared branch, keeping checkpoint history separately. | Closest candidate to current continuous-draft and concise-final-history behavior; normal paths on device branches. | Requires explicit mappings from checkpoints/sessions to published results and careful subsequent integration. This retains a Mesh protocol around Git. |

Squash-style publication is one possible mechanism for C: Git can prepare the combined result without recording the second-parent merge relationship. That means a later merge cannot rely on that missing ancestry to know the session was already incorporated; the application must account for it. This is an engineering tradeoff, not a command that automatically solves session publication. [Git merge: squash](https://git-scm.com/docs/git-merge#Documentation/git-merge.txt---squash)

**Candidate to investigate first:** C, because it retains the product's existing session objective. Compare it experimentally with B rather than treating concise history as free. A is a useful baseline, but adopting it would change the current product promise.

## Advantages

| Advantage | What users or maintainers gain | Qualification |
|---|---|---|
| Readable branch contents | A file browser opens `notes/onboarding.md` directly rather than navigating session folders. | The selected device branch still represents a draft view; it may differ from finalized `main` or another device's view. |
| Existing visualization | Ordinary diffs, history graphs, blame and comparisons work on user paths. | Generic tools do not understand pending approvals, scoped intent or Mesh session dependencies without additional conventions. |
| More standard mechanics | Git supplies objects, ancestry, merge bases and mechanical merging. | Mesh still owns scheduling, attribution, filesystem installation, durable state and user interactions. |
| Easier investigation and export | Support can inspect branch commits and extract file versions with standard tools. | A repository clone is not necessarily a full backup of unpublished local saves or local-only decision state. |
| Potentially smaller custom transport | The current before/after directory projection might be replaced by commit/tree references. | Reduced code and improved performance are hypotheses to measure, not established outcomes. |
| AI assistance with a familiar model | AI can explain changes and propose a combined text from preserved versions. | Neither AI nor a normal tree layout replaces human approval of AI-written resolutions. |

The source-reviewed competitors [[gitwatch]], [[Obsidian Git]] and [[Agentic Git Sync]] demonstrate variants of ordinary Git automation. [[Competitor storage approaches]] contains their pinned implementation evidence; their existence does not establish that every Mesh invariant is already handled.

## Disadvantages and how to address them

### 1. Frequent sharing can produce noisy permanent history

**Problem:** a save that must travel through Git needs a committed representation somewhere. Sending every small batch directly to shared history makes technical checkpoints permanent.

**Possible remedies:** put checkpoints on device/session branches, then either retain them through normal merges (B) or publish a separate session result (C). Preserve the five-minute session boundary independently of a short write-settling timer.

**Residual tradeoff:** B keeps detailed ancestry; C keeps concise shared history but requires more application bookkeeping. Hiding commits in a UI is not deleting them. Force-pushing shared history to tidy it is excluded.

### 2. A conflict can leave integration unfinished

**Problem:** Git can merge clean regions while retaining unresolved paths in its index. That is useful, but an unmerged index is not itself a normal branch commit ready to share. Existing competing commits remain available; an application must decide what to install and publish while the conflict is pending. [Git merge](https://git-scm.com/docs/git-merge)

**Possible remedies:** prepare integration separately, preserve the competing commits, retain the accepted visible version for unresolved files, and install independently safe clean changes. Persist a pending-conflict record so restart and other devices know what remains unresolved.

**Residual tradeoff:** publishing only some changes is an explicit partial-integration design. A commit with both original parents marks their histories as merged even if some content was deliberately omitted. Do not create such a commit merely to bypass the unresolved index and expect Git to rediscover omitted changes later. Record the remaining inputs/dependencies and define their later publication. Dependencies between apparently separate files also need consideration.

This is a reason to prototype the protocol carefully, not a claim that Git cannot keep unrelated work moving.

### 3. Users keep saving during integration

**Problem:** incoming changes and local saves can race. Automatically stashing and popping does not by itself define which version may safely replace the live file.

**Possible remedies:** checkpoint before planning; prepare integration in an isolated worktree/index; compare the current live version before installation; journal the installation and preserve displaced content. An outdated candidate is recomputed rather than overwriting a new save.

Git worktrees provide separate working-tree/index state, but filesystem application remains Mesh's responsibility. [Git worktree](https://git-scm.com/docs/git-worktree)

**Residual tradeoff:** isolation adds local objects/workspaces and lifecycle management. A main-process failure must not leave a temporary integration workspace as the only recoverable copy.

### 4. Mixing incoming drafts into snapshots can blur authorship

**Problem:** a device's visible tree may include another person's draft. Committing the whole visible state as the local author's new work can claim that other person's contribution.

**Possible remedies:** preserve original commits and provenance, compute local changes against the last installed baseline, and distinguish an integration commit's committer from its contributing authors. Retain stable session/intent associations across transport checkpoints and final publication.

**Residual tradeoff:** ordinary Git trees alone do not encode all those distinctions. B can retain ancestry; C needs explicit publication mappings. Test multi-file dependencies and later edits based on received drafts before choosing the design. AI summaries must not be used to infer authorship.

### 5. A merged result does not fully describe the human decision

**Problem:** commit parents and resulting bytes document a merge, but do not by themselves supply Mesh's exact candidate approval, approver, input revisions and pending-review state.

**Possible remedies:** retain a small durable decision record binding source object IDs, candidate hash, approval and publication result. Transport those records explicitly if devices must share them. A dedicated metadata ref, defined commit trailers or Git notes are possible building blocks, not interchangeable finished protocols.

Git notes attach metadata without changing the annotated commit and use their own refs; selecting notes requires explicit distribution and merge conventions. [Git notes](https://git-scm.com/docs/git-notes)

Git's `rerere` can reuse recorded resolutions of recurring conflict patterns. It is useful supporting machinery, not proof that a human approved this exact newly generated candidate or a replacement for cross-device decision provenance. [Git rerere](https://git-scm.com/docs/git-rerere)

**Residual tradeoff:** standard file trees improve interoperability, but some Mesh-specific metadata remains. Generic Git clients may ignore it.

### 6. Semantic text merging is helpful but not always correct

**Problem:** Git may report a technical conflict between compatible intentions. AI can propose new wording but can also omit a qualification or invent an interpretation.

**Possible remedy:** follow [[Story - US00004 - AI-proposed text conflict merges with human approval|US00004]]: show the proposed text with the source edits and require human approval of the exact result. A user may edit-and-confirm, choose an existing version or defer. Bind approval to unchanged inputs.

**Residual tradeoff:** review takes user attention. Measure acceptance and editing effort; do not replace approval with a confidence threshold. General consistency checking on clean commits remains outside this workflow.

### 7. Local failures and API outages still require recovery

**Problem:** Git preserves committed versions, but unsaved editor buffers, uncaptured filesystem saves, and partially installed updates are different states. An unavailable AI service cannot be the only recovery mechanism.

**Possible remedies:** retain local capture checkpoints and an apply journal; preserve original inputs and keep recovery operations deterministic. Queue AI requests with bounded retries and allow manual choices. Display “saved locally,” “draft shared” and “waiting for your review” according to observed facts.

**Residual tradeoff:** SQLite or equivalent local state likely remains. Classical remote trees do not imply a stateless client. No provider switching or broader text sharing is permitted as an outage workaround.

### 8. Rejected pushes and external operations can interrupt automation

**Problem:** another device may advance a branch, while other failures indicate authentication trouble or unexpected history changes.

**Possible remedies:** classify ordinary publication races, fetch and recompute against the new head, and retry without force-pushing shared history. Distinguish inaccessible remotes from rewritten/deleted history. Keep advanced diagnostics available while explaining user actions as “Reconnect” or “Review recovery.”

**Residual tradeoff:** inspectability through third-party tools does not make concurrent manual mutation safe. Supporting external editors that manipulate Git is a separate compatibility commitment under ADR-0006.

### 9. Ordinary Git history still grows

**Problem:** checkpoint commits and changing binary files retain reachable content. Squashing into a new shared commit does not remove device-branch ancestors.

**Possible remedies:** measure object/metadata size and fetch cost; define retention roots and a recoverable archive policy before any cleanup. Consider separate payload storage only as a future large-file design with explicit availability guarantees.

Git garbage collection manages objects according to reachability and retention rules; it is not an application policy that understands which pending session or approval matters to Mesh. [Git GC](https://git-scm.com/docs/git-gc)

**Residual tradeoff:** preserving all reachable checkpoint history and bounding historical storage indefinitely are competing goals. Removing a local cache does not remove copies retained by remote branches or other clients.

## Concrete end-to-end example

1. Alice and Bob start from the same finalized guide. Each saves overlapping wording changes on their own device.
2. Mesh records ordinary checkpoint commits on their device branches. Both versions can be browsed as `guide.md`, without `drafts/<id>/files/`.
3. Integration detects a text conflict in `guide.md`. Bob also fixed an unrelated spelling error in `faq.md`.
4. Mesh retains the guide inputs and accepted visible guide. The independent FAQ change can reach the working view where safe; any permanent partial publication follows an explicit protocol rather than pretending the entire guide conflict was resolved.
5. With consent, AI proposes a guide combining Alice's clarification and Bob's new instruction. The user sees the proposed wording and approves it, not a Git command.
6. The engine checks that the approved candidate and inputs are unchanged, records the decision and applies recoverably.
7. Session publication follows B or C. A race causes fetch/recomputation; it does not authorize overriding new saves or applying an old approval to new inputs.
8. Other devices receive the result and decision evidence. Later distinct edits remain new work rather than being suppressed by the earlier choice.

The representation is familiar Git; the experience is a managed document workflow. The hard questions are explicit preservation, attribution and publication rules, not the user's ability to run Git.

## What AI would and would not do

| Responsibility | Initial owner |
|---|---|
| Routine capture, fetch, clean merge, push/retry | Deterministic engine |
| Interpret a technical text conflict and propose meaning-preserving wording | AI with consent |
| Approve exact AI-written text or choose between incompatible intentions | Human |
| Verify freshness, persist decisions, apply safely, recover interruption | Deterministic engine |
| Explain unusual technical events in user language | Templates first; AI may assist |
| Review all document content for consistency | Outside this discovery's initial workflow |

This split can operate during API outages: local saving and deterministic integration continue; AI-assisted conflict proposals wait or are handled manually.

## Architecture questions and decision criteria

Before selecting B or C, resolve:

- Does “one permanent commit per session” mean literal shared history, or would users accept detailed checkpoint ancestry behind a session-oriented view?
- Does a device branch represent only local authored work, or the complete integrated view? How are peer contributions kept distinguishable in either case?
- How are dependency-bearing sessions and partial integration represented without applying or publishing changes twice?
- Where do stable session identities, scoped intent, pending conflicts and exact approvals live, and how are those records fetched and validated?
- Can a fresh client reconstruct the intended state from the remote alone after a device disappears?
- Which Git minimum version/backend is supported? Modern `merge-tree --write-tree` can prepare merge results without modifying an index or working tree, but capabilities must be checked against Mesh's supported Git versions. Its conflicted output must not be installed as though it were clean. [Git merge-tree](https://git-scm.com/docs/git-merge-tree)
- Which third-party activities are supported: reading history, exporting files, or modifying branches? Begin with explicit read-only interoperability.
- How will older clients distinguish old manifest branches from ordinary-tree branches? Do not silently replace the contents of existing `mesh/devices/*` refs while old clients still decode them as manifests.
- How can migration preserve all unpublished edits, conflict inputs, intent and pending approvals without rewriting shared history?

A prototype should compare current Mesh, B and C on the same disposable fixtures. Require preservation and convergence before judging convenience. Check continuous saves during fetch/integration, conflicting plus unrelated paths, restart at each durable boundary, opposite publication orders, received-draft attribution, stale/edited approvals, empty effective sessions, offline resumption and fresh-client reconstruction.

Then measure history readability in Git tools, storage growth, synchronization latency, code/metadata complexity and time needed to review AI proposals. A smaller tree layout alone is not evidence of a simpler complete system.

## Proposed next step and boundaries

Run a bounded design/prototype investigation of **ordinary device file trees with the existing session model**, comparing B and C. Start from the current behavior as the baseline. Document which custom transport/state responsibilities disappear, which move, and which remain.

Do not adopt ordinary shared-stream commits merely because the command sequence is shorter, and do not reject ordinary trees merely because orchestration is required. The decision should turn on the preservation of product behavior and a measured reduction in complexity or improvement in interoperability.

This document creates no implementation ticket and does not expand US00004. Any selected architecture needs an ADR review, versioned protocol/migration plan and separate implementation scope. It does not authorize live remote tests, deletion, force-pushes or changes to the user's wiki transport.

## Sources and maintenance

Discussion source: Kai's 2026-11-06 comments about classical Git interoperability, existing merge capabilities and human-approved AI text proposals. Competitor implementation evidence is pinned in [[Competitor storage approaches]] and linked profiles.

Current Mesh behavior: [[Device branch format]], [[SQLite database format]], [[Dispatch Mesh requirements and architecture]], [[Dispatch Mesh milestone plan]], ADR-0001/0002/0003/0006, and [[ADR-0008 — Store content as blobs referenced by metadata]]. Related discovery: [[AI-managed synchronization and human decisions]].

Git documentation was checked on 2026-11-06; links above support the mechanics, while architecture alternatives and mitigations are proposals/inferences. No prototype or runtime benchmark was performed.

Owner and maintenance: [[Kai]]. Refresh before refinement or an architecture decision, and check feature availability against the supported Git versions during prototyping. Registered in [[index]]; no recurring automation configured.

