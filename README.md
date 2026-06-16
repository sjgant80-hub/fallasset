# FallAsset

**Sovereign asset browser + non-destructive editor — one HTML file.**

A single-file replacement for Adobe Bridge (asset browser) and Lightroom (rate, tag, label, lightly edit). Your folder of images, your metadata, your edits — all on your device. No upload, no account, no telemetry.

- **Prime** · 467
- **Version** · 1.0.0
- **Phase** · FallStudio · phase 3
- **Seal** · ◊·κ=1 · MIT

---

## For people who want to use it

Open `index.html` in a browser. That's it. No install.

1. Press **pick folder** in the header.
   - On **Chrome/Edge** this uses the File System Access API and walks the whole directory tree, including subfolders.
   - On **Firefox/Safari** (no API support) it falls back to the multi-file picker, or you can drag a folder onto the empty-state panel.
2. Thumbnails lazy-load as you scroll. Use the slider in the toolbar to size them (80–360px).
3. Click a thumbnail to enter **loupe** view. Use **←/→** to navigate, **Esc** to return.
4. Cmd-click (or Ctrl-click) thumbnails to add to selection, then switch to **compare** view (2–4 images side by side).
5. The right panel holds metadata:
   - **info** tab: title, caption, tags (comma-separated, autocomplete), rating (0–5★), color label (red/yellow/green/blue/violet/none), and which collections this image belongs to
   - **edit** tab: crop (drag a rectangle on the loupe image), rotate 90°, flip H/V, and 7 adjust sliders — brightness, contrast, saturation, hue, blur, sepia, grayscale
6. **Collections** — left sidebar. Tap `+` to make one. Right-click any thumb to add. Click a collection to filter the grid to it.
7. **Filters** (left sidebar): rating, color label, file type, tag — all combine with AND.
8. **Sort** (toolbar): filename, date modified, size, or rating · asc / desc.
9. **Export with edits** — in the edit tab, hit **export PNG** or **export JPG**. Edits bake onto a canvas and download.
10. **Ctrl+K** opens **Ω autopilot**. Try: `show 5-star photos` · `rate this 4` · `tag landscape sunset` · `compare these two` · `rotate 90` · `sort by date desc` · `export PNG`.

All metadata, ratings, tags, labels, collections, and edit settings live in your browser's **IndexedDB**. Use **export metadata** in the sidebar to save a JSON snapshot, **import metadata** to restore.

### Edits are non-destructive

Edits never touch your original files. They live in IndexedDB keyed by file path + size + mtime. The same image, edited in FallAsset, can be re-opened anywhere unchanged. Hit **export PNG / JPG** when you want a baked copy.

### Browser support

| Browser | Folder picker | Drag-drop folder | File picker |
|---|---|---|---|
| Chrome 86+ / Edge | yes | yes | yes |
| Firefox | no | yes (webkit entries) | yes |
| Safari 15+ | no | yes (webkit entries) | yes |

Folder picker requires **HTTPS or `file://`** and a user gesture (the button click). Already handled.

---

## For people who want to read the code

One file. Vanilla JS. No build step. No npm. No CDN.

### Engine stack

- **File System Access API** (`window.showDirectoryPicker`) — walks the directory tree recursively, collects image files, stores `FileSystemFileHandle` per image. Graceful fallback to `<input type="file" webkitdirectory multiple>` and a `webkitGetAsEntry` drop zone.
- **IndexedDB** (`fallasset` db, two stores: `meta`, `collections`) — primary persistence. Keyed by a stable id derived from `SHA-1(path + size + lastModified)`.
- **Canvas 2D** — used for both the loupe preview (so live edits show) and the export bake (PNG/JPG with `toBlob`).
- **CSS filters** — same 8-slider pattern as FallMage (brightness, contrast, saturation, hue, blur, sepia, grayscale).
- **IntersectionObserver** — thumbnails only load their `<img>` when scrolled near. Critical for folders of thousands.

### Cascade (T0 / T2 / T3)

The shared estate Cascade pattern, verbatim:
- **T0** — fully offline. The keyword router (`t0Route`) handles every required command.
- **T2** — detected when Ollama is running at `127.0.0.1:11434`.
- **T3** — BYOK. Anthropic / Gemini / OpenAI / OpenRouter keys live in `localStorage` and `state.settings`. Routes a JSON intent for Ω.

The T3 path is purely an enhancement — no feature requires it. With keys set, Ω can write captions from filenames, and the JSON-action router handles freer phrasing.

### Ω autopilot

`Ctrl+K`. T0 regex router fires first against every intent. If it matches, done. If not and a key is set, the LLM returns a strict JSON object `{action, args}` that `execAction()` switches over. Either path mutates the same internal state.

### Estate snippets

All present:
- `Cascade` (T0/T2/T3) — `class Cascade` definition with `_probe`, `detectTier`, `generate`
- `KONOMI` sovereign shim — `window.KONOMI = {active:true, tier:'sovereign', prime:467, ...}`
- `fallmesh` BroadcastChannel — `new BroadcastChannel('fall-signal')`, `hello` on boot, `pong` on ping
- `postMessage` API — `window.addEventListener('message', ...)`, responds to `{target:'fallasset', action:'ping'}`
- PWA manifest via `data:application/manifest+json` URL with `◊` brass-amber icon

### State shape

```javascript
state = {
  files,            // [{id, name, path, type, size, lastModified, file, handle?, url}]
  meta,             // {id: {title, caption, tags, rating, label, dimensions, edits}}
  collections,      // [{id, name, items:[fileId]}]
  view,             // 'grid' | 'loupe' | 'compare'
  loupeIdx,
  sel,              // Set<fileId>
  thumbSize,
  sort: {by, dir},
  filt: {ratingMin, label, type, tag, collection},
  settings          // BYOK keys (localStorage)
}
```

### 14-point gate

| # | Constraint | Status |
|---|---|---|
| 1 | single HTML file, sovereign, works from file:// | yes |
| 2 | <400KB | yes |
| 3 | L1 FACE — domain views (grid, loupe, compare, metadata, edit) | yes |
| 4 | L2 SWARM — Ω palette + 7 named ops | yes |
| 5 | L3 CASCADE — T0 fully works offline | yes |
| 6 | L4 BLOOM — input routes to right action | yes |
| 7 | L5 PERSIST — IndexedDB primary, JSON export/import | yes |
| 8 | L6 SKIN — mobile-first, estate dark palette | yes |
| 9 | L7 ASS — empty state is helpful | yes |
| 10 | Konomi licence shim baked | yes |
| 11 | fallmesh BroadcastChannel `fall-signal`, prime 467 | yes |
| 12 | PWA manifest baked via data: URL | yes |
| 13 | README + MIT LICENSE | yes |
| 14 | .nojekyll | yes |

### File deliverables

```
fallasset/
├── index.html        ← the tool
├── README.md         ← this file
├── LICENSE           ← MIT
└── .nojekyll         ← empty (for GitHub Pages legacy deploy)
```

### Hack on it

The whole tool is in `index.html`. CSS at the top of `<style>`, JS at the bottom of `<script>`. Search for the section banners (`/* ───── GRID ───── */` etc.) to navigate. No build step — edit, save, reload.

---

**FallAsset** · part of the FallStudio plan · ◊·κ=1 · MIT
