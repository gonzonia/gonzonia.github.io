# ES Theme Studio v3.57

A single-file, browser-based visual editor for EmulationStation theme XML files. Designed to run locally alongside your theme files — no server required.

---

## Getting Started

1. Place `ES_Theme_Studio.html` in the **same folder** as your `theme.xml` and theme assets.
2. Open the HTML file in your browser (Chrome or Safari recommended).
3. Load your theme using one of the methods below.

> **Note:** The tool must be opened as a `file://` URL (directly from disk), not served from a web server, for relative asset paths to resolve automatically.

---

## Loading a Theme

### Drag and Drop
Drag your `theme.xml` file onto the drop zone in the sidebar.

### Browse
Click the drop zone to open a file picker and select your `theme.xml`.

### Paste Raw XML
Click **Show Raw Input**, paste your XML directly into the text area, and click **Parse Raw Text**.

When a theme loads successfully, the tool will:
- Parse all views and elements from the XML
- Attempt to auto-load images and fonts from their relative paths
- Display a **✓ Found** badge (blue) next to any asset resolved from a relative path
- Display a **✓ Loaded** badge (green) next to any asset manually uploaded

---

## Interface Overview

### Sidebar (left panel)
Contains all element cards for the current view. Each card shows the element's properties and controls.

The sidebar can be **collapsed and expanded** using the `◄` / `►` toggle button on its right edge. Collapsing the sidebar expands the monitor preview to full width.

### Monitor Preview (right panel)
A scaled 16:9 preview of the current view, showing elements positioned and styled as they would appear in EmulationStation.

### View Tabs
Switch between **system**, **basic**, **detailed**, **video**, and **grid** views using the tabs above the monitor.

---

## Element Cards

Each element in the current view has a card in the sidebar showing its properties. Cards are sorted by z-index, with visible elements first.

### Show / Hide
Click **Hide** to remove an element from the current view preview. Click **Add to View** to restore it. Hidden elements are shown at reduced opacity in the sidebar and are not rendered in the monitor.

### Delete
Removes a custom (`extra="true"`) element entirely from the current view.

### Asset Badges
- **✓ Found** (blue) — asset was auto-loaded from a relative path when the XML was parsed
- **✓ Loaded / ✓ Linked** (green) — asset was manually uploaded by the user

### Export Custom Text (md_lbl elements)
Label elements (`md_lbl_rating`, `md_lbl_releasedate`, etc.) have an **Export Custom Text** checkbox. By default the `<text>` property is not exported for these elements, since EmulationStation provides its own default labels. Check the box to export a custom label string.

---

## System Variable Preview

Paths in ES themes can contain system variables such as `${system.theme}`, `${system.name}`, and `${system.fullName}`. These cannot be resolved without knowing the current system context.

Click the **`${system.*}`** button in the tab bar to open the variable mapping dialog. Enter preview values for each variable (e.g. `system.theme = snes`) and click **Apply Preview**. The monitor will re-render with the substituted paths, loading real assets where files exist.

> These values are for **preview only** — they are not saved or exported with your theme.

Paths with unresolved variables display an amber placeholder in the monitor showing the variable name and file type (e.g. `[SVG] ${system.theme}`).

---

## Dragging Elements

Elements in the monitor preview can be repositioned by clicking and dragging.

- **Default (pixel snap):** Element position snaps to the nearest pixel equivalent (1/1920 × 1/1080), producing clean decimal values.
- **Grid snap:** Hold **Shift** while dragging to snap to 0.01 increments, useful for aligning elements.
- A **red outline** appears on the element while dragging.
- A **tooltip** near the cursor shows the live `pos` values and current snap mode (`[px]` or `[grid]`).
- The sidebar **pos X** and **pos Y** inputs update in real time as you drag.
- If an element has an `origin` set, the drag anchor respects it — e.g. an element with `origin: 0.5 0.5` is grabbed from its visual center.

Clicking an element without dragging selects it (highlighted in the sidebar, scrolled into view).

---

## Adding Custom Elements

Use the buttons at the bottom of the sidebar to add extra elements to the current view:

- **+ Extra Image** — adds a new `image` element with `extra="true"`
- **+ Extra Text** — adds a new `text` element with `extra="true"`

You will be prompted to enter a name. Custom element names should not clash with existing ES element names — consider a prefix like `e_` or `bg_`.

---

## System View

The system view renders a **carousel preview** showing logo slots. The center slot is displayed at the selected scale (`logoScale`), with flanking slots dimmed.

- The `logo` image is rendered inside carousel slots — it does not appear as a standalone element.
- `logoText` is used as a fallback when no logo image is available or fails to load.
- The `logo` and `logoText` sidebar cards show only their restricted properties (as defined by the ES spec for the system view).

---

## Exporting

Click **Compile & Export XML** to generate a theme XML file. The compiler:

1. **Groups shared properties within a view** — elements of the same type sharing identical non-positional properties are collapsed into multi-name blocks (e.g. `<text name="md_genre, md_lbl_genre, md_players">`)
2. **Groups shared properties across views** — elements with identical properties across multiple views are output as a single `<view name="detailed, video">` block
3. **Positional props follow individual blocks** — `pos`, `size`, `origin`, etc. are written per-element unless they are identical across views
4. **Off-screen elements** are compressed to `<pos>2 2</pos>`
5. **`<text>` is omitted** for metadata value elements whose text is the auto-generated placeholder
6. **`<visible>false</visible>`** is only written when explicitly set — ES defaults to visible

The output XML is displayed in a text area and can be copied or downloaded.

---

## Asset Management

### Images
Upload a local image file using the **Choose File** button on an element card. Uploaded images are used for monitor preview only and are not embedded in the exported XML — the original path string is preserved.

### Fonts
Upload a font file (`.ttf`, `.otf`) to preview text rendering with the correct typeface. The font path in the XML is preserved on export.

---

## Schema Coverage

The tool supports all standard EmulationStation element types and properties as documented in the [RetroPie THEMES.md](https://raw.githubusercontent.com/RetroPie/EmulationStation/refs/heads/master/THEMES.md):

| Element | Notes |
|---|---|
| `image` | Including `rotation`, `rotationOrigin`, `default`, `visible` |
| `video` | Including `showSnapshotDelay`, `default` |
| `text` | Including `rotation`, `backgroundColor` |
| `textlist` | Including `selectorImagePath`, `selectorHeight`, `scrollSound` |
| `datetime` | Full property set |
| `rating` | Including `rotation` |
| `imagegrid` | Including `padding`, `autoLayoutSelectedZoom`, `gameImage`, `folderImage`, `scrollLoop` |
| `gridtile` | Including `size`, `backgroundCornerSize` |
| `helpsystem` | Full property set |
| `carousel` | Full property set, rendered in system view preview |
| `ninepatch` | Full property set |
| `sound` | Path only |

Additional RCADE/fork-specific properties may be present in your theme XML — these are preserved on import and export even if not editable in the UI.

---

## Known Limitations

- **`file://` origin required** — the tool runs as a local HTML file. Serving it from a web server will break relative asset loading.
- **`<include>` tags** are not followed — only the properties defined directly in the loaded `theme.xml` are parsed.
- **`<variables>` block** — theme-defined variables (e.g. `${cyan}`) are preserved in path and color strings but are not substituted in the preview. Only `${system.*}` variables can be previewed.
- **Video elements** render as a placeholder in the monitor — actual video playback is not supported.
- **Tile images** are approximated visually — true CSS tiling is not applied in the monitor preview.

---

## Tips

- Load the HTML from the **root of your theme folder** so that paths like `./_assets/images/bg.png` resolve correctly.
- Use **Shift+drag** to snap elements to 0.01 grid increments when aligning multiple elements.
- The **sidebar collapse** button is useful when fine-tuning element positions — collapse the sidebar to get a full-width monitor preview, then drag elements into place.
- Font inconsistencies (same filename referenced from different paths) will trigger a **warning banner** at the top of the screen.
- The `${system.*}` variable dialog is for preview only — set `system.theme` to a real system name that has assets in your theme folder to preview logo images in the carousel.
