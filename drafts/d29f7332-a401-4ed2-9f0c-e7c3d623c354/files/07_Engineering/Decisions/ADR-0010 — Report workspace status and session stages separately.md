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

**Workspace detail** (amended 2026-11-06 with Kai's answers in [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]): `Status` is the only way hosts read workspace detail, so CLI `status` and the tray report the same state. All fields are additive; existing fields keep their names and meanings.

- Each session adds `started_at` and `last_saved` (Unix seconds), `title` (the summary title once published, otherwise `null`) and `purposes` (requested purposes of its recorded intents, in recorded order, without repeats). Authors stay an identity; nothing marks an agent.
- `has_remote` says whether the workspace synchronizes with a remote at all.
- `last_sync` is the time the last sync with the remote succeeded, kept in local state. A success rewrites it only when the stored time is at least 60 s old, so an idle workspace does not write its state every 3 s cycle; hosts show it to the minute. A workspace without a remote never sets it, and a failed sync leaves it unchanged.
- `saved_copies` lists the files Mesh moved to `.mesh/recovery/<id>/` before writing incoming changes (`id`, original `path`, `created_at`), recorded in local state when the file is displaced. A state that never recorded them is filled once from the recovery folder on the next open. `Status` carries no file-system location: hosts resolve `<root>/.mesh/recovery/<id>/content` from an id they received.

**Session device** (amended 2026-11-07 with Kai's answer to Q5 in [[Story - US00012 - Status window layout follows the design frames|US00012]]): each session adds `device`, the id of the device it was made on. Hosts compare it with the workspace's `device` to tell this device's sessions ("You") from received ones. It is an opaque id, not a name, and hosts never display it (US00012 Q6).

**Conflict list metadata** (amended 2026-11-07 with Kai's answers Q2 and Q3 in [[Story - US00006 - Conflict list and resolution in the status window|US00006]]): `unresolved_conflicts[]` keeps `id`, `path` and `binary` and adds, all in memory:

- `created_at`, when the conflict was recorded.
- `current` (the version in the folder) and `other` (the version kept aside), each `{ contributors, shared_history, deleted, size }`. `contributors` lists `{ author, device, last_saved, purposes }`, newest save first, taken from the sessions in the record's `accepted_sessions` (current) or its `session` (other). `last_saved` is that session's last edit of the file, and `purposes` are the requested purposes of its intents whose paths include the file. `shared_history` says no session contributed to the current version; `deleted` marks a deletion; `size` is the byte length, `0` for a deletion. The JSON names the sides `current` and `other`, never `accepted` and `incoming`. `device` is an opaque id, never displayed.
- `Status` stays in memory: no Git lookup and no file content. Everything that needs bytes, Git or the folder comes on demand from `Workspace::conflict_detail(id)` on a read-only `Workspace::inspect`: the three versions, `edited_after` (this device saved the file again after the conflict was recorded, so the folder no longer shows the current version), and the author and time of the last commit on `main` touching the path for the current version and for the base, only when that commit's blob is exactly that version. Resolving stays `Workspace::resolve`, the call behind `conflicts resolve`.

## Consequences

**Positive**

The tray can show one consistent state while detail views explain individual sessions. Local-only upload bookkeeping avoids an endless push loop caused by publishing push acknowledgments.

**Negative**

The new values replace `waiting_to_sync` and require consumers and tests to change. Summary precedence hides lower-priority activity, which remains visible in session details. This model remains to be implemented under TASK00001.

## Alternatives considered

Treating every sync failure as offline: rejected, because it hides failures that need action, such as expired credentials, behind a non-alarming state. Declaring offline only after several consecutive failures: rejected in favor of reporting it from the first connection-level failure. Showing an AI-summary failure only as `reason` without raising the summary: rejected, because nothing else would surface an actionable provider problem. A single undifferentiated waiting state or flags without precedence were rejected as insufficient for a predictable tray summary. Publishing upload bookkeeping in manifests was rejected because acknowledging a push would itself change the manifest.

## Related

[[Task - TASK00001 - Close remaining 0.2.x stabilization gaps]]; [[Story - US00002 - Local engine control interface for the tray host]]; [[Story - US00001 - Tray icon and status window host]]; [[ADR-0009 — Embed the engine with a runner handle and published status]]; [[Story - US00005 - Workspace detail with Finish session and recovery information]]; [[Story - US00012 - Status window layout follows the design frames]]; [[Story - US00006 - Conflict list and resolution in the status window]].

