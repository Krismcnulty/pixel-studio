# Pixel Studio: notes for Claude

Kris's standalone pixel art editor. Live at krismcnulty.github.io/pixel-studio (GitHub Pages, branch `main`, root).

## Shape of the code

- Everything is in `index.html`: plain HTML/CSS/JS, no build step and no dependencies. Keep it that way unless Kris asks otherwise.
- `PROJECTS` at the top of the script lists the apps the studio makes art for:
  - `habit-league` draws player heads with Habit League's own code.
  - `sketchbook` is free drawing at 16 to 64 pixels, for football badges, kits and ideas.
- **Engine.** A project with `engine` loads that app in a hidden iframe and reads `window.HL_ART`.
  - What the engine provides:
    - `svg(grid)` and `headGrid(id)`
    - `heads()`
    - `base(skin, shape)` and `build(parts)`
    - the `skins`, `hairs`, `shapes` and `parts` lists
  - This only works because every app is on the same origin (krismcnulty.github.io).
  - The engine page is fetched and run via `srcdoc` with a guard script that gives it in-memory `localStorage` and no service worker. Keep this: the app saves on every render and re-renders every minute, so an unguarded hidden copy overwrites real progress made in the app.
  - Only `svg` is required; the parts kit appears when the engine has both `build` and `parts`.
- **Ids** are 2 to 20 lowercase letters or numbers. An opened design keeps its id only while its name is unchanged, so renaming makes a new design. Names that belong to the app's built-in heads (in `HL_ART.heads()` but not in `art/library.json`) are blocked, because the intake rejects them.
  - The API is defined in the Habit League repo (`Krismcnulty/habit-league`, `index.html`, search `HL_ART`). Change both sides together.
  - Without an engine, `basicSvg()` renders the preview.
- **Designs** are JSON shaped like `{type, id, name, pal:{letter:hex}, px:[rows]}`.
  - Heads add `rarity` (`rare`/`epic`/`leg`) and `where` (`shop`/`none`), and must be 32×32 with no drawn outline (the app adds it).
  - Sketchbook designs are `type:'sprite'` with an `outline` boolean.
- **Storage** is localStorage `pixelStudio.v1.<project>`, with folders in `pixelStudio.v1.<project>.folders`.
  - The drawing in progress autosaves to `pixelStudio.v1.<project>.wip` (from `draw()`, debounced, and on pagehide). `openProject` flushes the old project's save before switching.
  - The Habit League project copies designs over once from the old HL Studio key `hlStudio.v1`.
- **Submit to app / Save draft to repo** open a prefilled GitHub issue on the project's repo, labelled `art` or `art-draft`.
  - That repo's action (`.github/workflows/art.yml` plus `tools/art-intake.js`, both in the Habit League repo) validates the design, commits it to `art/library.json` or `art/drafts.json`, replies with a preview and closes the issue.
  - Any change to the design format must stay compatible with `art-intake.js`.

- **Canvas input.** Zoom is a CSS transform on `#cv` inside `#cvWrap`; `cell()` uses the transformed rect, so drawing maths needs no zoom handling. A second finger cancels the stroke in progress and starts a pinch. Fill, pick and replace act on pointer release, so a pinch never triggers them.
- **Card size.** `cardPx`, `cardBorder` and `cardScale` make the small previews match a player card in the app: Habit League cards are a 68px frame with a 2px border, and heads are drawn with `HL_ART.svg(grid, 2)`, which gives every art pixel exactly 2 screen pixels (52 to 64px per head). Without a scale, `svg(grid)` stretches to fill its box as before. Change these whenever the app's card changes.
- **Skin/hair swap** matches the drawing against "families" of aligned colour ramps: the guide's palette tables, plus ramps worked out from heads the engine draws (`ART.base` per skin, `ART.build` per hair colour).

## Related

- The original HL Studio (`/habit-league/studio.html` in the Habit League repo) stays as it is for now. Kris plans to switch to this one eventually.
- The art style rules are in the claude.ai project doc "Pixel Art Guide": 32×32, lit from the top left, five skin tones, three hair tones, no shoulders, original characters only.
- A football version of Habit League is planned. When it exists, it gets its own repo and a new `PROJECTS` entry here (see README).

## Working rules

- Test with Playwright before pushing. Serve a folder that holds both repos side by side so the engine path `/habit-league/index.html` resolves, e.g. symlinks `site/habit-league` and `site/pixel-studio`, then `python3 -m http.server`.
- Check desktop (1400 wide) and phone (390 wide).
- Commit and push to `main` when a change is done.
