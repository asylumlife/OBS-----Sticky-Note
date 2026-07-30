# OBS Sticky Note Spec

A sticky note for OBS showing a checklist of stream goals. Viewers see it as
an overlay; the streamer checks items off from a clickable panel inside OBS.

## Decisions made

| Question | Decision |
|---|---|
| Check-off method | OBS Custom Browser Dock (control) + Browser Source (display), auto-synced |
| Visibility | Visible on stream as an overlay |
| Style | Sticky-note look with selectable themes; classic yellow is the default |
| Checked items | Strikethrough and dim, staying in place so progress accumulates |

## Architecture

**One self-contained HTML file (`sticky-note.html`): no server, no build, no
dependencies.** The same file loads in two places, with a URL parameter
choosing the mode:

- `sticky-note.html?view=dock` is added in OBS under *View → Docks → Custom
  Browser Docks*. Full editing UI: add, edit, delete, and reorder tasks,
  check items off, reset.
- `sticky-note.html` (default, overlay mode) is added as a Browser Source in
  the scene. Read-only, transparent page background; only the note itself
  renders.

### Sync between dock and overlay

Both contexts run in OBS's embedded browser (CEF) and share `localStorage`
when they load from the same origin. State lives under one localStorage key
(JSON: list title plus an array of `{id, text, done}`), and the overlay
updates on the `storage` event.

The Browser Source must be configured with *Local file unchecked* and the
same `file:///` URL the dock uses. OBS's "Local file" mode serves the page
from a different internal origin, which would split the localStorage. The
README spells this out.

As a fallback, the overlay also polls localStorage once per second. If events
fail to propagate across CEF contexts, sync still works with at most 1 s of
latency.

localStorage also gives us persistence across OBS restarts for free.

## Features

### Dock (control panel)
- Editable note title (default: "Today's Goals")
- Theme picker: a row of swatches; clicking one restyles both the dock
  preview and the live overlay (theme is part of the synced state)
- Custom colors: Paper and Ink pickers. Changing either switches to a
  "custom" theme derived from the two colors, with the gradient, corner curl,
  and dimmed ink computed from them. Clicking a swatch returns to a preset.
- Size slider (50–200%) and Tilt slider (±15°), both synced to the overlay
- Text input + Enter to add a task
- Click a checkbox to toggle done
- Edit task text inline; delete a task; reorder via up/down buttons
- "Uncheck all" (reuse the list next stream) and "Clear list" (with confirm)
- Shows the same sticky-note rendering as the overlay, so what you see is
  what viewers see

### Overlay (browser source)
- Renders the note only, on a transparent page background
- Checking an item: the checkmark draws in like a pen stroke, then an
  animated strikethrough crosses the text; the item dims to about 50% and
  stays in place
- Unchecking reverses the state without replaying animations
- Small progress line at the bottom of the note: "2 / 5 done"
- When the last item is checked, the note wiggles and an "All done!" stamp
  appears. No sound.

## Visual design and themes

Shared across themes:

- Sticky-note silhouette: soft drop shadow, a slight tilt, and a darker
  fold/curl at one corner
- Handwritten font (Caveat), embedded as base64 woff2 so the note works
  offline with no CDN dependency
- Hand-drawn-looking checkboxes; pen-stroke check and strikethrough
  animations
- Base size about 380 px wide at 1080p; height grows with the list. Size and
  tilt are adjustable with dock sliders, and a `?scale=` URL parameter adds
  an extra multiplier.

Themes bundle paper, ink, and accent colors. They're picked in the dock and
stored in synced state, so the overlay follows.

| Theme | Look |
|---|---|
| **Classic Yellow** (default) | #feff9c-ish paper, dark ink |
| **Pink** | Pastel pink paper, dark ink |
| **Blue** | Pastel blue paper, dark ink |
| **Green** | Pastel green paper, dark ink |
| **Lined Notepad** | White paper with ruled lines and a red margin line |
| **Kraft** | Brown paper texture, marker-style dark ink |
| **Midnight** | Dark charcoal note with chalk-white ink, for dark overlay setups |
| **Custom** | Any paper and ink color via the pickers; shading derived from the two |

Each theme is a CSS custom-property set, so adding a new one later takes a
few lines of CSS.

## Setup (ships as README)

1. Save `sticky-note.html` anywhere on disk.
2. OBS → View → Docks → Custom Browser Docks → add
   `file:///path/to/sticky-note.html?view=dock`. Dock it wherever you like.
3. Scene → Add → Browser Source → uncheck Local file → URL
   `file:///path/to/sticky-note.html` → size ~470×700.
4. Type tasks in the dock; the overlay follows.

## Out of scope (possible later)

- Multiple simultaneous notes / per-scene lists
- OBS hotkey integration ("check next item" without touching the dock)
- Remote control from a phone
- Sound effects
- Saved list templates (e.g. a recurring pre-stream checklist)

## Acceptance checklist

- [ ] Adding a task in the dock appears on the overlay within 1 s
- [ ] Checking an item plays the checkmark and strikethrough animation on the
      overlay
- [ ] The item stays visible, crossed out and dimmed
- [ ] All state survives a full OBS restart
- [ ] The overlay background is transparent over any scene
- [ ] Picking a theme in the dock restyles the overlay within 1 s, and the
      choice survives an OBS restart
- [ ] Works with networking disabled
- [ ] The note stays legible with 10+ tasks and long task text (wraps, no
      overflow)
