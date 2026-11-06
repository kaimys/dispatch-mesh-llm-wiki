---
id: US00012
type: story
status: Backlog
priority: medium
rank: 3750
version_target: v0.3.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 4
open_tests:
open_findings:
discussion:
updated: 2026-11-06
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
- **The design files are not readable as text.** `06_UX/Dispatch Mesh desktop app.pdf` is 13 pages of images with no text layer; `06_UX/Dispatch Mesh desktop app.html` is a compressed, self-unpacking bundle with no readable markup. An agent that greps or extracts text from them finds nothing. This ticket therefore spells the design out below; the frames remain the visual reference and must be **viewed as images** (render the page, then open the image).
- Frames, by PDF page (page = canvas artboard):
  - **p. 3 — 3a Status window (component)**: 440 × 640 overview, the layout this ticket implements.
  - **p. 4 — 3b One frame per summary state**: the same window in all six summary states.
  - **p. 5 — 3c Activity + presence**: session detail (out of scope here, see Open questions).
  - **p. 13 — 6 Tokens + components**: colors, type scale, spacing, status glyphs, stage track.
- **Newer than the PDF (canvas, 2026-11-06):** the last stage is labelled "Added to shared history" everywhere, and a workspace without a remote shows a three-segment track (no "Shared as draft"). The PDF export predates both; re-export it (see Open questions, Q4).
- Touches only `tray/ui/app.js`, `tray/ui/app.css` (and `tray/ui/index.html` if the structure needs it). `tray/src/model.rs` keeps all wording; if a label the design needs is missing from the view model, it is added there, unit-tested.
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

## Out of scope

- Session detail view (p. 5, 3c), conflicts tab content ([[Story - US00006 - Conflict list and resolution in the status window|US00006]]), presence ([[Story - US00007 - Recent-save presence in the status window|US00007]]), the several-workspaces list (p. 10), tray icons and menu (p. 1–2).
- Any change to `Status`, the engine or which sessions are listed (US00005 AC 5 stands).

## Open questions

- [ ] **Q1 — Window chrome.** Keep the native Windows title bar ("Dispatch Mesh", minimise/maximise/close) above the design's header, or draw the design's own top row (app mark, workspace name as switcher, hide button) with Tauri `decorations: false`? The second matches p. 3 but needs a drag region and native-feeling hide/move.
- [ ] **Q2 — Conflicts tab.** Show the Activity / Conflicts section nav now (Conflicts empty or disabled until US00006), or introduce the nav together with US00006?
- [ ] **Q3 — Font.** Bundle IBM Plex Sans + Mono with the app, or use the system fallback (Segoe UI, Consolas) and keep the design's sizes and weights?
- [ ] **Q4 — Readable design source.** Before development, add the design to the wiki in a form an agent can read: one PNG per frame (from the PDF, or exported from the canvas) under `06_UX/frames/`, plus a re-exported PDF that includes the 2026-11-06 changes. Who exports, and are PNGs enough or should the frame source (`.dc.html`) be added too?

## Acceptance criteria

1. With one workspace and an open session, the window at its default size matches frame p. 3 in structure: path under the top row, one state card holding glyph, label, reason, actions and last sync, then the activity list. Checked by a side-by-side screenshot recorded in the test plan.
2. Each of the six summary states renders like its frame on p. 4: card ground and border, glyph shape, label, reason line, and the action (primary **Open conflicts** / **Finish session** for attention reasons, secondary **Finish session** for Saved on this device and Shared as draft, none otherwise). State is never shown by color alone: glyph shape and label differ per state.
3. A session row shows avatar, author and meta on line 1, title or files on one ellipsized line 2, and the segmented track with the stage label on line 3. The stage name appears exactly once per row.
4. The track has 4 segments with a remote and 3 without; filled segments match the stage; a published session shows all segments in the published color; a received session's first segment is the pale variant. A discarded session shows a "Discarded" tag instead of a track.
5. At 440 × 640 at least three session rows are visible without scrolling, and no text overflows horizontally; long paths ellipsize (the full path is available as a tooltip and to screen readers).
6. Colors, sizes and spacing use the tokens above, in light and dark theme; all text meets 4.5:1.
7. Every control keeps its US00005 keyboard order, focus outline and accessible name; the track has the screen-reader text from Scope.
8. `tray/src/model.rs` unit tests cover any new or changed wording (reason lines per state, track segment count, stage-of-n text).

## Implementation plan

<!-- GUIDE: Written by /implementation-plan. -->

**Note for the planner and developer:** open frames p. 3, p. 4 and p. 13 as images before writing the plan (render with `pdftoppm -f 3 -l 4 -png -r 96 "06_UX/Dispatch Mesh desktop app.pdf" frame`, then view the PNGs), and compare a screenshot of the running window against them before handing to review. Text extraction from the PDF or HTML returns nothing.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

## As-built notes

## Follow-ups

## References

- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app.pdf]] (pp. 3, 4, 13)
- Screenshot of the US00005 window, 2026-11-06 (Kai)

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
