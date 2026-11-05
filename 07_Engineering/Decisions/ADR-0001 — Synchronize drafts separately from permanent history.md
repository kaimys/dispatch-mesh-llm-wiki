---
id: ADR-0001
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0001 — Synchronize drafts separately from permanent history

## Context

Saved changes must reach collaborators before a human finishes a logical task. Repeatedly amending the shared branch makes concurrent publication race against rewritten history.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Synchronize open session revisions on per-device refs under `mesh/devices/<device-id>`, identified by stable session IDs. Publish finalized integrated sessions as immutable commits on the configured shared branch; never amend or force-push that shared history.

## Consequences

**Positive**

Collaborators receive unfinished work without one permanent commit per save. Stable session identity survives changing draft revisions.

**Negative**

Clients must reconstruct and integrate device manifests, track dependencies, and recover interrupted publication. GitHub's `main` browser shows finalized work rather than every saved draft.

## Alternatives considered

Amend and force-push the last shared commit: rejected after discussing concurrent writers and unnecessary commits. Synchronize only after finalization: rejected because drafts would remain unavailable during active editing.

## Related

[[Dispatch Mesh requirements and architecture]]; [[ADR-0002 — Group files into sessions with an inactivity boundary]]; [[ADR-0008 — Store content as blobs referenced by metadata]].

