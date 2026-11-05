---
id: ADR-0002
status: accepted
date:
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0002 — Group files into sessions with an inactivity boundary

## Context

Related work often spans files. File switches and individual saves are unreliable indicators that the user has finished.

Decision provenance: the Dispatch Mesh development chat and the implemented console preview, confirmed for ADR recording by Kai on 2026-11-05. The original individual decision date is not recoverable from the available chat timestamps, so `date` is left unset rather than guessed. Recorded on 2026-11-05.

## Decision

Keep one editing session across files until five minutes pass without local edits or the user or agent optionally finishes it. Freeze that session for finalization and place later edits in a new session; incoming changes and presence updates do not reset the timer.

## Consequences

**Positive**

Related changes share meaningful history. Users can work without a mandatory finish step; agents can finish through the CLI and future desktop controls expose the same action.

**Negative**

A five-minute heuristic can split one task or combine unrelated tasks. Final publication still depends on integration, resolution and summarization; a maximum duration is not decided.

## Alternatives considered

Close on file switch: rejected because related changes span files. Mandatory explicit finish: rejected for ordinary users. Thirty-minute maximum: discussed but not confirmed.

## Related

[[Dispatch Mesh requirements and architecture]]; [[ADR-0001 — Synchronize drafts separately from permanent history]]; [[ADR-0004 — Bind agent intent to attributable changes]].

