---
id: ADR-0013
status: proposed
date: 2026-11-06
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-06
---

# ADR-0013 — Derive exclusion from each tree's own ignore policy

## Context

Until 0.2.5, `.gitignore` and `.meshignore` applied only to files that had never been synchronized. A rule matching an already synchronized file stopped capture, and an incoming path that the local policy ignored paused synchronization. Obsidian's per-device `.obsidian/workspace.json` therefore couldn't be excluded once it had been shared. [[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]] decided that the shared `.meshignore` stops synchronization of such a file. `main` records its removal, every device keeps its local copy, and a peer's late edit loses to the removal.

Mesh computes three kinds of trees: the shared `main` tree a session publishes, the draft view it renders from `main` plus unsettled sessions, and the view it installs into the folder, possibly replayed from an apply journal (ADR-0007). A receiving device holds a new rule in the incoming tree before the rule is on its disk.

Decision provenance: Kai's refinement answers in US00003 and sign-off of its implementation plan by invoking development, 2026-11-06.

## Decision

- **Every tree Mesh computes omits the paths that the tree's own ignore policy excludes.** That policy is the `.gitignore` / `.ignore` / `.meshignore` files contained in that tree plus the fixed secret excludes. It is never the disk's current policy.
- **On disk, an excluded path is left untouched**: no write, no backup, no removal. Leaving the tree never means deleting the file.
- **Rule files always belong to the tree.** A rule that matches `.gitignore`, `.ignore` or `.meshignore` has no effect on that file; only an excluded parent folder or a fixed exclude keeps it out. The shared policy is then always complete, and the folder walker and the tree's policy can't disagree because a rule file stayed local (added 2026-11-06, Kai, from the US00003 code review).
- When merging a session into a tree, a change to a path the tree's policy excludes is dropped without a merge. It creates no conflict, and the removal wins.

## Consequences

**Positive**

- Two devices that compute the same tree agree on it, and a recovered apply journal makes the same decision as the original apply.
- Receiving a rule and the removal it causes in one apply can't delete the local file.
- Later selective-sync or ignore work has one rule to extend instead of special cases in capture, render and install.

**Negative**

- Mesh carries its own ignore matcher over in-memory trees next to the folder walker, and the two must agree. Tests pin them against each other.
- A tree on `main` that holds an ignored path, from history before this decision, loses that path at the next publication without anyone asking for it.
- Engines before 0.3.0 don't follow the rule. They apply the received removal and move the local file into `.mesh/recovery`, so every device must be updated before such a rule is added.

## Alternatives considered

- **Disk policy decides** (the 0.2.5 `ignored()` check): rejected. The incoming rule isn't on disk yet when the removal is applied, so the file would be deleted, and journal replay could decide differently from the first attempt.
- **Refuse until the rule is restored** (0.2.5 behavior): rejected. The files that most need excluding can't be excluded.
- **Freeze the last version on `main`**: rejected in US00003. A stale file stays in the shared tree and reaches every new device.
- **Let rules exclude rule files, or reject that configuration**: rejected 2026-11-06. An excluded rule file still steers the walker on its own device, which is device-local policy by another name; rejecting it would keep the pause this ADR removes.
- **Device-local exclude lists**: out of scope for now. Only the shared `.meshignore` defines the policy.

## Related

[[Story - US00003 - Exclude device-local files such as the Obsidian workspace from sync|US00003]] (origin); [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions|ADR-0003]] (an exclusion settles an open conflict as a traceable resolution); [[ADR-0007 — Persist state transactionally with recovery journals|ADR-0007]].
