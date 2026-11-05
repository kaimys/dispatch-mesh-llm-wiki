---
id: ADR-0004
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0004 — Bind agent intent to attributable changes

## Context

A useful commit message should include the user's purpose, but a filesystem watcher cannot infer which writer caused a change or whether the requested outcome was achieved.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Agents register a concise purpose before editing through `intent begin --purpose ... --paths ...`, then close the scope with `intent end <id>`. Bind that intent only to attributable changes on the declared relative paths and add it locally to commit bodies for both AI and fallback messages.

## Consequences

**Positive**

Purpose survives synchronization and provider failures without storing a whole conversation. Direct human edits remain supported without intent records.

**Negative**

Only one intent scope is active per device. Mixed scoped and unscoped edits lose aggregate attribution; concurrent writers on one path cannot be reliably identified. Separate agent workspace/device contexts remain necessary.

## Alternatives considered

Infer purpose from filenames or associate all session changes with one purpose: rejected because unrelated edits would be claimed. Depend on the model to repeat intent: rejected because provider failures would remove it. Treat purpose as evidence of completion: rejected.

## Related

[[Dispatch Mesh requirements and architecture]]; [[ADR-0002 — Group files into sessions with an inactivity boundary]]; [[ADR-0005 — Make AI summaries configurable and opt-in]].

