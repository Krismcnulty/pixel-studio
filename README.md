# Pixel Studio

A pixel art editor for Kris's games. Live at **krismcnulty.github.io/pixel-studio**. It works on a phone or a computer.

## Projects

Pick a project from the menu at the top. Each project keeps its own designs and folders.

| Project | What it's for | Engine | Submits to |
|---|---|---|---|
| Habit League | Tabs for each cosmetic, like the app's Locker: Team (logo, kits, player backgrounds, heads) now; Arena and Effects later | Habit League's own drawing code (`/habit-league/index.html`) | `Krismcnulty/habit-league` |
| Sketchbook | Anything at 16, 24, 32, 48 or 64 pixels: football badges, kits, ideas | Built in | Nothing (save, PNG and files only) |

For a project with an engine, the studio loads that app in a hidden frame and draws with its code (`window.HL_ART`). The preview matches the app exactly, and the parts kit and "existing head" come from the app. This only works because every app is on the same site (krismcnulty.github.io).

The first time you open the Habit League project, any designs saved in the old HL Studio (`/habit-league/studio`) are copied across, folders included.

## Habit League tabs

- **Team Logo:** 16×16 pixel art. The app smooths and shades logos, so the preview shows it exactly as the app does.
- **Player Kits:** pick an existing kit to start from, then change the shirt, trim and number colours, number outline, pattern and number font.
- **Player Backgrounds:** pick an existing background, then swap any of its colours. Layout, texture and animation stay the same.
- **Player Heads:** 32×32 pixel art, as before.
- **Arena** (atmosphere, scoreboard, court, bench) and **Effects** (ball, shot style, win celebration, sound pack) are coming in later phases.

Each tab keeps its own designs and folders, and submits to the app in the same way.

## Features

- Pen, erase, fill, colour picker, replace, mirror, undo/redo. Shortcuts: B, E, G, I, R, M, Ctrl+Z, Ctrl+Y. Right-click erases.
- **Replace** (R): tap a colour on the drawing to change it everywhere to the selected colour.
- **Zoom:** pinch and drag with two fingers on a phone; mouse wheel, the − / + buttons, Space+drag or middle-drag to pan on a computer. **Fit** (or 0) shows the whole drawing.
- **Swap skin / Swap hair** (heads): changes every skin or hair tone at once, to another set or tinted with the selected colour, keeping the shading.
- **Autosave:** the drawing in progress and its details are kept in the browser for each project, so a closed tab or a refresh loses nothing.
- Auto outline: always on for Habit League heads, optional in the Sketchbook.
- **Describe it (AI):** type what you want (e.g. "zombie") and press **Copy AI prompt**. It copies a prompt with the art rules and your current drawing. Paste it into Claude, copy the reply, then press **Paste result**. Pasting forgives the usual AI slips: text around the code, rows of the wrong length, and letters missing from the palette.
- Start from an empty grid, an existing head, the parts kit, an image (cropped, shrunk, snapped to the palette), or pasted code.
- **My designs** has folders and is saved in the browser. **Download folder** and **Open file** move designs between devices as `.json` files.
- **Submit to app** and **Save draft to repo** open a ready-made GitHub issue on the project's repo. Its art intake action checks the design, commits it and replies with a preview.

## Design format

```json
{"type":"head","id":"skyhook","name":"Sky Hook","rarity":"epic","where":"shop","pal":{"a":"#1b120c"},"px":["...32 rows of 32 characters..."]}
```

- `px` is a square grid, one character per pixel, with `.` for empty.
- Sketchbook designs use `"type":"sprite"` and an `"outline"` true/false flag. They have no `rarity` or `where`.
- Habit League team logos use `"type":"logo"`, the same encoding at 16×16.
- Kits and player backgrounds are settings, not pixels:

```json
{"type":"kit","id":"teal","name":"Teal","rarity":"epic","where":"shop","b":"#0f766e","t":"#fde047","num":"#ffffff","pat":"hoops","font":"'Bebas Neue',sans-serif"}
{"type":"pbg","id":"redmist","name":"Red Mist","rarity":"rare","where":"shop","base":"mist","colors":{"#a7f3d0":"#ff0000"}}
```

- A kit's `pat` must be one of `HL_ART.kitPatterns` and `font` one of `HL_ART.kitFonts`; `ns` (number outline colour), `sw` (outline width) and `fw` (font weight) are optional.
- A player background copies a built-in one (`base`) and swaps the colours listed in `colors`.

## Adding a project (e.g. the football app)

1. In the app, expose its drawing code on `window.HL_ART` (see Habit League's `index.html`). It needs:
   - `svg(grid)`, `heads()`, `headGrid(id)`, `base(skin, shape)`, `skins`, `hairs` and `shapes`
   - optionally `build(parts)` and `parts`, which turn on the parts kit
   - optionally `logos()`, `logoGrid(id)`, `logoSvg(grid)`, `kits()`, `kitPatterns`, `kitFonts`, `kitSvg(def, name, num)`, `pbgs()` and `pbgStyle(bg, size)` for the logo, kit and background tabs
2. Copy `.github/workflows/art.yml` and `tools/art-intake.js` from the Habit League repo into the app's repo. Then create the labels `art` and `art-draft` there.
3. Add an entry to `PROJECTS` at the top of the script in `index.html`, for example:

```js
'football':{name:'Football League',note:'Player heads',type:'head',sizes:[32],outline:'always',fields:true,
  engine:'/football-league/index.html',app:'/football-league/',repo:'Krismcnulty/football-league',drafts:'/football-league/art/drafts.json'}
```
