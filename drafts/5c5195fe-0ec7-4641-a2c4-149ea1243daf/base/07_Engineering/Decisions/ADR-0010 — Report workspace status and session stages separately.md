---
id: ADR-0010
status: proposed
date: 2026-11-05
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0010 — Report workspace status and session stages separately

## Context

A tray summary and a detailed session view answer different questions. TASK00001 records Kai's answers dated 2026-11-05; the preview's existing three-value status cannot represent draft upload, finalization and offline operation accurately.

Decision provenance: Kai's answers recorded in the linked ticket on 2026-11-05. Recorded as an ADR on 2026-11-05. `proposed` follows the vault convention for agreed designs not yet built.

## Decision

Expose a workspace summary with precedence `needs_attention` > `offline` > `saved_locally` > `draft_uploaded` > `waiting_for_finalization` > `synced`, plus per-session stages `saved_locally`, `draft_uploaded`, `waiting_for_finalization`, `published` or `discarded`. Record each own session's last pushed revision only in local state, and provide a sanitized reason for attention or offline status.

**Classification rules** (amended 2026-11-05 with Kai's approval of the TASK00001 plan's review points):

- **Offline** means the last sync failed with a connection-level transport error: DNS resolution, connection refused or unreachable, timeout, or TLS handshake failure. It applies from the first such failure and clears on the next successful sync. Local capture continues.
- **Needs attention** covers authentication, permission and history failures, unresolved conflicts, capture errors, and engine-raised attention, because the user has to act.
- **A rejected push** (non-fast-forward or `[rejected]`) is `needs_attention`, even though a publication race normally resolves on the next cycle.
- **An unknown or unclassifiable Git failure** is `needs_attention`, never `offline`. Misclassifying it as offline would hide something the user must act on.
- **An AI-summary failure** is persisted and is `needs_attention`, although publication continues with the factual local message ([[ADR-0005 — Make AI summaries configurable and opt-in]]). Disabled text sharing is not a failure.
- **Status is read without the writer lock**, from the committed snapshot ([[ADR-0007 — Persist state transactionally with recovery journals]]). Computing it never writes state.

## Consequences

**Positive**

The tray can show one consistent state while detail views explain individual sessions. Local-only upload bookkeeping avoids an endless push loop caused by publishing push acknowledgments.

**Negative**

The new values replace `waiting_to_sync` and require consumers and tests to change. Summary precedence hides lower-priority activity, which remains visible in session details. This model remains to be implemented under TASK00001.

## Alternatives considered

Treating every sync failure as offline: rejected, because it hides failures that need action, such as expired credentials, behind a non-alarming state. Declaring offline only after several consecutive failures: rejected in favor of reporting it from the first connection-level failure. Showing an AI-summary failure only as `reason` without raising the summary: rejected, because nothing else would surface an actionable provider problem. A single undifferentiated waiting state or flags without precedence were rejected as insufficient for a predictable tray summary. Publishing upload bookkeeping in manifests was rejected because acknowledging a push would itself change the manifest.

## Related

[[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]; [[Story - US00002 - Local engine control interface for the tray host]]; [[Story - US00001 - Tray icon and status window host]]; [[ADR-0009 — Embed the engine with a runner handle and published status]].

