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
├── index.html          # App shell (~140 lines with metadata)
├── css/
│   └── style.css       # Single stylesheet: Neorgon dark theme + wireframe styling
└── js/
    ├── app.js          # Entry point: wires render + events on DOMContentLoaded
    ├── state.js        # Central state object (active layout, hovered comp, toggles)
    ├── data.js         # 40+ COMPONENT definitions, LAYOUT_COMPONENTS, CATEGORIES, + 3 extra layouts
    ├── layouts.js      # Wireframe HTML generators for landing/corporate/startup layouts
    ├── layouts2.js     # generators for portfolio/blog plus components/login/checklist layouts
    ├── render.js       # All rendering: tabs, mockup, browser, tooltip, highlight sync
    ├── events.js       # Event delegation: hover, clicks, search
    └── utils.js        # debounce, escHtml
```

**Key patterns:**
- State is a mutable exported object; modules import and mutate it directly
- Render functions read `state` and write to DOM via `innerHTML`; no virtual DOM
- Event delegation on containers; no per-element listeners
- No localStorage; state resets on reload (except checklist checkmarks)

### State Shape

```js
state = {
  activeLayout: 'landing',    // one of: landing, corporate, startup, portfolio, blog, components, login, checklist
  activeComp: null,           // id string of hovered component
  browserOpen: true,          // sidebar toggle
  outlinesOn: true,           // show wireframe boundaries
  searchQuery: '',            // component search filter
  dummyMode: true,            // render readable text + images vs placeholders
  heroBg: 'solid',            // hero background style (swatch picker)
}
```

### Data Model

**COMPONENTS** (in `data.js`): Object map of component metadata
- Key: component id (kebab-case)
- Value: `{ name, also: [], desc, variants: [], tip, frameworks: [], category }`

**LAYOUT_COMPONENTS**: Map from layout id to array of component ids present in that layout

**CATEGORIES**: Map from category key to display label

**LAYOUTS**: Array of `{ id, label }` for the tab bar

### Rendering Pipeline

1. **app.js** on DOMContentLoaded → calls all render functions + initEvents
2. **renderTabs()** → populates `#layoutTabs` from `LAYOUTS` + `state.activeLayout`
3. **renderMockup()** → selects layout function from `layouts.js`/`layouts2.js`, calls it with `{ dummy: state.dummyMode }`, injects HTML into `#mockupFrame`
4. **renderBrowser()** → builds searchable, categorized component list sidebar; shows "missing" pills for components not in current layout
5. **renderHeroBgPicker()** → swatch buttons for hero background variants
6. **initEvents()** in `events.js` attaches delegated listeners:
   - Layout tab clicks → update `state.activeLayout` → re-run `renderTabs()`, `renderMockup()`, `renderBrowser()`
   - Hover on `[data-comp]` → update `state.activeComp` → `showTooltip()` + `syncBrowserHighlight()`
   - Browser item click → scroll component into view OR switch layout if component lives elsewhere
   - Search input → debounced → `state.searchQuery` → `renderBrowser()`
   - Toggle buttons (Outlines, Dummy content, Component browser) → flip boolean in state → re-render

### Wireframe Layout Functions

Each layout function in `layouts.js`/`layouts2.js` accepts `opts` and returns an HTML string. They use helper factories returned by `makeHelpers(opts.dummy)`:

- `lw(w, h, text?)` → wire line (placeholder or real text)
- `img(w, h)` → placeholder image box or `<img>` tag
- `fbtn(label, w, h)` → filled button (real or placeholder)
- `obtn(label, w, h)` → outlined button
- `bdg(text)` → badge/pill
- `navTxt(text, w)` → navigation text style

Layout functions produce white-outline boxes on transparent backgrounds. When `dummy === true`, helpers render actual text, images, and interactive buttons for realism. When `false`, they render empty boxes with CSS classes (`wl`, `wf-img-box`, etc.) for the classic wireframe look.

All hoverable elements must have `data-comp="component-id"` attribute matching a key in `COMPONENTS`.

### Tooltip & Hover Sync

`showTooltip(compId, triggerEl)` (in `render.js`):
- Reads component from `COMPONENTS[compId]`
- Populates tooltip template: name, category, also-known-as, description, variants (pill row), prompt tip (copy-ready phrase), frameworks (pill row)
- Calls `positionTooltip()` to place tooltip to the right of trigger (or left if near viewport edge)
- `syncBrowserHighlight()` adds `.active` class to matching browser item

### Checklist Layout

Special layout (last tab) renders a component checklist with localStorage persistence:
- `CHECKLIST_CATEGORIES` defines category headers with icons
- `CHECKLIST_ITEMS` array of checklist items with category, label, desc, tip
- Checked state stored in `localStorage.checklistChecked` as JSON string array of ids
- `renderChecklist()` runs when layout switches to `checklist`; builds category sections with progress counts
- Checkbox change → update localStorage → re-render to update progress bars

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
