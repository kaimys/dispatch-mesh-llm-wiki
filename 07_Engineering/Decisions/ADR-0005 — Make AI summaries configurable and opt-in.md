---
id: ADR-0005
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0005 — Make AI summaries configurable and opt-in

## Context

Users need hosted and local models, while synchronization must work without a provider. Change snapshots can contain private text.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Use a provider boundary with an initial configurable Chat Completions adapter, explicit text-sharing consent, frozen snapshot inputs and validated output. On unavailable or invalid output, use a factual local message without contacting an alternate provider; never send binary contents and always append attributable recorded intent locally.

## Consequences

**Positive**

Users can change compatible providers or use a local model. Capture and synchronization remain independent of AI availability; metadata-only operation avoids transmitting document text.

**Negative**

Compatible servers differ in capabilities and response formats, so configuration, validation, bounded retries and diagnostics are required. Text-sharing consent does not guarantee secret detection; generic secret filtering is not implemented.

## Alternatives considered

Hard-code one provider or silently fall back to another: rejected for portability and privacy. Require cloud summaries for every session: rejected for offline operation. Responses and Anthropic Messages are possible future adapters, not initial requirements.

## Related

[[Dispatch Mesh requirements and architecture]]; [[ADR-0004 — Bind agent intent to attributable changes]].

