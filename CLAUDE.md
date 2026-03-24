# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

Single-file HTML app — `index.html` is the entire application. No build step, no dependencies, no frameworks. All logic, styles, and data are inline.

`guide-tanakh-laomek.html` is a separate teacher/user guide page (standalone, not linked to the main app state).

`extracted_pptx/` contains source PPTX files (reference material, not deployed).

## App Overview

**מובןלי — תנ״ך לעומק** is a Hebrew Bible close-reading tool. Users work through 5 steps:

| Step | Hebrew | Description |
|---|---|---|
| 0 | עימוד | Layout — break text into lines/paragraphs, set indentation |
| 1 | גילוי | Discovery — mark words with colors and styles |
| 2 | אריזה | Packaging — group words into named boxes |
| 3 | ארגון | Canvas — arrange packages visually, draw arrows |
| 4 | קריאה | Reading — final view with summary, legend, notes |

## State

Global state object `S` — everything persists here:
```js
S = {
  step, doc, font, darkMode,
  brks, indents, paraBreaks,   // step 0 layout
  fs, lh,                       // font size, line height
  cats, marks, activeCat, activeSub, vf,  // step 1 categories/marks/filter
  pkgs, pkgSel, pkgNewName, pkgNewCi, pkgNewStyle,  // step 2 packages
  nodes, conns, connMode, expandedPkgs,  // step 3 canvas
  stickies, labels, shapes, cvZoom, cvTool, shapeColor,
  notes,            // {0..4} notebook (one note per step)
  hiddenDocs,
  splash, showExport, showKeys, showUpload, showNotebook, nbTab, ...
}
```

`R()` is the main render function — always called after state changes.

## Key Data Structures

**Texts:** `TEXTS` object with keys: `psalms1, psalms13, psalms23, psalms67, psalms90, psalms100, psalms121, psalms131, psalms19`. Each has `{title, verses: [{n, t}]}`. `D()` returns the active text. `TEXTS.custom` can be set via paste/file upload.

**Categories (step 1):** `S.cats` array of `{id, name, styleId, ci, subs: [{id, name, ci}]}`. `STYLES` array (10 styles: highlight, underline, dashed, bold, etc.). `C` array (14 colors).

**Marks:** `S.marks` — keys are `"vi-wi"` (verse index, word index), values are `{catId, subId}`.

**Packages (step 2):** `S.pkgs` array of `{id, name, words: ["vi-wi"...], ci, style, emoji, parentId}`.

**Canvas nodes (step 3):** `S.nodes[i]` = `{x, y, w, h}` matching `S.pkgs[i]`. `S.conns` array of `{from, to, label, emoji, style, midX, midY}`.

**Canvas extras:** `S.stickies` (sticky notes), `S.labels` (text labels), `S.shapes` (rect/circle/diamond/line-h).

## Persistence

- **localStorage key:** `moovanly_save` — auto-saved every 60 seconds by `startAutoSave()`
- **JSON export/import:** `exportJSON()` / `importJSON()`
- **HTML export:** `exportHTML()` — injects full state into `data-saved-state` attribute on `<body>`; on reload a loader block reads it before `R()`
- **PDF:** `exportPrint()` — opens print window with cleaned HTML
- **PNG:** `exportImage()` — loads html2canvas from CDN

## Analytics

GoatCounter: `tanakh-laomek.goatcounter.com`

## Palette & Fonts

```
--bg: #F5EDE3   --text: #5A3A4A   --earth: #8B4A5E
--teal: #B8C9D4  --coral: #C48B9F
```

Dark mode via `html.dark` class, toggled by `S.darkMode`.

`FONTS` array: 7 Hebrew fonts. Default: `assistant`. `fontCSS()` returns the active font CSS string.

## Utility Functions

- `cv(light, dark)` — returns value based on dark mode
- `tc()`, `tcm()`, `bgc()`, `brd()`, `hov()` — themed color helpers
- `D()` — returns active text object
- `buildLines()` — builds rendered line array from `S.brks`, `S.indents`, `S.paraBreaks`
- `saveUndo()` / `doUndo()` / `doRedo()` — undo/redo (MAX_UNDO=50)
- `toast(msg)` — shows brief notification

## Side panels per step

`side()` dispatches to `sImud()`, `sDisc()`, `sPkg()`, `sCanv()`, `sRead()`.

Main content dispatches to `mImud()`, `mDisc()`, `mPkg()`, `mCanv()`, `mRead()`.

## Voice Dictation (step 0)

`toggleDictation()` — uses Web Speech API. Commands: "שורה", "פסקה", "ימינה", "שמאלה", "בטל", "מחק", or any word from the text to select it.

## Canvas (step 3)

SVG arrows drawn by `buildArrowsSvg()`. Arrow styles: straight, curved, elbow, custom (quadratic bezier via draggable midpoint). `autoLayout()` arranges nodes automatically. Minimap rendered by `renderMinimap()`.

## Demo

`loadDemo()` loads `psalms19` at step 4.

## Key Conventions

- **Edit tool fails on Hebrew strings** — use Python scripts with `str.replace()` for complex edits touching Hebrew text.
- Color palette is fixed — do not introduce new colors.
- No build, no deploy pipeline — the file is shared directly or via GitHub Pages if configured.
