# Patch Sheet — V2

> Rename `patchbay-v2-index.html` to `index.html` once it's in your repo
> (or its own `v2/` folder) — GitHub Pages and most static hosts expect
> that exact filename.

A single-page, no-build web app for labeling and documenting a recording
studio's patchbays and equipment racks — one section for a 16-point XLR
passthrough bay, one for TRS bays (which come in **48-point (24×2) or
64-point (32×2)**, chosen per bay), and one for rack layouts (1U–24U
frames you fill with named gear). Labeling, color-coded device grouping,
image export, and JSON/CSV import/export.

Everything lives in the one HTML file — no build step, no dependencies to
install, no server required. Open it in a browser and it works.

## Features

- Three sections (XLR / TRS / Rack), each supporting any number of bays —
  add, rename, or delete freely. Adding a bay under TRS prompts for
  48-point or 64-point; each bay's tab shows a small size badge so mixed
  sizes stay easy to tell apart.
- Click a port to label it; press **Enter** to jump straight to the next
  one for fast bulk entry. Shift-click for a range, ⌘/Ctrl-click to build
  a scattered selection — handy for coloring every input on one device at
  once.
- **64-point TRS layout**: built from two ganged 16-wide strips (1–16,
  17–32), each with its own top/bottom pair, so cells stay full-size
  instead of shrinking to fit 32 in one line — and every TRS/XLR row
  groups ports in blocks of 8 with a slightly wider gap, for faster
  scanning.
- **Routing state (TRS only)**: each column can be marked Normal,
  Half-Normal, or Thru, shown as a small bridge indicator between its top
  and bottom jack (solid = Normal, broken = Half-Normal, × = Thru). Click
  the indicator — or any jack — to select the column, then set its state
  from the Inspector; select several jacks first to set a whole run at
  once.
- **Device groups**: name a group (e.g. "A/D Converter 1"), pick a color,
  and assign it to any ports. Assigned ports get that color as a border,
  so you can see at a glance which jacks belong to the same piece of gear.
- **Rack layouts**: build a rack elevation from 1U to 24U tall, then add
  gear pieces 1U–4U tall, full-width or half-width. Half-width pieces
  automatically pair side-by-side into one row when they're added
  back-to-back with matching heights; otherwise they leave an open half
  slot that a later matching piece can fill. Each piece gets a name and a
  notes field, editable from the Inspector, which also has move up/down
  and delete controls. The toolbar shows how many U are still free, and
  adding a piece that won't fit is rejected with a toast instead of
  overflowing the frame.
- **Compact / Full view** toggle in the toolbar: Compact shows one bay at
  a time (today's default); Full stacks every bay in the current section
  on one scrollable page, so you can see them all at once. Bay tabs still
  work in Full view — clicking one scrolls you to that bay's panel.
- **Export**: the Export menu is split into two clearly labeled groups —
  **This bay** (PNG/JPG of just the bay or rack you're viewing, styled
  like a rack panel with a legend of the groups used) and **All bays
  (master sheet)** (one tall PNG/JPG stacking every bay and rack in the
  whole session — XLR, then TRS, then racks — independent of whatever
  view mode or section you're currently looking at) — plus a full project
  JSON file or a CSV of just the labels. Both image exports match the
  on-screen layout, including the 64-point split rows, routing
  indicators, and rack gear.
- **Import**: a JSON file replaces the whole project; a CSV updates
  labels/notes/groups/routing/gear by matching `port_id` (or a gear
  piece's id in that same column), so you can bulk-edit in a spreadsheet
  and bring it back in.
- **Light/dark toggle** in the top bar — follows your system setting by
  default, but you can override it; your choice is remembered on this
  device.
- Work-in-progress is mirrored to `localStorage` as a safety net, with a
  restore prompt if you reopen the page before exporting.

## Running it

Just open the HTML file in a browser — double-click it, or:

```bash
open index.html          # macOS
xdg-open index.html      # Linux
```

For the most reliable experience (some browsers restrict local storage
and file APIs for `file://` pages), serve it over HTTP instead:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

### Hosting on GitHub Pages

Push this repo, then in **Settings → Pages** set the source to the
branch/folder containing `index.html`. No build step is needed — GitHub
Pages will serve the file as-is.

## Data & file formats

All data lives in the browser (`localStorage`) until you explicitly
export it — there's no backend and nothing is sent anywhere. The two
export formats:

**Project JSON** — the full round-trip format; re-importing this
recreates every bay, port, label, note, group, and routing state exactly.
TRS bays carry a `size` field (`48` or `64`) and a `columns` array (one
entry per column, `state` is `"normal"`, `"half-normal"`, or `"thru"`); a
project file from an earlier version (missing `size` and/or `columns`)
still imports fine — `size` defaults to 48 and every column defaults to
`"normal"`.

```json
{
  "version": 1,
  "bays": [
    { "id": "trs-...", "type": "trs", "name": "TRS 64pt Bay 1", "size": 64,
      "ports": [
        { "id": "trs-...-top-1", "row": "top", "index": 1, "label": "SSL Out 1", "notes": "", "groupId": "grp-..." }
      ],
      "columns": [
        { "index": 1, "state": "half-normal" }
      ]
    },
    { "id": "rack-...", "type": "rack", "name": "Rack 1", "heightU": 12,
      "gear": [
        { "id": "gear-...", "name": "SSL Bus Compressor", "notes": "Stereo", "heightU": 2, "width": "full" },
        { "id": "gear-...", "name": "DI Box A", "notes": "", "heightU": 1, "width": "half" }
      ]
    }
  ],
  "groups": [
    { "id": "grp-...", "name": "SSL Bus Outs", "color": "#D64550" }
  ]
}
```

A rack's `gear` array is stacking order, top to bottom — vertical position
and half-width pairing are computed from that order on load, not stored.
A project file from an earlier version (no rack bays at all) still imports
fine, and any bay missing recognized fields falls back to safe defaults.

**Labels CSV** — one row per port (or, for rack bays, one row per gear
piece), for bulk editing in a spreadsheet:

```
bay_id,bay_name,bay_type,row,index,port_id,label,notes,group,routing,gear_height_u,gear_width
trs-...,TRS 64pt Bay 1,trs,top,1,trs-...-top-1,SSL Out 1,,SSL Bus Outs,half-normal,,
rack-...,Rack 1,rack,,1,gear-...,SSL Bus Compressor,Stereo,,,2,full
```

On import, rows are matched back to ports (or gear pieces) by `port_id`;
a `group` name that doesn't already exist is created automatically.
`routing` is only applied to TRS bays and only when it's one of `normal`
/ `half-normal` / `thru`. `gear_height_u` (1–4) and `gear_width` (`full`
or `half`) are only applied to gear rows — anything unrecognized is left
as-is.

## Notes for this version

File saves use the browser's native download (a small `<a download>`/Blob
fallback) instead of Claude's platform API, so Export/Import work the same
way whether you run this from GitHub Pages, a local server, or the
original Artifact.

Fonts (Big Shoulders Display, IBM Plex Sans, IBM Plex Mono) load from
Google Fonts via the `<link>` in the `<head>`; if you need a fully
offline copy, download those font files and swap the `<link>` for local
`@font-face` rules.

## Changelog vs. V1

- **Added:** 64-point TRS bays — chosen per bay from the "+ Add bay" menu
  under the TRS section (no new top-level tab; point count is a property
  of the bay, so 48pt and 64pt bays live side by side). Each 64pt bay is
  built from two 16-wide strips (1–16, 17–32), each with its own
  top/bottom pair stacked directly together, matching how the hardware
  is actually wired.
- **Added:** 8-column visual chunking on every TRS/XLR row for faster
  scanning.
- **Added:** per-column routing state (Normal / Half-Normal / Thru) for
  TRS bays, shown as a bridge indicator between each column's jacks and
  set from the Inspector; carried through JSON, CSV, and image export.
- **Added:** a light/dark toggle in the top bar, remembered per device.
- **Added:** a Compact/Full view toggle (see above) and a Master Sheet
  image export covering every bay in the session.
- **Fixed:** the Export/Import dropdown menus re-attached their click
  handler on every re-render, so after enough interactions a single menu
  click could fire the underlying action more than once. Listener wiring
  now happens once at startup instead of on every render. (This fix also
  shipped as a V1 patch.)
- **Fixed:** the 8-column chunking gap used a CSS margin that shrank the
  8th/16th/24th port in each row instead of adding real space between
  them. Ports are now uniform width throughout.
- **Fixed:** port cells used to stretch to fill extra window width on
  XLR rows but stay pinned at their minimum on TRS rows (an accident of
  which rows sat in a flex container vs. a plain block one), so labels
  entered at a wide window looked a different width than the fixed-size
  export. Every row type now uses a fixed port width that matches the
  export exactly, regardless of window size.
- **Changed:** the Export menu now groups image exports under **This
  bay** and **All bays (master sheet)** headings instead of separate
  "Image" / "Master Sheet" button labels, so the single-bay vs.
  whole-session choice is explicit before you click.
- **Added:** a third **Rack** section for building rack elevations —
  1U–24U frames filled with named, note-able gear pieces (1U–4U tall,
  full- or half-width, with auto-pairing for matching half-width pairs).
  Racks get the same Compact/Full view, image export, and Master Sheet
  inclusion as XLR/TRS bays, plus JSON and CSV round-trip support.

## Roadmap

Natural next steps from here: print-accurate label-strip export sized to
real hardware, drag-to-reorder rack gear, and multi-room bay organization.
