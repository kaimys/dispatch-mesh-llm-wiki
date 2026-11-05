---
id: ADR-0006
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0006 — Let Mesh own Git operations in managed folders

## Context

Target users include people unfamiliar with Git. Concurrent manual branch changes, rebases and pulls substantially expand reconciliation and recovery requirements.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Mesh owns repository operations for the initial managed-folder workflow; manual Git mutation alongside Mesh is outside supported scope. Preserve captured work and report actionable attention when unexpected repository changes invalidate state, rather than resetting history or silently discarding files.

## Consequences

**Positive**

A bounded workflow can offer synchronization without Git expertise. Future guided setup can perform repository and credential operations on the user's behalf.

**Negative**

Advanced developer interoperability is deferred. The current CLI preview still requires Git and preconfigured authentication; guided setup and bundled distribution are not yet implemented.

## Alternatives considered

Fully support arbitrary manual Git workflows from the outset: rejected as unnecessary complexity for initial users. Reset unexpected repository state: rejected because user history and edits must survive.

## Related

[[Dispatch Mesh requirements and architecture]]; [[Dispatch Mesh milestone plan]]; [[Task - TASK00002 - Choose the Git backend before bundling Git]]. This decision does not choose the Git implementation backend.

