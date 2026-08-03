# UI Anatomy — Rendering Internals (Level 2 reference)

> Moved verbatim from `CLAUDE.md` during a progressive-disclosure pass. Read this before
> changing the rendering pipeline, layout generators (`layouts.js`/`layouts2.js`), tooltip/hover
> sync (`render.js`), or the checklist layout.

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
