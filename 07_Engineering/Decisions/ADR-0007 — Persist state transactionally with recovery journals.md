---
id: ADR-0007
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0007 — Persist state transactionally with recovery journals

## Context

Capture, incoming file application and publication can be interrupted independently. Durable session state alone does not make partially applied filesystem changes recoverable.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Persist local engine state transactionally in SQLite and journal incoming file application, pending capture and displaced content before completing mutations. Preserve drafts, conflict inputs and recovery records across restarts, and migrate older workspace state atomically with its original data retained.

## Consequences

**Positive**

Restart recovery has durable evidence and captured work can survive network or process failure. Status can read a committed SQLite snapshot without taking the writer operation lock.

**Negative**

SQLite and filesystem updates need explicit coordination and recovery replay. Recovery data accumulates; automatic cleanup and retention policy remain future work. A desktop database is not a distributed cloud coordination service.

## Alternatives considered

In-memory sessions or best-effort shutdown saves: rejected because crashes lose work. Directly apply incoming content without a journal: rejected because interrupted writes become ambiguous.

## Related

[[Dispatch Mesh requirements and architecture]]; [[github-workflow-findings-2026-11-04]]; [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]; [[ADR-0008 — Store content as blobs referenced by metadata]].

