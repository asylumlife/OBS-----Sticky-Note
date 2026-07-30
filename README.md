# OBS Sticky Note

A sticky-note checklist for your stream. Viewers see the note as an overlay and
watch items get crossed off; you check them off from a clickable panel docked
inside OBS. One HTML file, no server, works fully offline, survives OBS
restarts.

## Files

- `sticky-note.html` — the whole app (font embedded; keep it wherever you
  like, but don't move it after setup or the OBS URLs will break)
- `SPEC.md` — design spec

## Setup (once, ~2 minutes)

You'll load the same file twice: once as a **dock** (your control panel) and
once as a **browser source** (what viewers see). Both must use a `file:///` URL
— and it must be the **same** URL in both places, or they won't sync.

Your file URL looks like this (spaces in the path must be written as `%20`):

```
file:///Users/you/path%20to/sticky-note.html
```

**Don't want to type that by hand?** Just double-click `sticky-note.html` to
open it in your normal browser — it shows both ready-made URLs with Copy
buttons. The dock shows the same URLs at the bottom of its panel.

### 1. Add the control dock

1. In OBS: **View → Docks → Custom Browser Docks…**
2. Dock Name: `Sticky Note`
3. URL: your file URL **plus** `?view=dock`, e.g.
   `file:///Users/you/path%20to/sticky-note.html?view=dock`
4. Click **Apply**. Drag the dock wherever you like in the OBS window.

### 2. Add the on-stream overlay

1. In your scene: **Sources → + → Browser**
2. **Uncheck "Local file"** — this matters; local-file mode loads the page
   from a different internal origin and the dock and overlay won't sync.
3. URL: your file URL with no parameters, e.g.
   `file:///Users/you/path%20to/sticky-note.html`
4. Width `470`, Height `700` (or whatever fits your list). Leave "Shutdown
   source when not visible" unchecked so state stays live.
5. Position and scale the note in your scene like any other source.

That's it. Type tasks in the dock; the overlay follows within a second.

## Using it

All editing happens in the dock:

- **Add a task** — type in the "Add a task" line, press Enter
- **Check off** — click the checkbox (viewers see the pen-stroke check and
  strikethrough; the item dims but stays visible)
- **Edit** — click any task's text or the title and type; Enter or clicking
  away saves, Esc cancels
- **Reorder / delete** — hover a task for the ▲ ▼ ✕ buttons
- **Theme** — click a color swatch in the toolbar (Classic Yellow, Pink, Blue,
  Green, Lined Notepad, Kraft, Midnight)
- **Custom colors** — use the Paper and Ink pickers in the toolbar; touching
  either one switches to your custom colors (click any swatch to go back to a
  preset)
- **Size and Tilt** — drag the sliders (50–200% and ±15°); the on-stream note
  follows live
- **Uncheck all** — resets the checkmarks so you can reuse the list next stream
- **Clear list** — deletes all tasks (click twice to confirm)

When everything is checked, the note wiggles and gets an "All done!" stamp.

## URL parameters

| Parameter | Effect |
|---|---|
| `?view=dock` | Control-panel mode (editing UI). Without it: overlay mode. |
| `?scale=1.5` | Extra scale multiplier (0.2–5) on top of the Size slider. Normally just use the slider. |
| `?demo=1` | Sample tasks, nothing saved — for previewing in a normal browser. |
| `?demo=all` | Sample tasks all checked (previews the All-done stamp). |
| `?demo=1&theme=kraft` | Preview a specific theme. |

Parameters combine with `&`, e.g. `?view=dock&scale=1.2`.

## Troubleshooting

**Dock and overlay don't sync**
- The browser source must have **Local file unchecked** and use the exact same
  `file:///` URL as the dock (the `?view=dock` suffix being the only
  difference).
- Right-click the browser source → **Refresh cache of current page**, and
  re-open the dock.
- Worst case, you can skip the dock entirely: right-click the browser source →
  **Interact**, and check items off in that window (add `?view=dock` to the
  source URL temporarily to edit, or keep a second interact-able source).

**Note doesn't appear on stream**
- Check the source URL starts with `file:///` (three slashes) and that spaces
  in the path are `%20`.

**I moved or renamed the file and OBS shows a blank dock/source**
- The OBS URLs still point at the old location. Double-click the file in its
  new location — it opens in your browser and shows the new Dock and Overlay
  URLs with Copy buttons. Paste them into the dock settings and the browser
  source. (Your tasks are safe; state isn't stored in the file.)

**My tasks disappeared after moving the file**
- State is stored in OBS's browser storage, keyed to the page origin — moving
  or renaming the file is fine (state is shared across all `file://` pages),
  but wiping OBS's browser cache (`plugin_config/obs-browser`) clears it.

**Text looks like a generic font**
- The handwriting font is embedded in the HTML; if you see a plain font, the
  file was probably re-saved by an editor that stripped it. Re-download
  `sticky-note.html`.
