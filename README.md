# OBS Sticky Note

A sticky-note checklist for your stream. Viewers watch items get crossed off
on an overlay while you check them off from a panel docked inside OBS. One
HTML file, no server to run. Works offline, and your list survives OBS
restarts.

## Files

- `sticky-note.html`: the whole app, handwriting font included. Keep it
  wherever you like, but if you move it after setup you'll need to update the
  URLs in OBS.
- `SPEC.md`: design spec

## Setup (once, about 2 minutes)

You load the same file twice: once as a **dock** (your control panel) and once
as a **browser source** (what viewers see). Both need a `file:///` URL, and it
must be the same URL in both places or they won't sync.

The URL looks like this (write spaces in the path as `%20`):

```
file:///Users/you/path%20to/sticky-note.html
```

To skip typing it by hand, double-click `sticky-note.html`. It opens in your
browser and shows both ready-made URLs with Copy buttons. The dock shows the
same URLs at the bottom of its panel.

### 1. Add the control dock

1. In OBS: **View → Docks → Custom Browser Docks…**
2. Dock Name: `Sticky Note`
3. URL: your file URL plus `?view=dock`, e.g.
   `file:///Users/you/path%20to/sticky-note.html?view=dock`
4. Click **Apply**, then drag the dock wherever you like in the OBS window.

### 2. Add the on-stream overlay

1. In your scene: **Sources → + → Browser**
2. Uncheck **Local file**. OBS serves local files from a different internal
   origin, and the dock and overlay won't sync if you leave it checked.
3. URL: your file URL with no parameters, e.g.
   `file:///Users/you/path%20to/sticky-note.html`
4. Width `470`, Height `700`, or whatever fits your list. Leave "Shutdown
   source when not visible" unchecked so the note stays live.
5. Position and scale it in your scene like any other source.

Type a task in the dock and the overlay picks it up within a second.

## Using it

All editing happens in the dock:

- **Add a task**: type in the "Add a task" line, press Enter
- **Check off**: click the checkbox. Viewers see the pen-stroke check and
  strikethrough; the item dims but stays visible.
- **Edit**: click any task's text or the title and type. Enter or clicking
  away saves, Esc cancels.
- **Reorder / delete**: hover a task for the ▲ ▼ ✕ buttons
- **Theme**: click a color swatch in the toolbar (Classic Yellow, Pink, Blue,
  Green, Lined Notepad, Kraft, Midnight)
- **Custom colors**: pick Paper and Ink colors in the toolbar. Touching either
  one switches to your custom scheme; click any swatch to go back to a preset.
- **Size and Tilt**: drag the sliders (50–200% and ±15°) and the on-stream
  note follows
- **Uncheck all**: resets the checkmarks so you can reuse the list next stream
- **Clear list**: deletes all tasks (click twice to confirm)

Check off the last item and the note wiggles, then an "All done!" stamp
appears.

## URL parameters

| Parameter | Effect |
|---|---|
| `?view=dock` | Control-panel mode (editing UI). Without it: overlay mode. |
| `?scale=1.5` | Extra scale multiplier (0.2–5) on top of the Size slider. |
| `?demo=1` | Sample tasks, nothing saved. Good for previewing in a browser. |
| `?demo=all` | Sample tasks all checked, to preview the All-done stamp. |
| `?demo=1&theme=kraft` | Preview a specific theme. |

Combine parameters with `&`, e.g. `?view=dock&scale=1.2`.

## Troubleshooting

**Dock and overlay don't sync**
- The browser source needs **Local file unchecked** and the exact same
  `file:///` URL as the dock. The `?view=dock` suffix should be the only
  difference.
- Right-click the browser source, pick **Refresh cache of current page**, and
  re-open the dock.
- Worst case, skip the dock: right-click the browser source, pick
  **Interact**, and check items off in that window. Add `?view=dock` to the
  source URL while you edit, or keep a second source for interacting.

**Note doesn't appear on stream**
- Check that the source URL starts with `file:///` (three slashes) and that
  spaces in the path are written as `%20`.

**I moved or renamed the file and OBS shows a blank dock/source**
- The OBS URLs still point at the old location. Double-click the file in its
  new spot. It opens in your browser and shows the new Dock and Overlay URLs
  with Copy buttons; paste those into the dock settings and the browser
  source. Your tasks are safe. The list lives in OBS's browser storage, not
  in the file.

**My tasks disappeared**
- OBS stores the list in its browser storage, keyed to the page origin.
  Moving or renaming the file keeps it intact, since all `file://` pages
  share storage, but wiping OBS's browser cache (`plugin_config/obs-browser`)
  clears it.

**Text looks like a generic font**
- The handwriting font is embedded in the HTML. If you see a plain font, an
  editor may have re-saved the file and stripped it. Re-download
  `sticky-note.html`.
