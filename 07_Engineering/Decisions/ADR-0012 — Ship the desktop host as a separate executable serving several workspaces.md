---
id: ADR-0012
status: proposed
date: 2026-11-05
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0012 — Ship the desktop host as a separate executable serving several workspaces

## Context

A tray application on Windows must be built with `windows_subsystem = "windows"` so no console window appears. A program built that way has no console when run from a terminal, so the CLI, which agents call constantly, could not print or pipe its output without a console-attach workaround with known flaws. The milestone speaks of "each workspace's state", 0.6 requires "one desktop host per user context and one engine owner per workspace", and the Claude Design result designs for several workspaces in one window.

Decision provenance: Kai's answers in [[Story - US00001 - Tray icon and status window host|US00001]] refinement, 2026-11-05.

## Decision

- The desktop host is a **separate executable**, `dispatch-mesh-tray.exe`, built with the GUI subsystem from its own package `dispatch-mesh-tray` in a **Cargo workspace** with the CLI. Both packages share one version (`[workspace.package]`), one `Cargo.lock`, and the engine through the `dispatch_mesh` library. `dispatch-mesh.exe` stays an ordinary console program. Both workspace members are default members, so the standard gates build and test both on every CI platform.
- **One host serves several workspaces**, each with its own `RunnerHandle`. The tray shows the highest-precedence state of the workspaces it runs (Needs attention > Offline > Working > Up to date). A workspace whose runner lock is held elsewhere is reported as "already running" with Retry and does not stop the others.
- Its app identifier is **`de.eightnine.mesh`**, which names its per-user directories and, later, its installer and startup registrations.

## Consequences

**Positive**

- The CLI's behavior for agents and scripts is untouched; no console-attach code.
- Login startup (0.6) registers the tray program alone, which is also where one-host-per-user is enforced.
- One release bump versions both programs.

**Negative**

- The installer (0.4) ships and signs two executables.
- Every build, including CLI-only work on Linux, compiles the tray and needs its system packages.
- Changing the identifier later moves the per-user directories and registrations and needs a migration.
- Multi-workspace aggregation is in every later tray surface (tooltips, list ordering, notifications).

## Alternatives considered

- **One binary with a `tray` subcommand and a console-attach workaround:** rejected; the shell prompt returns before output and piping and stdin misbehave, which breaks agent use of the CLI.
- **One host per workspace:** rejected; several tray icons for one product, and it contradicts 0.6's one host per user.
- **Excluding the tray from default members:** rejected; the gates would silently skip it.

## Related

[[Story - US00001 - Tray icon and status window host]] (origin); [[ADR-0009 — Embed the engine with a runner handle and published status]]; [[ADR-0010 — Report workspace status and session stages separately]]; [[ADR-0011 — Build the desktop host with Tauri 2]]; [[Dispatch Mesh milestone plan]] (0.4, 0.6).
