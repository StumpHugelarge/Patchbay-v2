# Patch Sheet — V2

> Rename `patchbay-v2-index.html` to `index.html` once it's in your repo
> (or its own `v2/` folder) — GitHub Pages and most static hosts expect
> that exact filename.

A single-page, no-build web app for labeling and documenting a recording
studio's patchbays — one section for a 16-point XLR passthrough bay, one
for TRS bays, which now come in **48-point (24×2) or 64-point (32×2)**,
chosen per bay. Labeling, color-coded device grouping, image export, and
JSON/CSV import/export.

Everything lives in the one HTML file — no build step, no dependencies to
install, no server required. Open it in a browser and it works.

## Features

- Two sections (XLR / TRS), each supporting any number of bays — add,
  rename, or delete bays freely. Adding a bay under TRS prompts for
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
- **Compact / Full view** toggle in the toolbar: Compact shows one bay at
  a time (today's default); Full stacks every bay in the current section
  on one scrollable page, so you can see them all at once. Bay tabs still
  work in Full view — clicking one scrolls you to that bay's panel.
- **Export**: PNG or JPG image of the current bay (styled like a rack
  panel, with a legend of the groups used), a full project JSON file, or
  a CSV of just the labels. The image matches the on-screen layout,
  including the 64-point split rows and routing indicators. There's also
  a **Master Sheet** image export — one tall PNG/JPG stacking every bay
  in the whole session (XLR first, then TRS), independent of whatever
  view mode or section you're currently looking at.
- **Import**: a JSON file replaces the whole project; a CSV updates
  labels/notes/groups/routing by matching `port_id`, so you can bulk-edit
  in a spreadsheet and bring it back in.
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
    }
  ],
  "groups": [
    { "id": "grp-...", "name": "SSL Bus Outs", "color": "#D64550" }
  ]
}
```

**Labels CSV** — one row per port, for bulk editing in a spreadsheet:

```
bay_id,bay_name,bay_type,row,index,port_id,label,notes,group,routing
trs-...,TRS 64pt Bay 1,trs,top,1,trs-...-top-1,SSL Out 1,,SSL Bus Outs,half-normal
```

On import, rows are matched back to ports by `port_id`; a `group` name
that doesn't already exist is created automatically. `routing` is only
applied to TRS bays and only when it's one of `normal` / `half-normal` /
`thru` — anything else is left as-is.

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

## Roadmap

Natural next steps from here: print-accurate label-strip export sized to
real hardware, and multi-room bay organization.
