---
id: ADR-0003
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions

## Context

Concurrent edits may be ambiguous, and a local-only conflict list cannot protect alternate versions on other clients or after restart. Binary versions cannot be represented faithfully by a textual diff alone.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Automatically merge clean text changes and retain a durable synchronized record of each unresolved conflict with exact base and competing snapshots, identities, path and resolution state. Keep the accepted view visible while unrelated paths synchronize; record version selections against their contributing input revisions so publication order does not undo a resolution.

## Consequences

**Positive**

Users can recover all versions and make a traceable choice. Text and binary conflict tests demonstrate selections surviving incoming-first publication.

**Negative**

Conflicts consume retained storage and can delay finalization of affected sessions. Clean text merges do not prove semantic correctness. The preview offers whole-version choices for text too; a text editor and AI proposals remain future work.

## Alternatives considered

Overwrite the losing version or rely only on console warnings: rejected because recovery is lost. One textual diff for every conflict: insufficient for exact binary recovery. Automatic binary or AI binary merging: rejected; choose one complete version.

## Related

[[Dispatch Mesh requirements and architecture]]; [[github-workflow-findings-2026-11-04]]; [[ADR-0007 — Persist state transactionally with recovery journals]].

