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

## Consequences

**Positive**

The tray can show one consistent state while detail views explain individual sessions. Local-only upload bookkeeping avoids an endless push loop caused by publishing push acknowledgments.

**Negative**

The new values replace `waiting_to_sync` and require consumers and tests to change. Summary precedence hides lower-priority activity, which remains visible in session details. This model remains to be implemented under TASK00001.

## Alternatives considered

Offline means connection-level failure from the first failed sync and clears on successful sync; authentication, permission and history failures require attention. A single undifferentiated waiting state or flags without precedence were rejected as insufficient for a predictable tray summary. Publishing upload bookkeeping in manifests was rejected because acknowledging a push would itself change the manifest.

## Related

[[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]; [[Story - US00002 - Local engine control interface for the tray host]]; [[Story - US00001 - Tray icon and status window host]]; [[ADR-0009 — Embed the engine with a runner handle and published status]].

