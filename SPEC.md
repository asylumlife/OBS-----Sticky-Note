# OBS Sticky Note — Spec

A sticky note for OBS showing a checklist of stream goals. Viewers see it as an
overlay; the streamer checks items off from a clickable panel inside OBS.

## Decisions made

| Question | Decision |
|---|---|
| Check-off method | OBS Custom Browser Dock (control) + Browser Source (display), auto-synced |
| Visibility | Visible on stream as an overlay |
| Style | Sticky-note aesthetic with multiple selectable themes; classic yellow is the default |
| Checked items | Strikethrough + dim, stay in place so progress accumulates |

## Architecture

**One self-contained HTML file (`sticky-note.html`), no server, no build, no
dependencies.** The same file is loaded in two places with a URL parameter
choosing the mode:

- `sticky-note.html?view=dock` — added in OBS under *View → Docks → Custom
  Browser Docks*. Full editing UI: add / edit / delete / reorder tasks, check
  items off, reset.
- `sticky-note.html` (default, overlay mode) — added as a Browser Source in the
  scene. Read-only, transparent page background, only the note itself renders.

### Sync between dock and overlay

Both contexts run in OBS's embedded browser (CEF) and share `localStorage` when
they load from the **same origin**. State is written to one localStorage key
(JSON: list title + array of `{id, text, done}`), and the overlay updates via
the `storage` event.

**Key implementation detail:** the Browser Source must be configured with
*Local file unchecked* and the same `file:///` URL the dock uses. (OBS's "Local
file" mode serves the page from a different internal origin, which would split
the localStorage.) Setup instructions will spell this out.

**Fallback:** in addition to the `storage` event, the overlay polls localStorage
once per second. If events ever fail to propagate across CEF contexts, sync
still works with ≤1 s latency.

Persistence across OBS restarts comes free with localStorage.

## Features

### Dock (control panel)
- Editable note title (default: "Today's Goals")
- Theme picker: a row of swatches; clicking one restyles both the dock preview
  and the live overlay (theme is part of the synced state)
- Custom colors: Paper and Ink color pickers; changing either switches to a
  "custom" theme derived from the two colors (gradient, corner curl, and dimmed
  ink computed automatically); clicking a swatch returns to a preset
- Size slider (50–200%) and Tilt slider (±15°), both synced to the overlay
- Text input + Enter to add a task
- Click checkbox to toggle done
- Edit task text inline; delete a task; reorder via up/down buttons
- "Uncheck all" (reuse the list next stream) and "Clear list" (with confirm)
- Shows the same sticky-note rendering as the overlay, so what you see is what
  viewers see

### Overlay (browser source)
- Renders the note only — page background fully transparent
- Checking an item: checkmark draws in like a pen stroke, then an animated
  strikethrough crosses the text; item dims to ~50% and stays in place
- Unchecking cleanly reverses the state (no animation replay spam)
- Small progress line at the bottom of the note: "2 / 5 done"
- When everything is done: brief, tasteful celebration (note does a little
  wiggle + "All done!" stamp). No sound.

## Visual design & themes

Shared across all themes:

- Sticky-note silhouette: soft drop shadow, rotated ~-2°, slightly darker
  fold/curl at one corner
- Handwritten font (e.g. Caveat), **embedded as base64 woff2** in the HTML so
  the note works offline and never depends on a CDN
- Hand-drawn-looking checkboxes; pen-stroke check and strikethrough animations
- Base size ~380 px wide at 1080p, height grows with the list; a `?scale=1.5`
  URL parameter scales the whole note for other canvas sizes

Themes (paper + ink + accent bundled; picked in the dock, stored in synced
state so the overlay follows instantly):

| Theme | Look |
|---|---|
| **Classic Yellow** (default) | #feff9c-ish paper, dark ink |
| **Pink** | Pastel pink paper, dark ink |
| **Blue** | Pastel blue paper, dark ink |
| **Green** | Pastel green paper, dark ink |
| **Lined Notepad** | White paper with ruled lines and a red margin line |
| **Kraft** | Brown paper texture, marker-style dark ink |
| **Midnight** | Dark charcoal note, chalk-white ink — for dark overlay setups |
| **Custom** | Any paper + ink color via pickers in the dock; shading derived automatically |

Themes are defined as CSS custom-property sets, so adding a new one later is a
few lines of CSS, not a redesign.

## Setup (will ship as README)

1. Save `sticky-note.html` anywhere on disk.
2. OBS → View → Docks → Custom Browser Docks → add
   `file:///path/to/sticky-note.html?view=dock`. Dock it wherever you like.
3. Scene → Add → Browser Source → **uncheck Local file** → URL
   `file:///path/to/sticky-note.html` → size ~420×600.
4. Type tasks in the dock; the overlay follows automatically.

## Out of scope (possible later)

- Multiple simultaneous notes / per-scene lists
- OBS hotkey integration ("check next item" without touching the dock)
- Remote control from a phone
- Sound effects
- Saved list templates (e.g. a recurring pre-stream checklist)

## Acceptance checklist

- [ ] Adding a task in the dock appears on the overlay within 1 s
- [ ] Checking an item plays checkmark + strikethrough animation on the overlay
- [ ] Item stays visible, crossed out and dimmed
- [ ] All state survives a full OBS restart
- [ ] Overlay background is transparent over any scene
- [ ] Picking a theme in the dock restyles the overlay within 1 s, and the
      choice survives an OBS restart
- [ ] Whole thing works with networking disabled (fully offline)
- [ ] Note stays legible with 10+ tasks and with long task text (wraps, no
      overflow)
