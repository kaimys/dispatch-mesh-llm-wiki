---
id: ADR-0008
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0008 — Store content as blobs referenced by metadata

## Context

Embedding file contents as integer arrays in JSON obscures ordinary files, expands metadata and complicates inspection. Content still needs exact preservation, deduplication and reachable recovery versions.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Encode remote metadata using references to native Git blobs kept reachable through the device snapshot tree, and store local payloads in content-addressed SQLite BLOBs. Distinguish empty files from deletion, validate referenced objects, read legacy snapshots and retain original state during migration.

## Consequences

**Positive**

Draft contents are inspectable as files and identical content can be deduplicated. Text and binary payloads retain exact bytes without integer arrays in new persisted JSON.

**Negative**

Codec and transport validation must enforce references, size limits and reachability. Collaborating clients need upgrading for version-two snapshots; earlier history retains its old representation.

## Alternatives considered

Continue embedding byte arrays in JSON: rejected after the user's feedback. Store only unreachable object IDs: rejected because fetch and garbage collection would not reliably retain the content. Rewrite historical commits to remove old encoding: rejected to preserve shared history.

## Related

[[Dispatch Mesh requirements and architecture]]; [[ADR-0001 — Synchronize drafts separately from permanent history]]; [[ADR-0007 — Persist state transactionally with recovery journals]]; [[Task - TASK00002 - Choose the Git backend before bundling Git]]. Blob representation does not settle CLI versus library backend.

