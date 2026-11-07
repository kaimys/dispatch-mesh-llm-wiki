---
id: US00012
type: story
status: "In progress"
priority: medium
rank: 3750
version_target: v0.3.0
size:
assignee: Kai
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
- **Newer than the PDF (canvas, 2026-11-06):** the last stage is labelled "Added to shared history" everywhere, and a workspace without a remote shows a three-segment track (no "Shared as draft"). The PDF export predates both; the PNGs were re-rendered on 2026-11-07 and show them (with Inter standing in for IBM Plex).
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
  > Amended 2026-11-07 (review F4): no app mark in the top row; the native title bar shows it.
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
  > Amended 2026-11-07 (review F4): the top row carries **no app mark**. The native title bar already shows it, and the design spec as revised on 2026-11-07 drops it from the row. The row holds the workspace name only.
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

**Planning decision (Kai, 2026-11-07).** Frozen [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]] AC 1 and AC 3 require each row to show the session's start time and its requested purpose, but the frame's three-line row shows neither. Decision: the meta line reads "N files · started 10:02 · last saved 2 min ago" (ellipsized). A session with a recorded purpose gets one more ellipsized line, "Requested purpose: …" (12, `text-3`), under line 2; rows without a purpose keep three lines. US00005's criteria stay met, so no correction to US00005 is needed.

## Acceptance criteria

1. With one workspace and an open session, the window opens at 440 × 640 (fitted to a shorter screen) and matches frame p. 3 in structure below the native title bar: a top row with the app mark and workspace name, the path, one state card holding glyph, label, reason, actions and last sync, the Activity / Conflicts nav, then the activity list. Checked by a side-by-side screenshot recorded in the test plan.
   > Amended 2026-11-07 (review F4): the top row holds the workspace name without an app mark (the native title bar shows it), per the revised design spec.
2. Each of the six summary states renders like its frame on p. 4: card ground and border, glyph shape, label, the reason and reassurance lines from Q8, and the action (primary **Open conflicts** / **Finish session** for attention reasons, secondary **Finish session** for Saved on this device and Shared as draft, none otherwise). State is never shown by color alone: glyph shape and label differ per state.
   > Amended 2026-11-07 (review F5): **Finish session** is a secondary action in every state while a session is open, unless it is the primary one (plan amendment in step 2; US00005 behaviour). The three working states (Saved on this device, Shared as draft, Finishing up) share one working glyph, as Scope and the design spec say; their labels differ. So state is never shown by color alone: the glyph shape differs between attention, offline, working and up to date, and the label differs for every state.
3. A session row shows avatar, author ("You" for this device's sessions) and meta ("N files · started … · last saved …", no device) on line 1, the title or files on one ellipsized line 2, "Requested purpose: …" on one ellipsized line only when a purpose is recorded, and the segmented track with the stage label last. The stage name appears exactly once per row.
   > Amended 2026-11-07 (Kai, after review): a published session's summary title is **not truncated**; it wraps over as many lines as it needs. A file list stays on one ellipsized line. Hovering a row shows a tooltip with the author's full name and the changed files, one per line (at most 20, then "… and N more"); screen readers get the changed files of a published row as text. A long author name ellipsizes and leaves room for the meta (review F1).
   > Amended 2026-11-07 (review re-check F3): the clipped meta line and the requested-purpose line keep their own tooltip with their full text. Hovering anywhere else on the row shows the author-and-files tooltip.
4. The track has 4 segments with a remote and 3 without. Filled segments match the stage, and a published session shows all segments in the published color. A session from another device has a pale first segment. A discarded session shows a "Discarded" tag instead of a track.
   > Amended 2026-11-07 (Kai, review F3): a session from another device **keeps its pale first segment after publication**, followed by published-color segments, as frames 3a/3b draw it. The track is 112 px wide with four segments and 84 px with three (review F6).
5. At 440 × 640 at least three session rows without a purpose are visible without scrolling, and no text overflows horizontally. Long paths ellipsize; the full path is available as a tooltip and to screen readers.
   > Amended 2026-11-07 (Kai, review re-check F2): the three-row guarantee holds for rows whose title or file list fits on one line. A full summary title matters more than a third visible row, so long titles that wrap may push later rows below the fold; the list scrolls.
6. Colors, sizes and spacing use the design spec's tokens, in light and dark theme; all text meets 4.5:1. Text renders in the bundled IBM Plex Sans / Mono, and the licence file ships with them.
7. Every control keeps its US00005 keyboard order, focus outline and accessible name; the track has the screen-reader text from Scope. The Conflicts tab has the accessible name "Conflicts, N unresolved".
   > Amended 2026-11-07 (Kai, review F7): except the old "‹ All workspaces" link, which becomes the top-row workspace switcher with the accessible name "<workspace>. Show all workspaces", for example "Team Wiki. Show all workspaces". The name starts with the visible text (WCAG 2.5.3, label in name). The switcher is a new control, not a rename; AC 7 still holds for every other control. With one workspace the name is plain text, not a control.
8. The **Conflicts** tab shows the unresolved count as a badge and the Q2 placeholder; **Open conflicts** on the attention card switches to it.
9. Avatar colors follow Q9: the same email always gets the same color, "You" is neutral grey, and white initials meet 4.5:1 in both themes.
10. CLI `status` includes `device` for every session; all existing fields are unchanged.
11. `tray/src/model.rs` unit tests cover the new and changed wording (reason and reassurance lines per state and reason, Finishing up for one and several sessions, track segment count, stage-of-n text, "You" vs. author, the received first segment, the avatar color rule, the Conflicts placeholder).
    > Amended 2026-11-07: also the row tooltip, the wrapping title flag, the received published track and the vertical window fit (review F2).

## Implementation plan

**Note for the planner and developer:** read [[Dispatch Mesh desktop app — design spec]] (*Status window*, *Tokens*, *Glyphs*) and open `06_UX/frames/03a-status-window.png`, `03b-summary-states.png` and `06-tokens-components.png` as images before writing the plan. Before handing to review, compare a screenshot of the running window against them, as in the spec's *How to check a build*. Text extraction from the PDF or HTML returns nothing.

### Ground truth that shapes the plan

Checked on 2026-11-07 against `main` at `403b165` (US00005 done) and the design spec as updated on 2026-11-07, which adds exported SVG glyphs. Since `c03cbf8`, `dispatch/invariants.md` says that UI work follows that spec.

- **Window code.**
  - `tray/ui/app.js` (280 lines) builds the window with `el()` and `textContent` only. `glyph(tone)` draws the *tray mark* with a corner badge, not the spec's status glyphs.
  - `card()` puts label, reason, `note`, an attention **Finish session** and **Retry** in one column. `activity()` has a caps heading "Recent activity" and a secondary **Finish session** beside it.
  - `sessionItem()` renders a bordered card, with the stage twice (text, then a ✓/●/○ list). `detail()` puts the name as `h1`, the path, and "Last synced" in a header above the card. Relative times are redrawn every 30 s.
- **Styles.** `tray/ui/app.css` has the old token subset: no `line-soft`, tints, `received`, card colors or `attention-text`. The body font is Segoe UI Variable.
- **Model** (`tray/src/model.rs`).
  - `WorkspaceView` has `label`, `tone` (`attention`/`offline`/`working`/`synced`/`neutral`), `reason`, `note` (offline note or `RETRY_NOTE`), `already_running`, `retry`, `last_sync` / `sync_label`, `open_session`, `action` (`finish_session`), `sessions` and `saved_copies`.
  - `SessionView` has `stage`, `steps` (label + `done`/`current`/`todo`), `files`, `author` (the name, always), times, `purposes` and `title`.
- **Attention reasons, as the engine writes them.**
  - Conflicts: "N unresolved conflict(s)" from `status_from`.
  - Capture: "Local capture waiting: …".
  - Sync: `SYNC_FAILURE` "Synchronization waiting: …".
  - Dependent edits: `RECEIVED_DRAFT_DEPENDENCY` "Edits depend on a received draft…", and "Dependent edits are saved locally…".
  - AI: `summary_failure`.
  - Paused synchronization: "…; synchronization paused" (history rewritten or branch deleted outside Mesh). This reason has no reassurance line under Q8, and it gets none.
- **Status.** `Status.device` and `has_remote` exist. `SessionStatus` has no `device` (Q5).
- **Glyphs.**
  - `06_UX/icons/status/status-<state>-<light|dark>.svg`: 24 × 24, with hard-coded token colors per theme, `role="img"` and `<title>`.
  - `06_UX/icons/tray/*`: the tray icons, which are out of scope. The tray's PNGs come from `scripts/tray_icons.py`.
- **Window config.** `tray/tauri.conf.json` sets 440 × 520 and min 380 × 360. The CSP is `default-src 'self'`, so self-hosted fonts and images load without a change, and `style-src 'self'` forbids inline `style` attributes, so per-row colors must be classes.

### Steps, in order

1. **Engine: `SessionStatus.device`** (`src/status.rs`). Add the session's device id as an additive field, which CLI `status` shows too. Amend ADR-0010 in the same change (dated: one field, for telling own sessions from received ones). It comes first because the model's "You" and received-segment rules test against it.
2. **Tray model** (`tray/src/model.rs`), pure and unit-tested; all wording lives here:
   - **`WorkspaceView` gains:**
     - `glyph`: `up-to-date` / `working` / `offline` / `needs-attention`, or `none` for already running;
     - `body`: the reason line. Attention and offline get the engine's `reason`, and working states get their sentence from the spec / Q8. Finishing up says "Adding your finished session to shared history." or "Adding your N finished sessions to shared history.", where N counts own sessions in that stage;
     - `reassurance`: Q8's line chosen by reason. Conflicts are recognized when `unresolved_conflicts` is non-empty, AI failure when the reason equals `summary_failure`, and the rest by their engine prefixes or constants; any other reason gets `None`. The sync-failure reassurance replaces `RETRY_NOTE`, which goes away. The offline note becomes the offline reassurance;
     - `conflict_count` and `conflicts_placeholder` (Q2);
     - `action` gains `open_conflicts` for the conflict reason;
     - `finish`: `primary` (attention, dependent edits), `secondary` (Saved on this device, Shared as draft, with an open session) or `None`. This moves the "where is Finish session" rule out of `app.js`.

       > ⚠️ Amended 2026-11-07, at the start of development: `finish` is `secondary` **whenever a session is open** and the card has no primary **Finish session**. That includes attention for other reasons and offline, not only Saved on this device and Shared as draft. US00005 offers **Finish session** in those states, and its model test asserts it ("Finish session stays available"). Limiting it to two states would silently remove frozen behaviour. The spec's table lists the actions a state *adds*; it does not forbid **Finish session**.
   - **`SessionView` changes:**
     - `author` reads "You" when `device == Status.device`; `own: bool` is added;
     - `avatar: { initials, color }`: initials are up to two letters of the author's name, and the "You" row uses the user's own name, as in frame 3a ("AL"). `color` is 0–5, from a stable FNV-1a hash of the lower-cased email, or `you`;
     - `meta`: "N file(s) · started <t> · last saved <t>". The model gives the count text, and the window fills in the times, as today;
     - `line` (title, else the files joined), plus `purpose: Option<String>` (the first purpose, followed by " +N more" when there are several);
     - `segments`: a list of `working` / `synced` / `received` / `empty`, with 4 or 3 entries. A published session is all `synced`; a received session's first segment is `received`;
     - `stage` (named once); `stage_text` "Stage: Shared as draft, 2 of 4" for screen readers;
     - `discarded: bool` replaces the empty `steps`. A discarded session shows a "Discarded" tag instead of the track. This tag replaces US00005's "No changes kept" stage label in the window, while `stage_label` stays for the menu and tooltips.

     `steps` is removed.
   - **Avatar palette** as constants `AVATAR = ["#2457C5", "#7A4FB0", "#2F6B4F", "#B4490B", "#3D3F44", "#0E6E7E"]` plus `YOU = "#3D3F44"`, with the same values in both themes, since all are dark enough for white initials. The palette lives in Rust so a test can check it (below); `app.css` mirrors it as classes `.avatar.c0`–`.c5` and `.you`.
3. **Glyphs** (`tray/ui/glyphs/`). Copy the eight status SVGs (four states × light/dark; `paused-reserved` is not needed) verbatim from `06_UX/icons/status/`, with a short `SOURCE.md` naming the origin and date.
   - They render as `<picture><source srcset="…-dark.svg" media="(prefers-color-scheme: dark)"><img src="…-light.svg" alt="<label>"></picture>`, at 28 px in the card and 22 px in the several-workspaces list.
   - The `alt` text is the state label (the spec's accessible name).
   - The tray-mark `glyph()` stays only for the app mark in the top row (16 px, no badge) and for the neutral already-running state.

   Copying beats re-typing paths: the artwork is exactly the export, and the wiki is not part of the repository, so the window needs its own copy.
4. **Fonts** (`tray/ui/fonts/`). IBM Plex Sans 400/500/600 and IBM Plex Mono 400/500 as woff2, from the official IBM Plex release, pinned to one version, with `OFL.txt` and a `SOURCE.md` giving the version, URL and SHA-256 of each file. `@font-face` in `app.css` with `font-display: block` (local files, so there's no flash); the fallbacks are `"Segoe UI", system-ui` and `Consolas, ui-monospace`.
5. **Window size** (`tray/tauri.conf.json`): 440 × 640, min 380 × 360. In `show_window` (`tray/src/main.rs`), if the monitor's work area is shorter than the window, set the height to the work area. That is the only Rust UI change.
6. **Styles** (`tray/ui/app.css`):
   - **Tokens:** replace the token block with the spec's full table, light and dark: `ground`, `surface`, `line`, `line-soft`, `line-strong`, `text`–`text-3`, `action`, the status tones with their tints, `received`, `offline-card` (ground and border), `attention-card` (ground and border), `attention-text` and `presence`.
   - **Type, spacing, radius:** the type scale, spacing and radius follow the spec.
   - **Components:** primary, secondary and focus buttons, the count badge, the tag, the avatar, the top row, the path, the state card per tone, the section nav and the session row.
   - **Track:** segments of 4 px height, radius 2, gap 3, 28 px each.
   - **Ellipsis:** one-line ellipsis on the path, the row's line 2 and the purpose.
7. **Window structure** (`tray/ui/app.js`, `index.html`):
   - **Top row:** the app mark and the workspace name. With several workspaces the name is a button with a chevron that returns to the workspace list, replacing "‹ All workspaces".
   - **Path:** one line, with the full path in `title` and the accessible name.
   - **State card:** row 1 holds the glyph, the label (17/600, `attention-text` for attention), the body and the reassurance (13/500). Row 2 holds the buttons on the left and the sync line on the right. **Open conflicts** (primary) switches to the Conflicts tab.
   - **Section nav:** `role="tablist"` with **Activity** and **Conflicts** plus the badge. The badge's accessible name is "Conflicts, N unresolved". Arrow keys move between tabs. The selected tab is remembered per workspace in memory.
   - **Activity tab:** divider-separated rows per the session-row spec. The track is `aria-hidden` and each row carries `stage_text`. Below it, saved copies keep their disclosure, restyled.
   - **Conflicts tab:** the placeholder text.
   - **Already running and stopped** states keep their current content in the new card style.
   - **Kept as is:** every existing `data-key`, the focus restoration and the accessible names.
8. **Design spec** (wiki; Kai owns the page, and the edit is proposed in the same change). Bring the spec in line with the decisions, because the invariants make it binding for UI work:
   - native title bar (Q1);
   - nav now, with the placeholder (Q2);
   - Plex bundled (Q3);
   - "You" and no device names (Q5, Q6);
   - the Q8 lines, including Finishing up;
   - the avatar rule and palette (Q9);
   - the meta and purpose lines (planning decision).
9. **Docs:** README (`status` gains `device` per session; the tray section describes the restyled window), the AGENTS.md repository map (`ui/` now holds `glyphs/` and `fonts/`), and the ADR-0010 amendment (step 1).

**Order and why.**
- **Step 1 before 2:** the model's "You" and received rules are tested against the new field.
- **Steps 3–4 before 6–7:** the stylesheet and markup reference the files.
- **Step 2 before 7:** the window only renders what the model decides, which keeps the wording testable.
- **Step 8 with the code:** the spec never contradicts a merged build. US00006 then plans against the updated spec.

### Tests

- **Engine** (`tests/cli.rs` or `tests/engine.rs`): `status` has `device` for an own and a received session, and every existing key is unchanged.
- **Tray model** (unit tests in `tray/src/model.rs`):
  - the reason and reassurance lines per state and per reason, including a "synchronization paused" reason with no reassurance and an unknown reason with none;
  - Finishing up for one and for three sessions;
  - `finish` placement per state;
  - the `open_conflicts` action;
  - segments for every stage, with and without a remote, published (all synced) and received (first `received`);
  - `stage_text`;
  - "You" vs. the author's name; the initials (one word, two words, an empty name);
  - the avatar color: stable for one email whatever its case, and spread across the six colors for a sample of emails;
  - the purpose line with zero, one and several purposes;
  - the meta file count, singular and plural;
  - the Conflicts placeholder with zero and with N conflicts;
  - removal of `RETRY_NOTE`, and US00005 AC 10's sentence now inside the reassurance.
- **Contrast check** (new unit test in `tray/src/model.rs`, which reads `../ui/app.css` with `include_str!`):
  - every text token passes 4.5:1 on `ground` and `surface`, in light and dark;
  - `attention-text` passes on `attention-card`;
  - white passes on each avatar color;
  - the CSS avatar classes use exactly the Rust palette.

  The stylesheet becomes testable by the existing `cargo test` with no new tool.
- **Host** (`tray/tests/host.rs`): unchanged, apart from adjustments to renamed view fields; no new host behavior.
- **Manual** (for `/test-plan`, following the spec's *How to check a build*):
  - screenshots at 440 × 640 against 3a and each state in 3b, light and dark, with and without a remote;
  - three rows visible;
  - long paths ellipsized, with the full path in the tooltip;
  - Plex actually rendering (DevTools computed font);
  - a short screen;
  - Narrator on the card, tabs, track text and buttons;
  - keyboard order unchanged from US00005.

### External changes and migration

- `status` JSON: `sessions[].device` added; no state or format change.
- `WorkspaceView` (internal, between the tray and its own window): fields renamed or added. `steps` and `note` are replaced. No external consumer.
- New shipped assets: eight SVGs and five woff2 files (about 250 KB) with `OFL.txt`. No new crate and no CSP change.

### Wiki pages this makes wrong

- [[Dispatch Mesh desktop app — design spec]]: step 8.
- [[ADR-0010 — Report workspace status and session stages separately]]: amended in step 1.
- README and AGENTS.md: step 9.
- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]: still true (start time and purpose stay visible); nothing to annotate. Its retry note's wording now sits inside a longer sentence, which still meets its AC 10.
- [[Story - US00006 - Conflict list and resolution in the status window|US00006]]: its plan already assumes this ticket's nav, tokens and "You" rule.

### Risks and rollback

- **Font licensing and supply.** Use only the official IBM release under the OFL, shipped with its licence, with checksums recorded.
- **WebView2 `<picture>` with `prefers-color-scheme`:** supported. If the window ever follows an app theme setting instead of the system one, glyph switching needs a class-based approach (out of scope).
- **Contrast of derived dark tokens** (`line-soft`, tints, `received`, card colors are "derived, still need checking"): the contrast test catches text pairs. Non-text segment contrast is checked manually.
- **Row height:** a purpose line makes some rows taller. Criterion 5 counts rows without a purpose.
- **Rollback:** revert the commits. `SessionStatus.device` is additive and ignored by older readers.

### Decisions for review

1. Status glyphs ship as verbatim copies of the exported SVGs, switched by `<picture>` and `prefers-color-scheme`, rather than re-drawn from paths in JS.
2. The avatar palette lives in Rust and is mirrored as CSS classes, checked by a test that reads `app.css` (no inline styles because of the CSP).
3. Reason → reassurance mapping keys off the engine's own constants and prefixes. Reasons without a mapping, such as "synchronization paused", get no reassurance rather than a guessed one.
4. A discarded session shows the "Discarded" tag (spec) instead of US00005's "No changes kept" label; the menu and tooltip wording are unchanged.
5. Several purposes show as the first one plus " +N more" on one line.
6. With several workspaces, the top-row name with its chevron returns to the workspace list. The list itself (3h) keeps its current layout, apart from the new glyphs.
7. Tray icons stay as they are. Whether `scripts/tray_icons.py` matches the exported `06_UX/icons/tray/*.svg` is a candidate follow-up, not part of this ticket.

### Size

Order of magnitude: about three to four focused days. Roughly one day for the model and tests (including the contrast check), two days for the styles and structure, and half a day for assets, config and the spec update. Not a commitment.

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

## As-built notes

**2026-11-07, development (branch `us00012-status-window-design`, commits `8e265dd`, `ee9afa5`, `fbb9529`, `2f2b73a`).** Built as planned, with the plan amendment on **Finish session** (step 2) and these deviations:

- **Top row has no app mark.** The design spec was revised on 2026-11-07, after the plan: the native title bar already shows the mark, and the row has `line-soft` borders above and below. The build follows the revised spec. The tray-mark drawing remains only as the glyph for the neutral "already running" state.
- **`on-action` and `on-attention` tokens.** The spec asked for white text on primary buttons and on the count badge. In dark theme, white is 2.4:1 on `#7FA6FF` and 2.0:1 on `#FF9E57`, so the contrast test failed it. Dark theme uses `#14161A`, as US00005's window already did. Both tokens were added to the spec's color table.
- **Wording added beyond Q8.**
  - **No remote:** "Saved on this device" without a remote reads "Your latest edits are captured. They join history after 5 minutes without edits, or when you finish the session." The spec's sentence promises a draft that is never shared there.
  - **Circular dependency:** the engine's circular-dependency reason ("Dependent edits are saved locally …") gets the dependent-edits reassurance, like "Edits depend on a received draft".
  - **One conflict:** the Conflicts placeholder reads "Review it" for one conflict and "Review them" for several.
  - **Spec updated:** all three were added to the spec.
- **Engine constants.** The capture reason prefix (`status::CAPTURE_WAITING`) and the circular-dependency reason (`engine::CIRCULAR_DEPENDENCY`) became constants, so the model matches on the engine's own text instead of copies of it.
- **Fonts.** IBM Plex Sans and Mono 1.1.0 (the same release for both) from the official `@ibm/plex-*` npm tarballs, each checked against the registry's sha512 before extraction. File hashes are in `tray/ui/fonts/SOURCE.md`. The licence is `OFL.txt`.
- **Track.** Segments are 25 px wide with 3 px gaps (109 / 81 px in total), close to the spec's 28 px per segment including the gap.

**Verification.**
- **Gates:** `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and the full `cargo test` passed on Windows. After the last wording change, the tray suites were run again: 27 model tests and 10 host tests.
- **New model tests:** the reason and reassurance per state and per reason, Finishing up for one and several sessions, Finish placement, segments, the received segment, stage text, "You", initials, avatar colors, the purpose line and the placeholder.
- **Contrast check:** a stylesheet test checks 15 text pairs in both themes, white initials on every avatar color, and that the CSS avatar classes match the Rust palette.
- **CLI and engine:** CLI `status` and an engine test check `device` for own and received sessions.

**Run on Windows at 150 % scaling.** The demo was a disposable workspace pair (`target/us00012-evidence/demo.sh`): a remote, Alice and Bob, one text conflict, a received draft, a published session and an intent. The debug tray ran on Bob's workspace. Screenshots are in `target/us00012-evidence/`.
- **Layout:** the window opens at 440 × 640 content. It shows the top row, the ellipsized path, and the attention card with its glyph, reason, reassurance, the primary **Open conflicts**, the secondary **Finish session** and "Last synced just now". Below it are the tabs with the badge, then three rows.
- **Rows:** "You" with the purpose line, Alice's received draft with the pale first segment, and the published session all green.
- **Keyboard:** Tab and Enter on **Open conflicts** switch to the Conflicts tab, with the focus on the tab and the dashed placeholder showing.
- **Earlier screenshot:** one attempt showed the page scrolled by the top row's height. It did not reproduce on a fresh start; that attempt had first focused another window.

**Not checked yet** (for `/test-plan`): dark theme, a screen shorter than 640, the several-workspaces list, the already-running state, Narrator, and side-by-side comparison with frames 3a and 3b for each summary state.

**2026-11-07, independent code review (`2f2b73a`).** F1 fails AC 3 and 5, F2 and F4 fail AC 1, F3 fails AC 4, F5 fails AC 2, F6 fails AC 6 and F7 fails AC 7. The ticket stays in `In progress`. The owner needs to resolve the published received-track rule before the related fix. See the fix list under *Code review*.


**2026-11-07, review fix round (commits `1a8f05b`, `4139a7c`).** All nine findings of the 2026-11-07 review are addressed; `open_findings` is cleared to empty until the review's re-check. Contract changes are dated amendments under Q1, Scope and AC 1–4, 7 and 11. Kai decided F3 and F7 and asked for two changes.

- **F1 long author:** `.author` shrinks and ellipsizes (at most 55 % of the line), and the meta takes the rest. The row tooltip carries the full name.
- **F2 short screen:** the geometry is now a pure function, `model::fit_vertically`, which shortens the window to the work area and then moves it so its top and bottom lie inside. `show_window` applies both. Unit tests cover an offset window on a short work area (the review's case), a window running past the bottom, a work area that doesn't start at 0, and one above the work area.
- **F3 received track (Kai):** the received first segment stays pale after publication. The model test asserts `received, synced, synced, synced`. AC 4 and the spec are amended.
- **F4 app mark:** no code change. Q1, Scope and AC 1 are amended to the revised spec: the top row has no app mark, and the native title bar shows it.
- **F5 actions and glyphs:** no behaviour change. AC 2 is amended: **Finish session** is secondary whenever a session is open, and the working states share one glyph. A model test asserts an open session while offline.
- **F6 track width:** the track is 112 px with four segments and 84 px with three, with segments sharing the width evenly and 3 px gaps.
- **F7 switcher name (Kai):** keeps "<workspace>. Show all workspaces", recorded as an exception under AC 7. *For `/test-plan`:* check the switcher's announced name and its place in the keyboard order instead of US00005's "‹ All workspaces".
- **F8 font sources:** the tarball URLs are in `tray/ui/fonts/SOURCE.md`.
- **F9 README:** the README says reassurance shows when offline and for the attention reasons that have one.
- **Kai's change 1, summary title:** a published summary title wraps instead of being cut off (`line_is_title`); a file list stays on one ellipsized line.
- **Kai's change 2, row tooltip:** hovering a session row shows the author's full name and "Changed files:" with one file per line, at most 20, then "… and N more". For published rows, screen readers get "Changed files: …" as text, because they don't read tooltips. The design spec's *Session row* is updated for both changes.

**Verification.**
- **Gates:** `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and the full `cargo test` pass on Windows (153 tests, 29 of them tray model tests).
- **Run:** the debug tray on a demo whose collaborator name is one long unbroken word (`target/us00012-evidence/long-author-440.png`, `long-author-380-tooltip.png`).
  - At 440 and at the 380 minimum width there is no horizontal overflow, the name ellipsizes, and the file count and start time stay visible.
  - The received published row shows one pale segment and three green ones.
  - Hovering shows the tooltip with the full name and the files.
- **Not exercised natively:** a wrapping long title (the demo's local summary titles are short) and the off-screen fit. The fit is covered by unit tests, and both belong in the test plan.

**2026-11-07, independent fix re-check (`4139a7c`).** The offset fit is corrected for restored windows, and the native maximize case still fails AC 1/Q1. Wrapped titles fail the current three-row requirement in AC 5, and clipping without full-text hover fails AC 3 for timestamps and requested purposes. Keep the ticket in `In progress`. See the re-check and fix list under *Code review*.


**2026-11-07, re-check fix round.** The three findings of the re-check at `4139a7c` are addressed; `open_findings` is cleared to empty until the next re-check.

- **F1 maximize:** `show_window` now shows and un-minimizes the window first, then fits it only when it is **not maximized**, and focuses it last. Fitting a maximized window would have resized or moved it, which undoes the maximize. Fitting before `unminimize` had a second problem: a minimized window reports a parking position far off screen, which the old order would have "fitted". *Not exercised natively:* reopening a maximized window needs a click on the tray icon, which the test harness here can't do. It is for `/test-plan`, together with the offset short-screen case.
- **F2 three rows (Kai):** "full commit messages matter more than three rows in the list; you can scroll". No code change. AC 5 is amended, and so is the design spec's *Status window* item 5.
- **F3 clipped text:** the meta line and the purpose line get back their own tooltips with the full text, and the rest of the row keeps the author-and-files tooltip. Checked in the running tray at 380 px with a long author name: hovering the clipped meta shows "2 files · started 12:51 · last saved 25 min ago", and hovering line 2 shows the author and files (`target/us00012-evidence/meta-tooltip-380.png`).
## Code review

### 2026-11-07 — branch us00012-status-window-design at 2f2b73a

Reviewed the four US00012 commits in `main..HEAD`, their tests, ADR-0010 and the current desktop design spec. No open PR. Independently ran the Windows mechanical gates. `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and the full `cargo test` passed (151 tests). Compared the existing Windows attention screenshot with frames 3a and 3b. A separate headless Chromium fixture exercised the unmodified window assets at 440 × 640. Measurements are in `target/us00012-review/layout.json`. The fixture uses synthetic tray payloads. Native short-screen behavior was checked in source, and the full Windows visual and Narrator pass belongs to the manual test plan.

**Verdict.** Blocked with seven blocking findings and two non-blocking findings. Keep US00012 in `In progress`. The ticket cannot advance to `/test-plan`.

1. **Blocking · AC 3 and 5 · A long author name clips the author and hides the metadata.** `tray/ui/app.css:129`, `tray/ui/app.js:180`. A received session with an unbroken collaborator name leaves the name at its intrinsic width because `.author` has `flex: none`. At 440 × 640 the measured name extended to x = 511.8 in a 425 px content viewport and the file count and timestamps had zero width. The page's hidden horizontal overflow clips the author. Give the author a shrinkable, ellipsized allocation and a full-name tooltip while reserving room for metadata. Exercise a long collaborator name at both default and minimum width.
2. **Blocking · AC 1 · The short-screen resize can leave the bottom off-screen.** `tray/src/main.rs:95–108`. The function sets the outer height to the work-area height but leaves the window's origin in place. On Windows the underlying resize preserves position. A window whose top is 40 physical pixels below the work-area top therefore still extends 40 pixels below the work area after the resize. Fit the outer rectangle against the work area's origin as well as its size, preserving the configured minimum and native window behavior. Verify from an offset position on a short work area.
3. **Blocking · AC 4 · Decide what the first segment means after a received session is published.** `tray/src/model.rs:493` and the assertion at line 1182. When a received draft becomes published, the published branch wins and turns its pale first segment green. AC 4 and the design spec say both that published fills every segment with the published color and that a received session starts pale. Frames 3a and 3b show the received published row with a pale first segment and three green ones. Confirm whether device origin stays visible after publication or the published color takes precedence. Align AC 4 and the spec, then make the implementation and its test assert that decision.
4. **Blocking · AC 1 · The top-row app mark remains in the contract.** `tray/ui/app.js:266`. In a single workspace, only the workspace name appears below the native title bar. The as-built note records that the build follows the revised design spec, which puts the app mark in the native title bar. Q1, Scope and AC 1 still require another mark in the content row. Reconcile the ticket through `/update-ticket` with the current design and retain the native title-bar mark.
5. **Blocking · AC 2 · The written action and glyph rules disagree with the agreed layout.** `tray/src/model.rs:348–357`, `tray/src/model.rs:684`. With an open session and an offline remote, the build keeps secondary **Finish session** per the dated plan amendment and the design spec. AC 2 says there is no action in Offline. The three working states also share one working glyph per Scope and the design spec, while AC 2 says the glyph shape differs per state. Reconcile AC 2 with the open-session rule and the shared working glyph, retaining the distinct text labels. Assert the amended action rule with an open offline session in the model test.
6. **Blocking · AC 6 · The track misses the design's total widths.** `tray/ui/app.css:133–134`. The 25 px segments and 3 px gaps produce 109 px with a remote and 81 px without one. The design spec sets 112 px and 84 px, respectively. The deviation is recorded in the as-built notes. Set the total widths and divide the available space evenly among the segments while keeping the specified gap.
7. **Blocking · AC 7 · The workspace switcher changes an existing accessible name.** `tray/ui/app.js:269–274`. With several workspaces, the previous button's name came from **‹ All workspaces**. The replacement uses **Team Wiki. Show all workspaces**, for example. The planned replacement explains the visible name, while AC 7 and plan step 7 require existing accessible names to stay. Reconcile the explicit name requirement with the planned switcher, and verify its announced name and its place in the keyboard order.

8. **Non-blocking · Font provenance is missing the source URLs.** `tray/ui/fonts/SOURCE.md:5` identifies the IBM packages and release and records matching hashes for all five files. Step 4 also asks for the URL. Add the pinned package or tarball URLs alongside the hashes.
9. **Non-blocking · README overstates when reassurance appears.** `README.md:145` says the card has a reassurance line for attention. A paused synchronization reason has none by design. Say reassurance is shown for attention reasons with an applicable message.

**What held up.** The device field is additive in one shared status builder, so the CLI and tray use the same device identity without a storage or transport change. Activity still sorts unsettled sessions ahead of the ten most recent settled sessions. The model owns the stage text and the DOM hides the track and duplicate visible label from assistive technology. The fixture confirmed that **Open conflicts** switches to the named tab and moves focus there, and that Left Arrow moves back to Activity. Three normal rows fit in the fixture, long paths retain their full text and tooltip, the used Plex faces loaded, the font hashes match their record and all eight SVGs match the design exports. The diff contains only US00012 commits.

**Before advancing.**

- Use `/update-ticket US00012` to reconcile F3, F4, F5 and F7, with the ticket owner's decision on F3.
- Use a fresh `/develop US00012` session to fix the layout and window fit in F1, F2 and F6, apply any code changes required by the contract decisions, and verify the long-name and short-screen cases.
- Re-check this review against the fixed commit before running `/test-plan`.

### 2026-11-07 — branch us00012-status-window-design at 4139a7c

Re-checked the original nine findings against `1a8f05b` and `4139a7c`, the dated contract amendments, the desktop design spec and ADR-0010. No open PR. `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and the full `cargo test` passed on Windows (153 tests). Ran the unmodified window assets in Chromium with synthetic tray payloads at 440 × 640 and 380 px wide. Measured decorated-window geometry with an isolated Win32 test window and traced the tray's resize and move calls through the pinned Tao runtime. Evidence is in `target/us00012-recheck/layout.json` and `target/us00012-recheck/win32-geometry.json`. The Win32 probe measured geometry independently of the running tray.

**Verdict.** Blocked with three blocking findings and zero non-blocking findings. Keep US00012 in `In progress` and hold `/test-plan`.

1. **Blocking · F2 re-check · AC 1 and Q1 · Fitting the window can undo native maximize.** `tray/src/main.rs:84–116`. The fit now handles the original offset case for a restored window. It also runs on every Open status window action, including when the window is already maximized. The Windows geometry probe measured a maximized outer rectangle of `[-11, -11, 2571, 1379]` against a work area of `[0, 0, 2560, 1368]`. Windows includes invisible borders in the outer rectangle. The fit asks for both a resize and a move in this case, and Tao clears the maximized flag on either call. Guard the native maximized state and apply the fit to restored geometry. Exercise both the offset short-screen case and reopening an already maximized window through the tray.
2. **Blocking · AC 5 · Full published titles can push the third row below the viewport.** `tray/ui/app.js:185`, `tray/ui/app.css:133`. Three purpose-free published rows with provider-valid titles under the 160-character limit wrap to three lines each. At 440 × 640 the third row ends at y = 718.6, with its stage below the fold. The same fixture with short titles ends at y = 605.5. The amended AC 3 asks for titles to wrap without truncation and AC 5 still requires three complete purpose-free rows at the default size. Decide whether the three-row requirement applies to the single-line reference layout or must also hold for wrapped titles. Align AC 5 and the design spec with that decision, then assert it in the layout check.
3. **Blocking · AC 3 and prior US00005 AC 1/3 · The row tooltip removes access to clipped timestamps and purposes.** `tray/ui/app.js:177–187`. With a long author name and a two-day-old session at 440 × 640, the metadata needs 371 px but gets 160 px. The last-save value is clipped. The child tooltip used to expose the full text. Hovering now inherits the row's author-and-files tooltip. The change also removed the full-text tooltip from the intentionally ellipsized requested-purpose line, so long purposes cannot be read visually from the window. Restore a way to inspect the full metadata and purpose while retaining the author-and-files tooltip on the rest of the row. Re-check an older session and a long purpose at both widths.

**Verified fixes.** F1's author overflow is fixed at both widths and the full author remains in the row tooltip. The track measures 112/84 px and the first segment of a received published row stays pale (F3/F6). The dated amendments now match the title-bar icon, the shared working glyph and open-session Finish action, and the workspace switcher's accessible name (F4/F5/F7). The model test covers finishing an open session offline. The font URLs and the README's reassurance wording address F8/F9. The wrapping title stays within its row horizontally and published rows carry the changed files as screen-reader text.

**Before advancing.** Reconcile the density criterion with the owner's title-wrapping decision through `/update-ticket`. In a fresh `/develop` session, preserve native maximize on reopen, keep clipped metadata and purpose inspectable, and implement any layout change the decision requires. Re-check the revised commit before `/test-plan`.

## Follow-ups

## References

- [[Story - US00005 - Workspace detail with Finish session and recovery information|US00005]]
- [[Claude Design briefing — desktop app]], [[Dispatch Mesh desktop app — design spec]], [[Dispatch Mesh desktop app.pdf]] (pp. 3, 4, 13), `06_UX/frames/`
- Screenshot of the US00005 window, 2026-11-06 (Kai)

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
