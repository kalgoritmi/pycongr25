# PyCon GR 2025 Slide Deck

> Slidev-based slides for the PyCon Greece 2025 presentation.

## Prerequisites
- Node.js 18 or newer (includes npm 9+)
- macOS/Linux/Windows terminal that can run Node.js commands

## Install Dependencies
```bash
npm install
```
This pulls in the Slidev CLI, the default themes, and Playwright (used for exporting).

## Start the Slide Dev Server
```bash
npx slidev talk.md
```
Slidev launches at http://localhost:3030 with hot-module reloading.

## Export Slides
- Powerpoint export
```bash
npx slidev export talk.md --format pptx --output pycongr25.pptx --with-clicks
```

- PDF export
```bash
npx slidev export talk.md --output pycongr25.pdf --with-clicks

```

## Customize the Deck
- Edit `talk.md` to add new slides or tweak content.
- Refer to the [Slidev documentation](https://sli.dev) for syntax, components, and theming tips.
