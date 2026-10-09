# Pixel Studio

A pixel art editor for Kris's games. Live at **krismcnulty.github.io/pixel-studio**. It works on a phone or a computer.

## Projects

Pick a project from the menu at the top. Each project keeps its own designs and folders.

| Project | What it's for | Engine | Submits to |
|---|---|---|---|
| Habit League | 32×32 player heads | Habit League's own drawing code (`/habit-league/index.html`) | `Krismcnulty/habit-league` |
| Sketchbook | Anything at 16, 24, 32, 48 or 64 pixels: football badges, kits, ideas | Built in | Nothing (save, PNG and files only) |

For a project with an engine, the studio loads that app in a hidden frame and draws with its code (`window.HL_ART`). The preview matches the app exactly, and the parts kit and "existing head" come from the app. This only works because every app is on the same site (krismcnulty.github.io).

The first time you open the Habit League project, any designs saved in the old HL Studio (`/habit-league/studio`) are copied across, folders included.

## Features

- Pen, erase, fill, colour picker, mirror, undo/redo. Shortcuts: B, E, G, I, M, Ctrl+Z, Ctrl+Y. Right-click erases.
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

## Adding a project (e.g. the football app)

1. In the app, expose its drawing code on `window.HL_ART` (see Habit League's `index.html`). It needs:
   - `svg(grid)`, `heads()`, `headGrid(id)`, `base(skin, shape)`, `skins`, `hairs` and `shapes`
   - optionally `build(parts)` and `parts`, which turn on the parts kit
2. Copy `.github/workflows/art.yml` and `tools/art-intake.js` from the Habit League repo into the app's repo. Then create the labels `art` and `art-draft` there.
3. Add an entry to `PROJECTS` at the top of the script in `index.html`, for example:

```js
'football':{name:'Football League',note:'Player heads',type:'head',sizes:[32],outline:'always',fields:true,
  engine:'/football-league/index.html',app:'/football-league/',repo:'Krismcnulty/football-league',drafts:'/football-league/art/drafts.json'}
```
