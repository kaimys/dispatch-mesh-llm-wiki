---
id: US00012
type: story
status: "Refinement"
priority: medium
rank: 3750
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 0
open_tests:
open_findings:
discussion:
updated: 2026-11-07
completed:
frozen:
---
Bring the status window's header, state card and activity list in line with the design frames. [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] delivered the right data and behaviour, but its layout was written from prose and does not follow [[Dispatch Mesh desktop app.pdf]]: the stage track is a wrapping text list, sessions are large cards whose heading is a file path, the stage is shown twice, the state card has no reason line, and **Finish session** sits outside the card. This ticket restyles the existing window to the frames. No engine or `Status` change.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a person working in a Mesh workspace, I want the status window to show my recent changes as a compact, scannable list with one clear state at the top, so that I can tell at a glance whether everything is fine or something needs me.

## Context

- Follow-up of [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] (Done, frozen). Its contract cited "frames in [[Dispatch Mesh desktop app.pdf]]" but no acceptance criterion checked the layout, and plan step 6 described the window in prose only. The built window (screenshot, 2026-11-06, build after `819d9b0`) differs from the frames as listed under *Gaps* below.
- **The design files are not readable as text.** `06_UX/Dispatch Mesh desktop app.pdf` is 13 pages of images with no text layer; `06_UX/Dispatch Mesh desktop app.html` is a compressed, self-unpacking bundle with no readable markup. An agent that greps or extracts text from them finds nothing. Since 2026-11-06 the readable versions are [[Dispatch Mesh desktop app — design spec]] (tokens, glyph paths, components, per-view rules), one PNG per frame in `06_UX/frames/`, and the frame markup in `06_UX/frames/source/`. This ticket also spells out the parts it needs below.
- Frames, by PDF page (page = canvas artboard):
  - **p. 3 — 3a Status window (component)**, `06_UX/frames/03a-status-window.png`: 440 × 640 overview, the layout this ticket implements.
  - **p. 4 — 3b One frame per summary state**, `06_UX/frames/03b-summary-states.png`: the same window in all six summary states.
  - **p. 5 — 3c Activity + presence**, `06_UX/frames/03c-activity-presence.png`: session detail (out of scope here, see Open questions).
  - **p. 13 — 6 Tokens + components**, `06_UX/frames/06-tokens-components.png`: colors, type scale, spacing, status glyphs, stage track.
- **Newer than the PDF (canvas, 2026-11-06):** the last stage is labelled "Added to shared history" everywhere, and a workspace without a remote shows a three-segment track (no "Shared as draft"). The PDF export and the PNGs predate both; the design spec lists them under *Changes since the PDF export*.
- Touches `tray/ui/app.js`, `tray/ui/app.css`, `tray/ui/index.html` and the new `tray/ui/fonts/` (Q3), `tray/tauri.conf.json` (Q7), and one additive `Status` field (Q5). `tray/src/model.rs` keeps all wording; if a label the design needs is missing from the view model, it is added there, unit-tested. *(Widened at refinement, 2026-11-07; originally "only `tray/ui/*`".)*
- Vocabulary and principles: [[Claude Design briefing — desktop app]] §3 (quiet by default), §5 (words).

### Gaps between the build and frame 3a/3b

| Area | Built (US00005) | Design (p. 3, p. 4) |
|---|---|---|
| Header | Workspace name as a large heading, path, "Last synced …" on its own line | Workspace name in the top row (as the workspace switcher), path below in mono 12 px muted; last sync inside the state card, bottom right |
| State card | Tray mark as glyph, label only | 28 px status glyph (p. 13), label 17/600, **reason line** 13/400 under it (and the reassurance line where one exists), actions row inside the card: buttons left, "Last synced …" right |
| Finish session | Next to the "RECENT ACTIVITY" heading | Secondary button in the state card's actions row (for Saved on this device and Shared as draft); the attention action stays the card's primary button |
| Section heading | "RECENT ACTIVITY" caps heading | Underlined section nav: **Activity** (current). Conflicts tab: see Q2 |
| Session item | Bordered card; heading = file path(s) wrapping to two lines; stage label, then the stage list, then author line | Divider-separated row, no card. 28 px avatar with initials. Line 1: author 13/600 + meta 12 muted ("2 files · last saved 2 min ago"). Line 2: summary title once published, else the files, **one line, ellipsized**. Line 3: segmented stage track + current stage label |
| Stage track | Text list "✓ Saved on this device ○ Shared as draft …", wraps over two lines; stage also printed above it | 4 segments (3 without a remote), each 4 px high, 3 px gap, 112 px wide in total; segments up to the current stage filled; stage named **once**, as text right of the track |
| Density | 2 sessions visible in the default window | 3 sessions visible at 440 × 640 |

### Design tokens (from p. 13)

- Ground `#F7F7F5`, surface `#FFFFFF`, line `#E3E2DE`, line-strong `#C9C8C3`; text `#1C1D1F`, text-2 `#3D3F44`, text-3 `#55575C`.
- Action `#2457C5` (primary button, links, focus). Working segments `#2457C5`; published track `#2F6B4F` (all segments); empty segment `#E3E2DE`; a received session's first segment `#C9D6F2` (it never had a local stage on this device).
- Attention card ground `#FDF1E7`, border `#F0C9A8`, label `#8A3608`, glyph `#B4490B`. Offline card ground `#F1F2F4`, border `#DCDFE4`.
- Dark values: the token table on p. 13.
- Type: IBM Plex Sans / Mono, falling back to Segoe UI and Consolas (see Q3). 17/600 state label, 14/600 workspace name, 13/400 body, 12/400 meta, mono 12 paths.
- Spacing 4 · 8 · 12 · 16 · 24; radius 4 controls, 8 cards and window.

## Scope

- Window header, state card and activity list restyled as in the table above, in light and dark theme.
- Status glyphs for the state card (24-unit viewBox, stroke 1.6, shapes on p. 13): check in a circle (Up to date), two arc arrows in a circle (Saved, Shared as draft, Finishing up), slash in a circle (Offline), exclamation in a triangle (Needs attention).
- The reason line under the label for every state; for the working states the view model supplies a short sentence (p. 4 wording, e.g. Shared as draft: "Collaborators can see your session. It joins shared history after 5 minutes without edits, or when you finish it.").
- Segmented stage track with the stage named once in text; screen readers get "Stage: Shared as draft, 2 of 4".
- Saved copies (US00005) keep their disclosure, restyled with the same tokens, placed below the activity list.
- Native title bar, then the design's top row: app mark and workspace name (switcher with several workspaces), no own hide button (Q1).
- Activity / Conflicts section nav with the count badge; the Conflicts tab shows the placeholder from Q2.
- IBM Plex Sans and Mono bundled (Q3); default window 440 × 640 (Q7).
- `SessionStatus.device` (Q5) for "You" and the received first segment; no device names (Q6); avatar colors per Q9.

## Out of scope

- Session detail view (p. 5, 3c), conflicts tab content ([[Story - US00006 - Conflict list and resolution in the status window|US00006]]), presence ([[Story - US00007 - Recent-save presence in the status window|US00007]]), the several-workspaces list (p. 10), tray icons and menu (p. 1–2).
- Any change to `Status` other than `SessionStatus.device` (Q5), any engine change, and which sessions are listed (US00005 AC 5 stands).
- Custom window chrome (`decorations: false`) and device names (Q1, Q6).

## Open questions

Refinement started 2026-11-07 (no tracker or chat; questions recorded here). Checked against `main` at `403b165`.

**Facts from the code (2026-11-07):**
- `SessionStatus` (`src/status.rs`) has `author`, `stage`, times, `files`, `title` and `purposes`, but **no device**. `tray/src/model.rs` shows the author's name for every session; there is no "You". Telling own sessions from received ones, which the frames need for "You" and the pale first track segment, needs data that `Status` doesn't carry today.
- A device id is a random UUID (`Uuid::new_v4()` at `init`). No human-readable device name exists anywhere, so the frames' "laptop" / "on Bob's desktop" has no source.
- The window is 440 × 520 in `tray/tauri.conf.json`; the frames and criterion 5 assume 440 × 640. Scope says only `tray/ui/*` changes.
- The view model has `note` only for offline (`OFFLINE_NOTE`). The spec gives a reassurance line for the conflict reason only, and a reason line per working state.

- [x] **Q1 — Window chrome.** Keep the native Windows title bar ("Dispatch Mesh", minimise/maximise/close) above the design's header, or draw the design's own top row (app mark, workspace name as switcher, hide button) with Tauri `decorations: false`? The second matches p. 3 but needs a drag region and native-feeling hide/move.
  **Answer (Kai, 2026-11-07):** keep the native Windows title bar (move, snap, close = hide as today). Below it, the design's top row without its own hide button: app mark and workspace name, which is the switcher when there are several workspaces. No `decorations: false`, no drag region.
- [x] **Q2 — Conflicts tab.** Show the Activity / Conflicts section nav now (Conflicts empty or disabled until US00006), or introduce the nav together with US00006?
  **Answer (Kai, 2026-11-07):** build the nav now. The **Conflicts** tab carries the count badge from the existing unresolved count and shows a placeholder: "N unresolved conflicts. Review them with the command line until conflict review arrives." (with none: "No unresolved conflicts."). [[Story - US00006 - Conflict list and resolution in the status window|US00006]] replaces the placeholder.
- [x] **Q3 — Font.** Bundle IBM Plex Sans + Mono with the app, or use the system fallback (Segoe UI, Consolas) and keep the design's sizes and weights?
  **Answer (Kai, 2026-11-07):** bundle IBM Plex Sans 400/500/600 and IBM Plex Mono 400/500 as woff2 in `tray/ui/fonts/`, with the SIL OFL licence file alongside. Fallbacks: Segoe UI / system-ui and Consolas. No CSP change (`default-src 'self'` covers fonts).
- [x] **Q5 — Own vs. received sessions.** "You" for own sessions and the pale first segment for received ones need the session's device. Add `device` to `SessionStatus`, an additive field that contradicts "no `Status` change" under Out of scope, or drop both (always the author's name; every track starts with a normal segment)?
  **Answer (Kai, 2026-11-07):** add `device` to `SessionStatus`, an additive field that CLI `status` shows too. The model compares it with `Status.device`: "You" for own sessions, and the pale first segment for received ones. This amends [[ADR-0010 — Report workspace status and session stages separately|ADR-0010]]. US00006's conflict sides use the same rule.
- [x] **Q6 — Device names.** No device has a readable name. Leave the device out of the meta line ("Alice · 1 file · last saved 40 s ago"), or add a device name (a new setting and a new ticket)?
  **Answer (Kai, 2026-11-07):** leave the device out: "Alice · 1 file · last saved 40 s ago". Never show a device id. A device name can come later as its own ticket. US00006 follows the same rule.
- [x] **Q7 — Default window size.** Change `tauri.conf.json` to the frames' 440 × 640 (min size unchanged)? Scope currently limits the ticket to `tray/ui/*`.
  **Answer (Kai, 2026-11-07):** default 440 × 640 in `tray/tauri.conf.json`, minimum unchanged (380 × 360); on a shorter screen the window fits the work area. Scope widens to include `tauri.conf.json`.
- [x] **Q8 — Reason and reassurance lines.** Which wording per state and reason? The spec gives the conflict reassurance ("Both versions of each file are kept. Other files keep syncing.") and the working-state lines. Finishing up says "Writing a summary for your session (N files) …", but finalization also waits for earlier sessions, and N is ambiguous with several closed sessions. Other attention reasons (capture error, sync failure, AI summary failure, dependent edits) have no reassurance line in the spec.
  **Answer (Kai, 2026-11-07):** as proposed, all wording in `tray/src/model.rs`:
  - Conflicts: "Both versions of each file are kept. Other files keep syncing."
  - Sync failure: "Your edits are saved on this device. Mesh retries automatically." (replaces the separate retry note)
  - Capture error: "Saved files stay as they are. Mesh tries again on the next save."
  - Dependent edits: "Your edits are saved on this device. Finish the session to share them."
  - AI summary failure: "Your session is added to shared history with a local summary."
  - Finishing up: "Adding your finished session to shared history." / "Adding your N finished sessions to shared history." (no "writing a summary", no file count)
  - Saved on this device, Shared as draft, Up to date: the spec's lines.
- [x] **Q9 — Avatar colors.** The spec says "a per-person color" without a rule or palette. How is the color chosen, and from which colors (white initials need 4.5:1)?
  **Answer (Kai, 2026-11-07):** a stable hash of the author's email selects one of six colors, each passing 4.5:1 with white initials in light and dark theme, for example `#2457C5`, `#7A4FB0`, `#2F6B4F`, `#B4490B`, `#3D3F44`, `#0E6E7E`, with dark variants checked. "You" always uses the neutral grey of frame 3a (`#3D3F44`, kept dark enough for white initials in both themes). The same person gets the same color on every device.
- [x] **Q4 — Readable design source.** Before development, add the design to the wiki in a form an agent can read.
  **Answer (Kai, 2026-11-06):** done. `06_UX/frames/*.png` (one per frame, from the PDF), `06_UX/frames/source/*.dc.html` (frame markup from the canvas, including the 2026-11-06 changes) and [[Dispatch Mesh desktop app — design spec]]. Where the spec and a PNG disagree, the spec wins.

## Acceptance criteria

1. With one workspace and an open session, the window opens at 440 × 640 (fitted to a shorter screen) and matches frame p. 3 in structure below the native title bar: a top row with the app mark and workspace name, the path, one state card holding glyph, label, reason, actions and last sync, the Activity / Conflicts nav, then the activity list. Checked by a side-by-side screenshot recorded in the test plan.
2. Each of the six summary states renders like its frame on p. 4: card ground and border, glyph shape, label, the reason and reassurance lines from Q8, and the action (primary **Open conflicts** / **Finish session** for attention reasons, secondary **Finish session** for Saved on this device and Shared as draft, none otherwise). State is never shown by color alone: glyph shape and label differ per state.
3. A session row shows avatar, author ("You" for this device's sessions) and meta ("N files · last saved …", no device) on line 1, the title or files on one ellipsized line 2, and the segmented track with the stage label on line 3. The stage name appears exactly once per row.
4. The track has 4 segments with a remote and 3 without. Filled segments match the stage, and a published session shows all segments in the published color. A session from another device has a pale first segment. A discarded session shows a "Discarded" tag instead of a track.
5. At 440 × 640 at least three session rows are visible without scrolling, and no text overflows horizontally. Long paths ellipsize; the full path is available as a tooltip and to screen readers.
6. Colors, sizes and spacing use the design spec's tokens, in light and dark theme; all text meets 4.5:1. Text renders in the bundled IBM Plex Sans / Mono, and the licence file ships with them.
7. Every control keeps its US00005 keyboard order, focus outline and accessible name; the track has the screen-reader text from Scope. The Conflicts tab has the accessible name "Conflicts, N unresolved".
8. The **Conflicts** tab shows the unresolved count as a badge and the Q2 placeholder; **Open conflicts** on the attention card switches to it.
9. Avatar colors follow Q9: the same email always gets the same color, "You" is neutral grey, and white initials meet 4.5:1 in both themes.
10. CLI `status` includes `device` for every session; all existing fields are unchanged.
11. `tray/src/model.rs` unit tests cover the new and changed wording (reason and reassurance lines per state and reason, Finishing up for one and several sessions, track segment count, stage-of-n text, "You" vs. author, the received first segment, the avatar color rule, the Conflicts placeholder).

## Implementation plan

<!-- GUIDE: Written by /implementation-plan. -->

**Note for the planner and developer:** read [[Dispatch Mesh desktop app — design spec]] (*Status window*, *Tokens*, *Glyphs*) and open `06_UX/frames/03a-status-window.png`, `03b-summary-states.png` and `06-tokens-components.png` as images before writing the plan. Before handing to review, compare a screenshot of the running window against them, as in the spec's *How to check a build*. Text extraction from the PDF or HTML returns nothing.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

## As-built notes

## Follow-ups

## References

- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app — design spec]], [[Dispatch Mesh desktop app.pdf]] (pp. 3, 4, 13), `06_UX/frames/`
- Screenshot of the US00005 window, 2026-11-06 (Kai)

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
