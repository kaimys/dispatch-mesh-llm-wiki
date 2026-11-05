---
id: ADR-0009
status: proposed
date: 2026-11-05
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0009 — Embed the engine with a runner handle and published status

## Context

The tray needs reusable engine lifecycle and observation APIs while the CLI remains usable. US00002 records Kai's answers dated 2026-11-05; these are agreed design requirements, not completed implementation.

Decision provenance: Kai's answers recorded in the linked ticket on 2026-11-05. Recorded as an ADR on 2026-11-05. `proposed` follows the vault convention for agreed designs not yet built.

## Decision

Move the run loop into the library with a public async entry point and a `Runner::start(root) -> RunnerHandle` wrapper owning a background thread and Tokio runtime. The handle provides changed status snapshots through a watch receiver plus `stop()` and `join()`; retain on-disk coordination with CLI commands and refuse a second runner with an explicit retryable error.

## Consequences

**Positive**

UI toolkits retain the main thread and need no Tokio runtime. One shared loop serves CLI and desktop hosts, with status refreshed from durable state about once per tick.

**Negative**

A UI-process crash also stops its embedded engine until restart. Persist summary failures and include capture, sync and summary errors in snapshots without secrets. Stop must capture saved edits without waiting for network completion. This API remains to be implemented and verified under US00002.

## Alternatives considered

A supervised child runner or IPC service: deferred; existing CLI changes are picked up within the sync cadence. UI polling SQLite: rejected in favor of runner-published snapshots. Observer attachment or automatic takeover of a locked runner: rejected; explain and retry after the existing owner stops.

## Related

[[Story - US00002 - Local engine control interface for the tray host]]; [[Story - US00001 - Tray icon and status window host]]; [[ADR-0010 — Report workspace status and session stages separately]].

