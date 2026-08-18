# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UI Anatomy is a zero-dependency, ES-module-based interactive wireframe simulator. Users hover over white outlined mockup components to learn their names, variants, and AI prompt tips. The app teaches UI vocabulary without build tools or dependencies.

**Live:** uianatomy.neorgon.com

**Run locally:** `make serve` → http://localhost:8820 (or `python3 -m http.server 8820`)

## Architecture

### Modular ES Module Structure

```
ui-anatomy-site/
├── index.html          # App shell (~200 lines with metadata)
├── css/
│   └── style.css       # Single stylesheet: Neorgon dark theme + wireframe styling
└── js/
    ├── app.js          # Entry point: wires render + events on DOMContentLoaded
    ├── state.js        # Central state object (active layout, hovered comp, toggles)
    ├── data.js         # 50+ COMPONENT definitions, LAYOUT_COMPONENTS, CATEGORIES, checklist + mistakes data
    ├── layouts.js      # Wireframe HTML generators (landing/corporate/startup); layouts2–6.js cover the rest
    ├── render.js       # All rendering: tabs, mockup, browser, tooltip, highlight sync
    ├── quiz.js         # Quiz mode: question generation, scoring (localStorage quizStats), panel render
    ├── prompt.js       # Layout → AI prompt export: builds prose from the rendered DOM, modal + clipboard
    ├── events.js       # Event delegation: hover, clicks, search, quiz + prompt wiring
    └── utils.js        # debounce, escHtml, prefersReducedMotion
```

**Key patterns:**
- State is a mutable exported object; modules import and mutate it directly
- Render functions read `state` and write to DOM via `innerHTML`; no virtual DOM
- Event delegation on containers; no per-element listeners
- localStorage is used only for checklist checkmarks (`checklistChecked`) and quiz stats (`quizStats`); everything else resets on reload
- Quiz mode swaps the component browser for the quiz panel and suppresses tooltips (they would leak answers); quiz questions read the component pool from the rendered DOM, as does the prompt export. Both stay in sync with the visible mockup by construction

### State Shape

```js
state = {
  activeLayout: 'landing',    // one of the 21 layout ids (8 basic + 13 industry; see LAYOUTS in data.js)
  activeComp: null,           // id string of hovered component
  pinnedComp: null,           // id of clicked/pinned component
  browserOpen: true,          // sidebar toggle
  outlinesOn: true,           // show wireframe boundaries
  searchQuery: '',            // component search filter
  dummyMode: true,            // render readable text + images vs placeholders
  heroBg: 'solid',            // hero background style (swatch picker)
  quizMode: false,            // quiz panel replaces browser; mockup clicks answer questions
  quizStats: { ... },         // answered/correct/streak/best, hydrated from localStorage by quiz.js
}
```

### Data Model

**COMPONENTS** (in `data.js`): Object map of component metadata
- Key: component id (kebab-case)
- Value: `{ name, also: [], desc, variants: [], tip, frameworks: [], category }`

**LAYOUT_COMPONENTS**: Map from layout id to array of component ids present in that layout

**CATEGORIES**: Map from category key to display label

**LAYOUTS**: Array of `{ id, label }` for the tab bar

### Deep internals (rendering, layouts, tooltip, checklist)

**📖 Read `docs/references/rendering-internals.md` before changing** the render pipeline, the wireframe layout generators (`layouts.js`/`layouts2.js` + `makeHelpers` factories), tooltip/hover sync (`showTooltip`/`syncBrowserHighlight`), or the checklist layout (localStorage persistence). All hoverable elements must carry `data-comp="component-id"` matching a `COMPONENTS` key.

## Development Commands

- `make serve`: Start HTTP server on port 8820 (ES modules require HTTP, not file://)
- `make kill`: Kill server on port 8820
- `python3 -m http.server 8820`: Alternative if make isn't available

## Design Constraints

- **No build step**: App must run on static hosting (GitHub Pages)
- **No dependencies**: Vanilla JS only, no npm packages
- **ES modules**: Use `type="module"` and import/export; requires HTTP server in development
- **Neorgon brand**: Dark theme, `rgba(255,255,255,.03)` glass cards, `#0063e5` blue accent
- **Accessibility**: Semantic HTML, ARIA attributes on tabs and search, `.sr-only` utility class available
- **Mobile**: Responsive CSS Grid/Flexbox; sidebar stacks below mockup on narrow screens
