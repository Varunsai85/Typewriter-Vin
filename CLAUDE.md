# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Typewriter-Vin is an Obsidian theme: `manifest.json` + a single `theme.css`. There is no build, lint or test step. Text is typed in a typewriter font on aged, faded-yellow book paper (think *48 Laws of Power*), with a dark "paper at night" variant.

## Deploying and checking changes

The source of truth is this folder. The user's active vault holds a copy that must be synced after every change:

```sh
V=~/"Rust Programming Language/Notes/.obsidian/themes/Typewriter-Vin"
cp theme.css README.md manifest.json "$V/" && diff -rq . "$V"
```

Obsidian may need a theme switch-and-back or restart to pick up changes.

There is no Obsidian test harness. To check visuals, build a small HTML mock-up that links `theme.css` and reproduces the relevant Obsidian DOM (`body.theme-light`/`theme-dark`, `.workspace-leaf-content[data-type="markdown"]`, `.markdown-rendered`, `.markdown-source-view.mod-cm6 .cm-line.HyperMD-codeblock`, …). Screenshot it with headless Brave (Chromium, same engine as Obsidian), then view the PNG:

```sh
"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" --headless=new --disable-gpu \
  --allow-file-access-from-files --hide-scrollbars --window-size=1400,1000 \
  [--force-device-scale-factor=2] --screenshot=out.png file:///path/to/preview.html
```

Check both light and dark mode, and a 2× zoom for texture work. Keep mock-ups out of this folder.

## theme.css layout

The numbered sections in the header comment are, in order: palette/variables, paper surfaces, typography, book details, interface, embedded font.

- **Theming works through variables.** `body` sets fonts and sizes. `.theme-light` and `.theme-dark` each define a private `--tw-*` palette (`--tw-paper`, `--tw-ink`, `--tw-ribbon-red`, `--tw-edge`, `--tw-bleed`, …) and map it onto Obsidian's variables (`--background-primary`, `--text-normal`, `--interactive-accent`, …). Rules later in the file use variables, so a color change normally belongs in the palette blocks only.
- **The paper texture is set on `.workspace-leaf-content`** for the `markdown`, `empty` and `bases` view types. It's a stack of background layers: a vignette gradient plus `--tw-mottle`, `--tw-pores` and `--tw-grain`. The inner Obsidian layers (`.view-content`, `.cm-editor`, `.cm-scroller`, …) are forced to `background-color: transparent !important` so the sheet shows through. A new view type that should look like paper needs adding to both selector lists.
- **Sidebars, the ribbon, tab headers, menus and modals get `--tw-grain` only in light mode.** Dark mode sets `background-image: none` on them.
- **The font is Special Elite**, embedded as base64 woff2 in two `@font-face` rules (latin and latin-ext `unicode-range`) at the very end of the file. The file is ~120 KB because of this, so read the earlier sections with an offset or limit. Code uses `"Courier New"` (`--font-monospace-theme`) because Special Elite's letters aren't all the same width.
- **Typewriter conventions are deliberate.** Special Elite has one weight, so bold is a faux double strike (`text-shadow`) and italics are rendered as underlines. `hr` becomes `*   *   *` via `::after`. Body text has a faint ink-bleed `text-shadow`.

## Texture variables

`--tw-grain`, `--tw-pores` and `--tw-mottle` are each one very long line holding a URL-encoded inline SVG (`%3C`, `%3E`, `%23`, `%25`), defined separately in `.theme-light` and `.theme-dark`. Edit them with a small script (regex-replace the whole `/* Paper texture… */` block in each theme) rather than by hand.

- **How the noise works:** each layer is `feTurbulence type='fractalNoise'` → `feColorMatrix`. The RGB rows set a constant color, and the alpha row `0 0 0 slope intercept` turns noise into coverage. A steep slope (e.g. `-30 7.3`) gives sparse, crisp pores; a gentle slope gives soft clouds. Light specks use a positive slope. `opacity` on the `<rect>` caps intensity.
- **Avoiding seams:** make `baseFrequency × tile size` an integer and use `stitchTiles='stitch'`, or visible tile seams appear. The `background-size` values in the sheet rule must match the SVG `width`/`height`.
- **When replacing texture blocks,** make sure the old block is actually removed. Duplicate definitions silently let an earlier one win.

## Decisions the user has made

- **Texture:** pores/grain plus soft mottling. The user rejected visible fiber streaks ("strikes") and a lit, embossed `feDiffuseLighting` "felt" texture (asked to revert it).
- **Code blocks:** no borders or lines. In Live Preview every code line is its own `.HyperMD-codeblock` element, so any border draws a rule under each line. Code text is thickened with `-webkit-text-stroke: 0.4px currentColor` because Courier New is too thin.
- **Accent:** red "second ribbon" (`--tw-ribbon-red`) for links, tags, checkboxes and the caret.

## Licensing

Special Elite is Apache 2.0 (`FONT-LICENSE.txt`). Keep that file alongside the theme if the font is embedded.
