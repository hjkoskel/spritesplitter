# Sprite Splitter

A single-page browser tool for cutting sprite sheets into separate PNG files. You can also clip **sub-pictures** out of each frame, for example the small part of an animation frame that changes. The result downloads as one ZIP with whatever folder layout you choose.

**▶ Use it here: [hjkoskel.github.io/spritesplitter](https://hjkoskel.github.io/spritesplitter)**

- One `index.html`, no build step, no libraries, no server.
- Everything runs locally in your browser. Images are never uploaded.
- PNG output keeps transparency. Pixel-art friendly: no smoothing when zooming.

---

## Quick start

1. **Open a sprite sheet** with the button, drag-and-drop, or paste it (`Ctrl+V`).
2. **Define frames.** Drag rectangles on the image, add rows by hand, or use **Add a grid of frames** for regular sheets and animation strips.
3. **Give each frame a path** such as `player/walk/frame_01.png`. Folders are created inside the ZIP. `.png` is added if missing.
4. *(Optional)* **Add sub-pictures** to frames (see below).
5. Press **Process → ZIP**. Besides the images, the ZIP root contains `<zipname>.json`, the frame table in the import/export format, so you can load the same cut again later.

## Frames

Each table row is one frame: `x0`, `y0`, `w`, `h` in sheet pixels, plus an output path.

| On the image | Action |
|---|---|
| Drag on empty area | Create a new frame |
| Drag inside a frame | Move it |
| Drag the corner handle | Resize |
| `Alt` + drag | Create a new frame even on top of an existing one |
| Arrow keys | Nudge selected by 1 px (`Ctrl`: 8 px) |
| `Shift` + arrow keys | Resize selected |
| `Delete` / `Esc` | Delete selected / clear selection |
| `Ctrl` + mouse wheel | Zoom |

At 8× zoom and above, a pixel grid is drawn.

### Grid generator

Set the first cell's `x0`, `y0`, `width`, `height`, then the number of `columns` and `rows`, the `gap x` / `gap y` between cells, and a name pattern:

| Token | Meaning |
|---|---|
| `{i}` | running index, starting from *start #* |
| `{r}` / `{c}` | row / column |
| `{i:3}` | zero-padded → `007` |

Example: `anim/walk_{i:2}.png` → `anim/walk_00.png`, `anim/walk_01.png`, …

## Sub-pictures

Retro sprite animations often change only a small area between frames, such as a head, a hand or a muzzle flash. A sub-picture is a rectangle inside a frame that is exported as its own file.

- Add one with **+ sub** on a frame row, or `Shift` + drag inside a frame.
- Sub-picture `x0` / `y0` are **relative to the frame's top-left corner**, so moving the frame moves its sub-pictures too.
- Each sub-picture has its own path, which can use these tokens from its frame:

| Token | Meaning |
|---|---|
| `{name}` | frame file name without `.png` (unnamed frame → `frame_01`, `frame_02`, …) |
| `{dir}` | frame's folder |
| `{n}` | frame number, 1-based; `{n:3}` → `001` |
| `{s}` | sub-picture number within the frame |

Default: `{dir}/{name}_part{s}.png` → frame `anim/walk_03` gives `anim/walk_03_part1.png`.

**Copy selected frame's subs to all frames** gives every frame the same sub-pictures. This is handy when the changing part sits in the same spot in every frame.

### Is the full frame exported?

| Frame path | Has sub-pictures | `out` ticked | Exported |
|---|---|---|---|
| set | no | – | frame |
| set | yes | yes | frame + sub-pictures |
| set | yes | no | sub-pictures only |
| **empty** | yes | – | sub-pictures only |
| empty | no | – | ✖ error, nothing to export |

The line under each path shows the final resolved file name. Empty or duplicate output paths are marked red and block processing. Rectangles that go past the image or frame edge are marked orange; the area outside comes out transparent.

## Preview

Click any thumbnail in the table to open a large preview with sharp pixels.

- **◀ / ▶** step through frames. On a sub-picture, they step through the *same* sub-picture number in every frame, so you watch only the changing part.
- **Play** animates the sequence at the chosen fps. The scale stays fixed so the animation doesn't jump.
- Background: checkerboard, black, white or magenta.
- Keys: `←` `→` step, `Space` play/pause, `+` `−` scale, `0` fit, `Esc` close.

## Saving the frame table

**Import / export…** shows the table as JSON. You can edit it, copy it, save it as `.json`, or load a file. Reusing a table on another sheet with the same layout is quick.

```json
{"zip": "hero.zip", "clips": [
  {"x":0,"y":0,"w":32,"h":32,"path":"hero/walk_00.png","out":true, "subs": [
    {"x":8,"y":2,"w":16,"h":12,"path":"hero/heads/head{n:2}.png"}
  ]}
]}
```

CSV is also accepted, one frame per line, with `sub,…` lines attaching sub-pictures to the frame above:

```csv
x,y,w,h,path
0,0,32,32,hero/walk_00.png
sub,8,2,16,12,hero/heads/head{n:2}.png
32,0,32,32,hero/walk_01.png
```

The current table, ZIP name, panel width and preview background are also remembered in the browser's `localStorage`. The image itself is not stored.

## Layout

Drag the bar between the image and the table to resize the side panel. Double-click it to reset. Table inputs widen with the panel.

## How it works

- Clips are drawn to an off-screen `<canvas>` and encoded with `canvas.toBlob("image/png")`.
- The ZIP is written by a small built-in writer: *store* method (PNGs are already compressed), CRC-32, UTF-8 file names, and folder entries.
- No dependencies and no network requests.

## Install as an app (PWA)

Sprite Splitter is an installable Progressive Web App and works fully offline after the first visit.

- **Chrome / Edge (desktop & Android):** click **⤓ Install app** in the header, or use the install icon in the address bar or the browser menu → *Install*.
- **Safari (iOS / macOS):** Share → *Add to Home Screen* / *Add to Dock*.

When installed on Chromium-based desktop browsers, the app also registers as a handler for PNG, GIF, WebP, BMP and JPEG files, so you can right-click an image → *Open with → Sprite Splitter*.

## Repository layout

```
index.html              the whole app
manifest.webmanifest    PWA manifest (name, icons, file handlers)
sw.js                   service worker: offline cache
icons/                  app icons (any + maskable), favicon, apple-touch-icon
```

All paths are relative, so the app works under a sub-path such as `/spritesplitter/`.

## Running locally / hosting

Open `index.html` directly in a browser; everything except offline install works from `file://`. For the full PWA, serve the folder over HTTP:

```sh
python3 -m http.server
```

GitHub Pages: push these files to the repository root (or `/docs`) and enable Pages for that branch in **Settings → Pages**.

### Releasing an update

Installed copies load the newest `index.html` whenever they are online, and fall back to the cached copy offline. If you change the icons, manifest or the file list, bump `VERSION` in `sw.js` (e.g. `"v2"`) so old caches are dropped.
