# Pixel Studio: notes for Claude

Kris's standalone pixel art editor. Live at krismcnulty.github.io/pixel-studio (GitHub Pages, branch `main`, root).

## Shape of the code

- Everything is in `index.html`: plain HTML/CSS/JS, no build step and no dependencies. Keep it that way unless Kris asks otherwise.
- `PROJECTS` at the top of the script lists the apps the studio makes art for:
  - `habit-league` makes Habit League cosmetics (tabs: Team logo, kits, player backgrounds, heads; Arena and Effects to come) with the app's own code.
  - `sketchbook` is free drawing at 16 to 64 pixels, for football badges, kits and ideas.
  - `test` (Testing area) is for experiments at 32/48/64. With `compare:true`, an uploaded image is converted at every size (`compare()`, using `fromImage`) and shown side by side at card (68px) and profile (134px) size; **Edit** loads one into the canvas. Nothing submits anywhere.
- **Tabs (kinds).** A project with `kinds` shows tabs from `TABS` (Habit League: Team / Arena / Effects, matching the app's Locker). Each tab's settings in `KINDS` are merged over the project's into `P`, and `openKind()` sets up its editor. `openProject()` only loads the engine.
  - `mode:'pixel'` tabs use the pixel editor: heads (64×64 for new heads, 32×32 for old ones) and team logos (16×16).
  - `mode:'form'` tabs start from an existing item and change its settings: kits (colours, pattern, number font) and player backgrounds (swap the hex colours in an existing `--pbg`). `FORM` holds the current settings.
  - `soon:n` tabs are shown but not usable yet (Arena = phase 2, Effects = phase 3).
  - Each tab has its own storage: heads keep `pixelStudio.v1.habit-league`; other tabs add `.<kind>` (e.g. `pixelStudio.v1.habit-league.kit`), with `.folders` and `.wip` after that.
  - Tabs that need engine functions the app doesn't have yet (`need`) show a note instead of the editor.
- **Engine.** A project with `engine` loads that app in a hidden iframe and reads `window.HL_ART`.
  - What the engine provides:
    - `svg(grid)` and `headGrid(id)`
    - `heads()`
    - `base(skin, shape)` and `build(parts)`
    - the `skins`, `hairs`, `shapes` and `parts` lists
  - This only works because every app is on the same origin (krismcnulty.github.io).
  - The engine page is fetched and run via `srcdoc` with a guard script that gives it in-memory `localStorage` and no service worker. Keep this: the app saves on every render and re-renders every minute, so an unguarded hidden copy overwrites real progress made in the app.
  - Only `svg` is required; the parts kit appears when the engine has both `build` and `parts`.
  - Phase 1 adds `logos()`, `logoGrid(id)`, `logoSvg(grid)`, `kits()`, `kitPatterns`, `kitFonts`, `kitSvg(def, name, num)`, `pbgs()` and `pbgStyle(bg, size)`.
  - Phase 1 details (as built in the app):
    - `logoGrid(id)` returns a fresh 16×16 copy (cells `'#rrggbb'` or null); `logoSvg(grid)` draws with the app's `gridSvg` (smoothed and shaded).
    - `kits()` leaves out fields a kit doesn't use and skips the hidden `scout` kit. `kitSvg` also accepts a kit id. The Home kit's number outline (none) and the Away kit's dark one only apply when passed by id; a definition without `id` gets a unique internal id.
    - `kitFonts` lists all nine fonts the app loads, not only the ones kits use today. To preview them here, load the same font files (e.g. from `/habit-league/fonts/`). No `font` means the default bold sans.
    - `pbgs()` adds `base` to custom backgrounds. Brick wall's colour inside its data URL is written `%23140a06`; the app's swap handles `#` and `%23`, so match both. Classic uses `var(--panel2)`/`var(--well)`, and `transparent` stays a keyword.
    - `pbgStyle(bg, size)` returns `background:…;background-size:…` with `var(--…)` resolved to the current atmosphere. It can contain double quotes, so set it with `el.style.cssText` or escape `"`. It's a still image (no Holo/Snowfall animation).
    - Swap rules: an 8-digit key matches only that colour; a 6-digit key matches 6- and 8-digit colours and keeps the alpha.
  - Intake extras: a pbg `base` must be a built-in background and every `colors` key must be in it; kit `sw` 0–8, `fw` whole 100–900; colours saved lowercase; the same id may be used in different types. Previews: heads `art/previews/<id>.png`, others `<type>-<id>.png`. `art/drafts.json` can hold any type, so check `type`.
- **Ids** are 2 to 20 lowercase letters or numbers, unique within their type (a logo and a kit may share one). An opened design keeps its id only while its name is unchanged, so renaming makes a new design. Names that belong to a built-in item of the same type (in the engine's list, e.g. `HL_ART.kits()`, but not in `art/library.json`, plus `HIDDEN_IDS` such as the `scout` kit) are blocked, because the intake rejects them.
  - The API is defined in the Habit League repo (`Krismcnulty/habit-league`, `index.html`, search `HL_ART`). Change both sides together.
  - Without an engine, `basicSvg()` renders the preview.
- **Designs** are JSON shaped like `{type, id, name, pal:{letter:hex}, px:[rows]}`.
  - Heads add `rarity` (`rare`/`epic`/`leg`) and `where` (`shop`/`none`), with no drawn outline (the app adds it). New heads are 64×64, turned slightly to the left; old ones are 32×32 straight on and still open (⤢ Double to 64×64 prepares one for redrawing, keeping its identity if it's a library head).
  - Sketchbook designs are `type:'sprite'` with an `outline` boolean.
  - Logos: `type:'logo'`, 16×16, same encoding as heads.
  - Kits: `{type:'kit', id, name, rarity, where, b, t, num, pat, ns?, sw?, font?, fw?}`.
  - Player backgrounds: `{type:'pbg', id, name, rarity, where, base, colors:{'#oldhex':'#newhex'}}`. `base` must be a built-in background. Keys are the base's colours as written, lowercase, with `%23…` (inside encoded images) written as `#…`.
    - The app's rule: an 8-digit key changes only that exact colour, to exactly the value given; a 6-digit key changes the 6- and 8-digit forms and keeps the alpha.
    - So for a see-through (8-digit) colour the studio saves the new colour with the original alpha appended (`'#0a0c1e80':'#22c55e80'`), or the app would make it solid. `pbgBg()` mirrors the app exactly; keep the two in step.
  - Kit fonts are any of `HL_ART.kitFonts`; `engineFonts()` copies the engine's @font-face rules into the page so they preview. The id `scout` is reserved for kits.
  - `art/drafts.json` and `art/library.json` hold every type, so lists filter by `type` (missing = head).
- **Storage** is localStorage `pixelStudio.v1.<project>` (plus `.<kind>` for tabs other than heads), with folders in `<key>.folders`.
  - The design in progress autosaves to `<key>.wip` (from `draw()` or a form change, debounced, and on pagehide). Pixel tabs save the grid; form tabs save `FORM`. `openProject` and `openKind` flush the old save before switching.
  - The last tab used is kept in `pixelStudio.v1.<project>.kind`.
  - The Habit League project copies designs over once from the old HL Studio key `hlStudio.v1`.
- **Where.** In the app, `where:'shop'` puts an item in the Item Shop pool (bought at its rarity's price); `where:'none'` makes it **free for every player** (the app only locks shop, pack, event and achievement items). There is no hidden option yet; that needs a Habit League change.
- **Unlock status.** `HL_ART.unlockInfo(type, id)` describes how an item is unlocked in the app ("Free for everyone", "Item Shop (rare)", "Reward: …", "Store pack: …", "Event: …"). `unlockNote()` shows it under Rarity/Where whenever one of your library items is open.
  - All 16 team logos and 12 kits that came with the app now live in `art/library.json` (Habit League v204). Their unlock rules stay in the app's code by id, so `where` and `rarity` are ignored for them: only the artwork and name update.
  - `HL_ART.ready` is a promise that resolves once the app's library has loaded; `logos()` and `kits()` are empty until then, so `openProject()` waits for it (up to 8 seconds).
- **Editing your own items.** "Start from" lists include your library items, marked "(yours, editable)" (backgrounds under "Yours"). Loading one calls `editLib()`: it keeps the id, name, rarity and where, so Submit updates it in the app (after a confirm). Loading anything else calls `notEditing()`, which clears that name so a built-in can't accidentally replace your item.
- **Submit to app / Save draft to repo** open a prefilled GitHub issue on the project's repo, labelled `art` or `art-draft`.
  - That repo's action (`.github/workflows/art.yml` plus `tools/art-intake.js`, both in the Habit League repo) validates the design, commits it to `art/library.json` or `art/drafts.json`, replies with a preview and closes the issue.
  - Any change to the design format must stay compatible with `art-intake.js`.

- **Canvas input.** Zoom is a CSS transform on `#cv` inside `#cvWrap`; `cell()` uses the transformed rect, so drawing maths needs no zoom handling. A second finger cancels the stroke in progress and starts a pinch. Fill, pick and replace act on pointer release, so a pinch never triggers them.
- **Card size.** `cardPx`, `cardBorder` and `cardScale` make the small previews match a player card in the app: Habit League cards are a 68px frame with a 2px border. `cardScale` is screen pixels per art pixel for a 32×32 head (2); `cardSvg()` uses `cardScale*32/size`, so a 64×64 head is drawn at 1. Without a scale, `svg(grid)` stretches to fill its box. Change these whenever the app's card changes.
- **64×64 heads.** Supported by Habit League from v207 (`HL_ART.headSizes` = `[32, 64]`; the intake accepts both). `headOk(n)` is true for 32, and for 64 once the app lists it in `HL_ART.headSizes`; until then 64×64 heads are previewed with `basicSvg`. The parts kit card is hidden (it only builds old-style 32×32 heads; the app keeps it for scouted players). `HEAD_RULES` is the 64×64 AI rule set, `HEAD_RULES_32` the old one, picked by the grid size.
- **Head style reference.** `STYLE_REF` is Kris's 64×64 "Afroman style" head (palette-snapped, turned slightly left): the look all new heads should match. For 64×64 heads the AI prompt includes it (or another 64×64 library head picked under "Match the style of") and, on an empty grid, asks the AI to build on its layout. Pasted AI heads are snapped to the game palette (`aiSnap`), like Snap to the palette does for images. Snap to the palette is what gives heads their clean look.
- **Clean up.** `cleanUp()` works on the grid in place: merge colours with `kmeans()` (farthest-point start weighted by pixel count, so small distinct colours like eye whites survive), remove speckles (a pixel with no same-coloured 4-neighbour takes the main neighbour colour unless it's very different), then a gentle contrast boost. The AI can't reliably make small edits to a 64×64 grid (it tends to redraw), so Clean up is the tool for "keep it, make it crisper".
- **Image import colours.** Snap to the palette, keep its own colours, or reduce to 24/16/12/8 colours (`kmeans()`), which gives the crispest features.
- **Skin/hair swap** matches the drawing against "families" of aligned colour ramps: the guide's palette tables, plus ramps worked out from heads the engine draws (`ART.base` per skin, `ART.build` per hair colour).

## Related

- The original HL Studio (`/habit-league/studio.html` in the Habit League repo) stays as it is for now. Kris plans to switch to this one eventually.
- The art style rules are in the claude.ai project doc "Pixel Art Guide": heads (64×64, turned slightly left, lit from the top left, clearly defined features, 5–6 skin and 3–4 hair tones, no shoulders, original characters only; old 32×32 heads documented too), team logos (16×16, flat colours, no outline or shading), kits and player backgrounds. Keep the AI prompt rules (`HEAD_RULES`, `LOGO_RULES`) in step with it.
- A football version of Habit League is planned. When it exists, it gets its own repo and a new `PROJECTS` entry here (see README).

## Working rules

- Test with Playwright before pushing. Serve a folder that holds both repos side by side so the engine path `/habit-league/index.html` resolves, e.g. symlinks `site/habit-league` and `site/pixel-studio`, then `python3 -m http.server`.
- Check desktop (1400 wide) and phone (390 wide).
- Commit and push to `main` when a change is done.
