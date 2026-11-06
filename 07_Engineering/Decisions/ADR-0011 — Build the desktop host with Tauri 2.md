---
id: ADR-0011
status: accepted
date: 2026-11-05
superseded_by:
owner: "[[Kai]]"
recorded: 2026-11-05
---

# ADR-0011 — Build the desktop host with Tauri 2

## Context

0.3 needs a tray icon, a compact status window, toasts and screen-reader labels on Windows, and the same surfaces must port to the macOS menu bar (0.7) and Linux trays (0.8). Planned later work also needs an installer (0.4), login startup (0.6) and updates. The milestone plan lists "Tray UI toolkit" among the decisions to close before implementing 0.3. The engine runs in-process through `RunnerHandle` (ADR-0009). The visual design was delivered by Claude Design as HTML with a token sheet ([[Dispatch Mesh desktop app.pdf]]).

Decision provenance: Kai's answer in [[Story - US00001 - Tray icon and status window host|US00001]] refinement, 2026-11-05, with a Windows spike as a condition before the plan relies on it.

**Accepted 2026-11-06 by Kai:** the decision below is built and verified under US00001 (Windows spike passed; reviewed clear; all 11 manual checks passed; CI green on `windows-latest` and `ubuntu-latest`, the first Linux build of the tray, after adding a PNG app icon in `296a09d`). Merged to `main` at `29d3d58`.

## Decision

The desktop host is a **Tauri 2** application. Its Rust side embeds the engine through `RunnerHandle` on Tauri's Tokio runtime and awaits the status `watch` channel; the window is a small **static** HTML/CSS/JS frontend served from the app's own assets, with **no npm, no bundler and no remote content**, styled from the design's token sheet as a compact utility window, not a web-app shell. Presentation logic (labels, icon choice, aggregation, tooltips) lives in Rust, so the frontend only renders a view model.

## Consequences

**Positive**

- The design's colors, type and spacing carry over almost directly; text and side-by-side comparison are cheap to render.
- Maintained plugins exist for tray, notifications, autostart, installer bundling and updates, matching 0.4, 0.6, 0.7 and 0.8 work.
- No second language toolchain in 0.3: `cargo build` embeds the static assets.

**Negative**

- Depends on the system webview: WebView2 on Windows (shipped with Windows 11; the 0.4 installer must bootstrap it elsewhere), WKWebView on macOS, WebKitGTK on Linux, whose quality varies by distribution.
- Higher memory use than a native or immediate-mode UI, from the WebView2 processes.
- Building on Linux needs the WebKitGTK/AppIndicator development packages, in CI and for every Linux developer.
- If later views outgrow plain JS, adding a frontend toolchain is a new decision.

## Alternatives considered

- **egui/eframe + `tray-icon`** (runner-up): all Rust and small, but non-native text rendering, hand-built comparison view, toasts, autostart and installer, and known friction keeping the process alive with the window hidden and combining the tray with eframe on Linux. It is the fallback if the spike fails.
- **Slint:** compact and closer to native, but a smaller ecosystem, separate crates for tray and notifications, and a licence review.
- **Native Win32:** smallest on Windows, but rewritten for 0.7 and 0.8, which the milestone rules out.
- **Tauri with an npm/bundler frontend:** rejected for 0.3; the window is small enough that plain static files avoid the toolchain.

## Related

[[Story - US00001 - Tray icon and status window host]] (origin); [[ADR-0009 — Embed the engine with a runner handle and published status]] (engine embedding); [[ADR-0010 — Report workspace status and session stages separately]] (the status rendered); [[ADR-0012 — Ship the desktop host as a separate executable serving several workspaces]]; [[Dispatch Mesh milestone plan]] → *Decisions to close during delivery*.
