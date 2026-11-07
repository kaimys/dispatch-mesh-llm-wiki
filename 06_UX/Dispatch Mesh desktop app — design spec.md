---
type: design-spec
status: draft
owner: "[[Kai]]"
updated: 2026-11-07
---
# Dispatch Mesh desktop app — design spec

The design frames written out as text, so they can be built and checked without opening a picture. It goes with the [[Claude Design briefing — desktop app|design brief]] (what and why). This page covers what it looks like.

**For agents and developers:** [[Dispatch Mesh desktop app.pdf]] has no text layer and `Dispatch Mesh desktop app.html` is a compressed bundle. Text extraction from either returns nothing. Use, in this order:

1. This page: tokens, glyph paths, components, per-view rules.
2. `frames/*.png`: one image per frame. Open the image to see it.
3. `frames/source/*.dc.html`: the frame markup, with exact inline styles and sample data. It needs the design tool's runtime to render, but it reads as plain HTML. `03a2-conflicts-tab` and `03b-summary-states` mount the window frame by its original name (`Window`), so they don't render standalone.

The PNGs were rendered from the frame source on 2026-11-07 at 2× scale, so they match the source and this page. IBM Plex wasn't available to the renderer, so the PNGs show **Inter** in place of Plex Sans and **DejaVu Sans Mono** in place of Plex Mono: take the typeface from *Type* below, not from the pictures. When this page and a PNG disagree, this page wins. The PDF (export of 2026-11-05) predates the changes listed below.

## Frame index

| # | Frame | PNG | Source | PDF page | Built in |
|---|---|---|---|---|---|
| 1 | Tray icon set | `frames/01-tray-icons.png` | `frames/source/01-tray-icons.dc.html` | 1 | US00001 |
| 2 | Tray menu + Quit variants | `frames/02-tray-menu.png` | `frames/source/02-tray-menu.dc.html` | 2 | US00001, US00005 |
| 3a | Status window, 440 × 640 below the native title bar | `frames/03a-status-window.png` | `frames/source/03a-status-window.dc.html` | 3 | US00005, US00012 |
| 3a2 | Conflicts tab placeholder (until US00006) | `frames/03a2-conflicts-tab.png` | `frames/source/03a2-conflicts-tab.dc.html` | — | US00012 |
| 3b | Six summary states | `frames/03b-summary-states.png` | `frames/source/03b-summary-states.dc.html` | 4 | US00012 |
| 3c | Session detail + presence | `frames/03c-activity-presence.png` | `frames/source/03c-activity-presence.dc.html` | 5 | US00007, later |
| 3d | Conflicts list | `frames/03d-conflicts-list.png` | `frames/source/03d-conflicts-list.dc.html` | 6 | US00006 |
| 3e | Text conflict detail | `frames/03e-text-conflict.png` | `frames/source/03e-text-conflict.dc.html` | 7 | US00006 |
| 3f | Binary conflict detail | `frames/03f-binary-conflict.png` | `frames/source/03f-binary-conflict.dc.html` | 8 | US00006 |
| 3g | Already running + first run | `frames/03g-already-running-first-run.png` | `frames/source/03g-already-running-first-run.dc.html` | 9 | US00001 |
| 3h | Several workspaces + tray aggregation | `frames/03h-several-workspaces.png` | `frames/source/03h-several-workspaces.dc.html` | 10 | US00001 |
| 4 | Notifications | `frames/04-notifications.png` | `frames/source/04-notifications.dc.html` | 11 | US00008 |
| 5 | Guided setup (0.5 sketch) | `frames/05-guided-setup.png` | `frames/source/05-guided-setup.dc.html` | 12 | 0.5 |
| 6 | Tokens + components | `frames/06-tokens-components.png` | `frames/source/06-tokens-components.dc.html` | 13 | all |

`frames/source/canvas.json` holds the canvas layout and the note with the proposed answers to the brief's open questions.

## Changes since the PDF export

Made on the canvas on 2026-11-06. The PDF doesn't show them; the PNGs do:

- The last session stage is labelled **Added to shared history** everywhere (it was "In shared history" in 3a, 3b, 3c and 6).
- A workspace **without a remote** shows a **three-segment** stage track (Saved on this device → Finishing up → Added to shared history), "This device only" where the last sync would be, and only the user's own sessions (US00005 AC 2).
- The Paused row on the icon sheet (1) is drawn at full strength and marked RESERVED in text, instead of faded.

Refinement decisions of 2026-11-07 ([[Story - US00012 - Status window layout follows the design frames|US00012]] Q1–Q9, [[Story - US00006 - Conflict list and resolution in the status window|US00006]] planning), applied on the canvas the same day:

- Every window keeps the **native Windows title bar**, which carries the app icon. Below it the app's top row holds only the workspace name: no app mark and no hide button of its own. The window content opens at 440 × 640, fitted to shorter screens.
- The **Activity / Conflicts** tabs are there from the start. Until US00006, Conflicts shows a placeholder (frame 3a2).
- **No device** is ever shown: "Alice · 1 file · last saved 40 s ago", never "laptop" or a device id. Own sessions read **You**; sessions from another device get the pale first track segment.
- **Avatars** are always circles. A hash of the email picks one of six colors; "You" is always grey.
- New **reason and reassurance lines** (see *State card per summary state*). Offline has no Retry now button.
- **Conflicts:** buttons name the author, and **Review and choose** is secondary. A later edit disables both buttons until US00010. Image conflicts say "the other version stays kept".

## Tokens

### Color

| Token | Use | Light | Dark |
|---|---|---|---|
| `ground` | Window background | `#F7F7F5` | `#1B1C1E` |
| `surface` | Cards, menus, title row | `#FFFFFF` | `#242528` |
| `line` | Dividers, card borders, empty track segment | `#E3E2DE` | `#34363A` |
| `line-soft` | Row dividers inside lists | `#ECEBE7` | `#2D2F33` |
| `line-strong` | Secondary button border | `#C9C8C3` | `#4A4C51` |
| `text` | Primary text | `#1C1D1F` | `#ECECEA` |
| `text-2` | Body on cards | `#3D3F44` | `#C4C5C8` |
| `text-3` | Meta, timestamps, paths | `#55575C` | `#A2A5AA` |
| `action` | Primary button, links, focus ring | `#2457C5` | `#7FA6FF` |
| `synced` | Up to date glyph, published track | `#2F6B4F` | `#6FBF97` |
| `synced-tint` | Up to date glyph fill | `#E9F3EE` | `#1F3329` |
| `working` | Working glyph, track in progress | `#2457C5` | `#7FA6FF` |
| `working-tint` | Working glyph fill | `#EAF0FC` | `#1E2A44` |
| `received` | First track segment of a session received from another device | `#C9D6F2` | `#34466E` |
| `offline` | Offline glyph | `#4A5568` | `#A7ADB8` |
| `offline-card` | Offline card ground / border | `#F1F2F4` / `#DCDFE4` | `#25272B` / `#3A3D43` |
| `attention` | Attention glyph, conflict count badge | `#B4490B` | `#FF9E57` |
| `attention-card` | Attention card ground / border | `#FDF1E7` / `#F0C9A8` | `#3A2618` / `#5C3A22` |
| `attention-text` | Attention label | `#8A3608` | `#FFB37D` |
| `presence` | Recent-save presence ring | `#7A4FB0` | `#B793E6` |
| `on-action` | Text on a primary button | `#FFFFFF` | `#14161A` |
| `on-attention` | Text on the count badge | `#FFFFFF` | `#14161A` |

`on-action` and `on-attention` were added in US00012 (2026-11-07): white on the dark-theme `action` (`#7FA6FF`) and `attention` (`#FF9E57`) is about 2.4:1 and 2.0:1, so dark theme uses near-black text there. `tray/src/model.rs` tests every text pair of the shipped stylesheet against 4.5:1.

Light values are the ones used in the frames. Dark values marked on frame 6 are proposed; the others (`line-soft`, tints, `received`, card colors) are derived for this page and still need checking against 4.5:1 in the build. Every text color passes 4.5:1 on `ground` and `surface`, and each status tone passes 4.5:1 on its own card ground.

### Type

IBM Plex Sans (400/500/600) and IBM Plex Mono (400/500), bundled with the app as woff2 in `tray/ui/fonts/` with the SIL OFL licence file alongside (US00012 Q3). Fallbacks: Segoe UI / system-ui and Consolas.

| Role | Size / weight | Notes |
|---|---|---|
| Spec heading (frames only) | 28 / 600, −0.01em | not in the app |
| Detail heading | 18 / 600 | session title in 3c |
| State label | 17 / 600, line-height 1.3 | state card |
| Workspace name | 14 / 600 | top row |
| Body | 13 / 400, line-height 1.45 | reasons, rows, buttons |
| Meta | 12 / 400, `text-3` | times, counts, last sync |
| Caps label | 11 / 600, uppercase, 0.04em | "Requested purpose", "Current version" |
| Path | Mono 12 / 400, `text-3` | folder path, file paths |

### Spacing, radius, elevation

- Spacing scale 4 · 8 · 12 · 16 · 24 · 32. Window content padding 16; card padding 16; row padding 12 vertical.
- Radius 4 controls, 6 inner panels, 8 cards and windows, 999 pills and count badges.
- Lines 1 px. One shadow, for floating surfaces only (menus, toasts, popovers): `0 8px 24px rgba(20,20,20,0.12)`. No gradients, blurs or translucency, so any toolkit can draw it.

## Glyphs

SVG files: `06_UX/icons/tray/tray-<state>-<light|dark|template>.svg` (16 × 16) and `06_UX/icons/status/status-<state>-<light|dark>.svg` (24 × 24). States are `up-to-date`, `working`, `offline`, `needs-attention` and `paused-reserved`. In the tray icons the ring around the badge is cut out with a mask, so the icons are transparent on any taskbar. The template variants are black plus alpha, with the badge glyph knocked out.

### Status glyphs (state card, workspace list)

24-unit viewBox, drawn at 28 px in the state card and 22 px in lists. Outer path: `fill` = tint, `stroke` = tone, width 1.6, round joins. Inner path: no fill, `stroke` = tone, width 1.8, round caps and joins.

| State | Outer | Inner |
|---|---|---|
| Up to date | `M12 3a9 9 0 1 1 0 18a9 9 0 1 1 0-18z` | `M8 12.4l2.8 2.8L16.2 9.6` |
| Saved on this device, Shared as draft, Finishing up | circle as above | `M8.3 12a3.7 3.7 0 0 1 6.6-2.3M15.7 12a3.7 3.7 0 0 1-6.6 2.3M15.2 7.4v2.4h-2.4M8.8 16.6v-2.4h2.4` |
| Offline | circle | `M6.6 6.6l10.8 10.8` |
| Needs attention | `M12 3.5L21.2 19.6H2.8z` | `M12 9.6v4.4M12 16.9v0.05` |
| Paused (reserved, not in 0.3) | circle | `M10 9v6M14 9v6` |

The glyph has `role="img"` and the state label as its accessible name.

### Tray icon (frame 1)

16-unit master. Mark: nodes at (4,4), (12,4), (4,12), radius 2.2, joined by 1.4 strokes `M4 4L12 4M4 4L4 12M12 4L4 12`. Badge centred at (12,12): a knock-out circle r 4.6 in the taskbar color, a badge circle r 3.6, and a 1.3 stroke glyph in white (light taskbar) or `#14161A` (dark).

| State | Badge | Glyph path | Mark |
|---|---|---|---|
| Up to date | none | — | full |
| Working | `#2457C5` / dark `#7FA6FF` | `M12 10a2 2 0 1 1-2 2` | full |
| Offline | `#5B6170` / dark `#A7ADB8` | `M10.5 13.5l3-3` | 50 % opacity |
| Needs attention | `#C2540F` / dark `#FF9E57` | `M12 10v2.3M12 14.1v0.05` | full |
| Paused (reserved) | as offline | `M11 10.5v3M13 10.5v3` | full |

Sizes: Windows 16 and 32 px (light taskbar `#F3F3F3`, dark `#202020`); macOS 18 px template image (black + alpha, glyph knocked out). Tooltip "Dispatch Mesh — <state>[: <reason>]" is also the accessible name. Hand-hint the 16 px pixels before shipping.

## Components

- **Primary button:** `action` fill and border, `on-action` 13/500 text (white in light theme), padding 7 × 14, radius 4. **Secondary:** `surface` fill, `line-strong` border, `text`. Focus: 2 px `action` outline, offset 2. One primary button per view at most.
- **Count badge:** `attention` fill, `on-attention` 11/600 (white in light theme), pill, line-height 17, padding 0 × 6, accessible name "N unresolved".
- **Tag:** 11/500 `text-2`, 1 px `line-strong` border, radius 3, padding 1 × 6 ("AI summary", "Discarded"). **Coming-later tag:** 11 `text-3`, 1 px dashed border ("later in 0.3"), only on disabled items.
- **Avatar:** 28 px circle, initials 11/600 white. A stable hash of the author's email picks one of six colors: `#2457C5`, `#7A4FB0`, `#2F6B4F`, `#B4490B`, `#3D3F44`, `#0E6E7E`, each passing 4.5:1 with white initials (check the dark variants). **You** is always neutral grey `#3D3F44`. The same person gets the same color on every device. There is no agent marker (US00005).
- **Stage track:** see *Status window → Session row*.

## Status window (3a, 3b)

Native Windows title bar ("Dispatch Mesh", minimise, maximise, close = hide), then content of 440 × 640 by default, fitted to the work area on a shorter screen (minimum 380 × 360). Top to bottom:

1. **Top row**, 44 px, `surface`, borders `line-soft`: workspace name 14/600, aligned with the content edge. No app mark, since the native title bar already shows it. With several workspaces the name is a button with a chevron (the switcher) whose accessible name is "Team Wiki. Show all workspaces": the visible name first, then the action (WCAG 2.5.3 Label in Name; US00012 review F7). It replaces the old "‹ All workspaces" link. With one workspace it is plain text. No hide button: the native close hides the window.
2. **Path**, mono 12 `text-3`, one line, ellipsized; full path as a tooltip.
3. **State card**, radius 8, padding 16, gap 12:
   - Row 1: glyph 28 px, then label 17/600, reason 13 `text-2` under it, and the reassurance line 13/500 where the state has one.
   - Row 2 (actions): buttons on the left, "Last synced …" 12 `text-3` pushed right ("Not synced yet", or "This device only" without a remote).
4. **Section nav**: underlined tabs, **Activity** and **Conflicts** (current tab 13/600 with a 2 px `text` underline). Conflicts carries the count badge and the accessible name "Conflicts, N unresolved". Until US00006 the Conflicts tab shows a dashed placeholder: "N unresolved conflicts. Review them with the command line until conflict review arrives." (with none: "No unresolved conflicts."). **Open conflicts** switches to this tab.
5. **Activity list**: session rows separated by `line-soft` 1 px dividers. No cards. At 440 × 640 at least three rows are visible.

### State card per summary state

| Engine value | Label | Reason line | Reassurance | Actions | Card ground / border | Label color |
|---|---|---|---|---|---|---|
| `needs_attention` | Needs attention | the engine's `reason`, e.g. "2 unresolved conflicts" | per reason, below | primary per reason (**Open conflicts**, **Finish session**, …) | `attention-card` | `attention-text` |
| `offline` | Offline — changes saved on this device | the engine's `reason`, e.g. "Could not reach the remote (connection timed out)." | "Keep working — your changes upload when the connection returns." | none | `offline-card` | `text` |
| `saved_locally` | Saved on this device | "Your latest edits are captured. Sharing them as a draft in a moment." Without a remote: "Your latest edits are captured. They join history after 5 minutes without edits, or when you finish the session." | — | secondary **Finish session** | `surface` / `line` | `text` |
| `draft_uploaded` | Shared as draft | "Collaborators can see your session. It joins shared history after 5 minutes without edits, or when you finish it." | — | secondary **Finish session** | `surface` / `line` | `text` |
| `waiting_for_finalization` | Finishing up | "Adding your finished session to shared history." / "Adding your N finished sessions to shared history." | — | none | `surface` / `line` | `text` |
| `synced` | Up to date | "Everything you saved is in shared history." | — | none | `surface` / `line` | `text` |

Reassurance line per attention reason (US00012 Q8; all wording lives in `tray/src/model.rs`):

| Reason | Reassurance |
|---|---|
| Conflicts | "Both versions of each file are kept. Other files keep syncing." |
| Sync failure | "Your edits are saved on this device. Mesh retries automatically." (replaces the separate retry note) |
| Capture error | "Saved files stay as they are. Mesh tries again on the next save." |
| Edits depend on a received draft, or would create a circular draft dependency | "Your edits are saved on this device. Finish the session to share them." |
| AI summary failure | "Your session is added to shared history with a local summary." |

Any other reason (for example "synchronization paused" after history changed outside Mesh) gets no reassurance line.

**Finish session** (US00012, 2026-11-07): the Actions column lists what a state adds. As in US00005, a secondary **Finish session** is offered in every state while a session is open, unless it is already the primary action.

Only Needs attention gets color and a primary button. Offline is grey, never red. Working states stay white.

### Session row

```
[avatar 28]  Author 13/600 ("You" for this device)   meta 12 text-3 ("2 files · started 11:27 · last saved 2 min ago", never a device)
             Summary title once published (wraps, never cut off), else files (one line, ellipsized): 13/400
             [▬▬ ▬▬ ▭▭ ▭▭]  Stage label 12 text-2
```

- Gap 12 between avatar and text; 6 between the three lines.
- **Meta** always carries the file count, the session's start time and its last save. The start time is a clock time ("started 11:27"; a date once it isn't today), the last save is relative.
- **Every published session has a summary title**, whoever authored it: the AI-written title when AI summaries are on, otherwise the factual local summary. It replaces the file list on line 2. Unpublished sessions show their files. Agent sessions aren't special.
- **Stage track:** 4 segments with a remote, 3 without; each 4 px high, radius 2, gap 3, total width 28 px × segment count (112 / 84). Segments up to the current stage are filled with `working`; a published session fills the rest with `synced`; later segments are `line`. For a session from another device (its `device` differs from `Status.device`, US00012 Q5) the first segment is `received`, because this device never had it saved locally. It stays `received` after publication, as frames 3a and 3b draw it (US00012 review F3).
- **Author** 13/600 shrinks and ellipsizes for long names, but leaves room for the meta (US00012 review F1).
- **Tooltip** (Kai, US00012, 2026-11-07): hovering a row shows the author's full name, then "Changed files:" and the files one per line (at most 20, then "… and N more"). For a published row, whose line 2 is the title, screen readers get "Changed files: …" as text.
- The stage is named **once**, as text right of the track. The track itself is `aria-hidden`; the row carries "Stage: Shared as draft, 2 of 4" for screen readers.
- **Requested purpose** (US00012 planning decision, Kai, 2026-11-07; keeps US00005 AC 3): a session with a recorded purpose gets one more line under line 2, "Requested purpose: …" (12, `text-3`, ellipsized; several purposes read "… +N more"). Rows without a purpose keep three lines.
- Discarded sessions show a "Discarded" tag instead of the track.
- Incoming sessions show the other author, never "You".

## Session detail (3c)

Back link "‹ Activity / Session" in the top row. Then: status tag (`synced` on `synced-tint`, "Added to shared history") plus "AI summary" tag when the title came from AI; title 18/600; author line 12. A **Requested purpose** panel (`surface`, radius 8): caps label "Requested purpose · by <author>", the purpose in quotes 14/400, and "Applies to N of M files. This is what was asked for, not a check of what changed." Then the file list (mono 12, per-file "Requested purpose" tag), then a vertical stage timeline: 10 px dots, 2 px connector, label 13 left, time 12 right.

## Presence (3c, right half)

Recent-save presence only: dashed `presence` ring with a filled centre dot, "Alice recently changed this file · 12 s ago", with a small bar showing the time left of the 5 minutes. As a notice on a file: a dismissible `surface` card with the same glyph. The reserved "is editing now" signal (future editor integrations) is a solid filled `presence` pill. It must never look like the recent-save ring.

## Conflicts (3d, 3e, 3f)

Planning decisions of 2026-11-07 are in [[Story - US00006 - Conflict list and resolution in the status window|US00006]]; the frames follow them.

- **List:** heading "N unresolved conflicts", explanation 13 `text-2` ("Nothing was overwritten: both versions are kept …"). One card per conflict: file icon, path mono 13/500, "Text · recorded today 11:14". Two side boxes: **Current version · shown** (1 px solid `text` border) and **Other version · kept** (1 px dashed `#8A8C91`), each with the author ("You" for this device, "Shared history" when no session contributed) and time, never a device. **Review and choose** is a **secondary** button on every card.
- **Text detail** (window widened to about 1000 × 760): breadcrumb "‹ Conflicts / 1 of 2", path heading mono 18, a toggle **Show common base**, a base strip (`#EFEEEA`, caps label "Common base"), then two columns. Current version has a 1.5 px solid border and "In the folder now"; other version is dashed with "Kept, not shown". Line numbers in `text-3`; differing lines tinted `attention-card` with a "≠" gutter mark (named for screen readers). A side that is a deletion shows "Deleted by Bob · 10:50" instead of content, with no diff.
- **Buttons** name the side's author: secondary **Use other version (Bob)** and primary **Keep current version (Alice)**. Several contributors read "(Alice +1)", no contributing session reads "(shared history)", and a deletion reads "(deleted by Bob)". Footer: "Both versions stay in history whichever you choose."
- **Edited again after the conflict** (until [[Story - US00010 - Resolve a conflict after the file was edited again|US00010]]): a `working-tint` note "You saved this file again at 11:20, after the conflict was recorded. Your edit is safe. Choosing a version is possible once the file matches the current version again." Both buttons are disabled. US00010 enables them and switches to "That edit is protected and reconciled separately once you choose."
- **Binary detail:** same frame, checkerboard preview boxes, a metadata list (saved by, saved, size, purpose), whole-file choice only. Sentence: "Images can't be combined. Choose which whole file stays in the folder; the other version stays kept." Never "saved copy", which means a recovery file since US00005. Footer: "Whole-file choice only. Both versions stay in history."

## Other surfaces

- **Tray menu (2):** header with workspace and state, **Open status window** (default), **Finish session** with a one-line hint, a disabled **Pause sync** with the coming-later tag, separator, **Quit Dispatch Mesh** with the hint "Your edits stay saved. The session continues next start." Variant A (no Quit prompt) is the one built.
- **Already running / first run (3g):** centred message "Mesh is already running for this workspace" with **Retry**. First run: Up to date card, dashed empty state "No changes yet" with **Open folder**, and a one-time dark popover under the native close button: "Closing this window (×) doesn't stop Mesh. It keeps syncing from the tray." · **Got it**.
- **Several workspaces (3h):** list of workspace cards sorted by precedence (Needs attention › Offline › Working › Up to date). The tray shows the highest state, and its tooltip names that workspace.
- **Notifications (4):** toast on a new conflict (batched within 60 s), on entering or changing a Needs attention reason, and on leaving it (only if the start was toasted). Never for offline/online, working stages or publishing, never the same workspace + reason twice within 30 min, quiet under Do not disturb.
- **Guided setup (5):** flow-level sketch for 0.5; not a build spec.

## How to check a build against this page

1. Run the window with sample data covering every summary state, with and without a remote.
2. Take a screenshot at the default size and put it next to the matching PNG.
3. Go through *Status window* above line by line: structure, the stage named once, track segment count, tokens, at least three rows visible.
4. Record the comparison in the ticket's test plan.
