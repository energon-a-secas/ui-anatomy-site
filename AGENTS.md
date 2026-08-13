# AGENTS.md

Guide for AI agents working in the UI Anatomy codebase.

---

## Project Overview

**UI Anatomy** is an interactive educational tool that teaches UI component vocabulary through hoverable wireframe mockups. Users hover over parts of multiple wireframe layouts to learn component names, variants, and AI prompt tips. **NEW**: Includes a Website Checklist tab for launch preparation.

- **Live site**: https://uianatomy.neorgon.com/
- **Tech stack**: Vanilla HTML, CSS, JavaScript (ES6 modules)
- **No build step**: Direct browser execution via HTTP server
- **Total codebase**: ~3,000 lines (JS + CSS + HTML)
- **Deployment**: GitHub Pages with custom domain

### Key Features
1. **21 wireframe layouts**: 8 basic types (Landing, Corporate, Startup, Portfolio, Blog, Component Library, Login, Link in Bio) plus 13 industry templates (SaaS, Fintech, Ecommerce, Gaming, Docs, …)
2. **50+ component definitions**: name, variants, descriptions, AI prompt tips
3. **Interactive hover**: hover mockup elements to learn component names
4. **Component browser**: searchable sidebar with all components
5. **Quiz mode**: recall practice over the live wireframe (name-the-part / find-the-part), score + streak persisted in localStorage
6. **AI prompt export**: generates a copy-ready prompt describing the visible layout, components in page order
7. **Website Checklist** (Learning Hub): 45+ items for SEO, technical setup, analytics, legal, performance, security, accessibility, social media
8. **Dummy mode**: toggle between wireframe lines and realistic content
9. **Show outlines**: reveal all component boundaries
10. **Hero backgrounds**: test different hero section styles

---

## Essential Commands

### Development

```bash
# Start local dev server (port 8820)
make serve
# or
python3 -m http.server 8820

# Stop server
make kill

# View available commands
make help
```

Then open http://localhost:8820

**Critical**: ES modules require an HTTP server. The `file://` protocol will not work.

### No Build/Test/Lint Commands

This project has **no build step, no test suite, no linter, and no package.json**. It runs pure vanilla JavaScript directly in the browser.

---

## Architecture

### File Structure

```
ui-anatomy-site/
├── index.html              # App shell (~130 lines)
├── css/
│   └── style.css           # All styles: Neorgon dark theme + wireframe elements
├── js/
│   ├── app.js              # Entry point, wires up render + events
│   ├── state.js            # Active layout, hovered comp, toggles (9 lines)
│   ├── data.js             # 40+ component definitions + checklist data
│   ├── layouts.js          # Landing, Corporate, Startup wireframe templates
│   ├── layouts2.js         # Portfolio, Blog, Components, Login, Checklist templates
│   ├── render.js           # Tabs, mockup, browser, tooltip, checklist rendering
│   ├── events.js           # Hover, click, search, toggles, checklist interactions
│   └── utils.js            # debounce, escHtml (15 lines)
├── docs/
│   ├── architecture.mmd    # Mermaid source
│   └── architecture.svg    # Generated diagram
├── Makefile                # make serve (port 8820), make kill
├── CNAME                   # uianatomy.neorgon.com
├── favicon.ico
├── energon-classic-logo.png
├── og-preview.jpg          # OG image (1200x630)
├── robots.txt
└── sitemap.xml
```

### Module Graph

```
app.js (entry)
  ├─> state.js (global state object)
  ├─> render.js
  │     ├─> layouts.js + layouts2.js (7 wireframe template functions)
  │     └─> data.js (COMPONENTS, CATEGORIES, LAYOUT_COMPONENTS, LAYOUTS)
  ├─> events.js
  │     ├─> render.js (circular import for updates)
  │     └─> utils.js (debounce)
  └─> utils.js
```

### Data Flow

1. **State**: Single global `state` object in `state.js` (not reactive, manually updated)
2. **Render**: Functions in `render.js` read state and rebuild DOM sections
3. **Events**: Event handlers in `events.js` update state and call render functions
4. **No framework**: Plain DOM manipulation, no virtual DOM, no reactivity

---

## Code Organization

### State Management (`state.js`)

```js
export const state = {
  activeLayout: 'landing',    // 'landing' | 'corporate' | 'startup' | 'portfolio' | 'blog' | 'components' | 'login'
  activeComp: null,           // id of currently hovered component (string | null)
  browserOpen: true,          // sidebar visibility
  outlinesOn: false,          // show all component boundaries
  searchQuery: '',            // component search filter
  dummyMode: false,           // show dummy text/images vs wireframe lines
  heroBg: 'solid',            // hero background style
};
```

- **No persistence**: State resets on page reload
- **Manual updates**: No setter functions, directly mutate properties
- **Re-render pattern**: Update state → call render function(s)

### Component Data (`data.js`)

**COMPONENTS** object: 40+ component definitions

```js
export const COMPONENTS = {
  'component-id': {
    name: 'Display Name',
    also: ['Alternate Name 1', 'Alternate Name 2'],
    desc: 'Short description of the component and its purpose.',
    variants: ['Variant 1', 'Variant 2', 'Variant 3'],
    tip: '"Example AI prompt showing how to request this component."',
    category: 'navigation' | 'layout' | 'content' | 'forms' | 'media' | 'feedback',
  },
};
```

**LAYOUT_COMPONENTS** object: Maps layout IDs to arrays of component IDs present in that layout

```js
export const LAYOUT_COMPONENTS = {
  landing: ['navbar', 'hero-section', 'feature-grid', ...],
  corporate: ['navbar', 'breadcrumb', ...],
  // ...
};
```

**LAYOUTS** array: Defines the tab labels

```js
export const LAYOUTS = [
  { id: 'landing', label: 'Landing' },
  { id: 'corporate', label: 'Corporate' },
  // ...
];
```

**HERO_BACKGROUNDS** array: Hero background style options

```js
export const HERO_BACKGROUNDS = [
  { id: 'solid', label: 'Solid', swatch: '#ffffff' },
  { id: 'gradient-purple', label: 'Purple Gradient', swatch: 'linear-gradient(...)' },
  // ...
];
```

**CHECKLIST_CATEGORIES** object: Website checklist categories (NEW)

```js
export const CHECKLIST_CATEGORIES = {
  seo: { label: 'SEO & Metadata', icon: '🔍' },
  technical: { label: 'Technical Setup', icon: '⚙️' },
  analytics: { label: 'Analytics & Tracking', icon: '📊' },
  legal: { label: 'Legal & Compliance', icon: '⚖️' },
  performance: { label: 'Performance', icon: '⚡' },
  security: { label: 'Security', icon: '🔒' },
  accessibility: { label: 'Accessibility', icon: '♿' },
  social: { label: 'Social Media', icon: '📱' },
};
```

**CHECKLIST_ITEMS** array: 45+ checklist items (NEW)

```js
export const CHECKLIST_ITEMS = [
  {
    id: 'favicon',
    category: 'technical',
    label: 'Favicon',
    desc: '32×32px icon shown in browser tabs',
    tip: 'Add <link rel="icon" type="image/x-icon" href="favicon.ico"> or use PNG...'
  },
  // ... 44 more items
];
```

### Wireframe Layouts (`layouts.js`, `layouts2.js`)

Each layout is a function that returns an HTML string:

```js
export function landingLayout(opts = {}) {
  const H = makeHelpers(opts.dummy);
  return `
    <div class="wf-navbar" data-comp="navbar">
      <!-- navbar HTML -->
    </div>
    <div class="wf-hero" data-comp="hero-section">
      <!-- hero HTML -->
    </div>
    <!-- ... -->
  `;
}
```

**Key patterns**:
- Every hoverable component has `data-comp="component-id"` attribute
- `makeHelpers(dummy)` returns helper functions for wireframe primitives (lines, images, buttons)
- Helper functions render either wireframe placeholders OR real dummy content based on `dummy` flag
- Shared elements: `sharedNavbar(H, opts)`, `sharedFooter(H)`, `featureGrid(H, count, data)`

**Wireframe Helpers** (in `makeHelpers`):
- `lw(width, height, text)` → wireframe line OR text span
- `img(w, h)` → dummy image OR wireframe box
- `fbtn(label, w, h)` → filled button
- `obtn(label, w, h)` → outline button
- `bdg(text)` → badge pill
- `navTxt(text, w)` → nav link text
- `iconSq(size)` → icon square
- `avatarEl(size)` → circular avatar

### Rendering (`render.js`)

**Main render functions** (exported, called by events):

- `renderTabs()` → Rebuild layout tabs from LAYOUTS array
- `renderMockup()` → Rebuild wireframe mockup using current activeLayout (also triggers renderChecklist if needed)
- `renderBrowser()` → Rebuild component browser sidebar (filtered by search)
- `renderHeroBgPicker()` → Rebuild hero background swatches
- `applyHeroBg()` → Add/remove CSS class on hero element based on `state.heroBg`

**Tooltip functions**:

- `showTooltip(compId, triggerEl)` → Show tooltip next to hovered component
- `hideTooltip()` → Hide tooltip
- `positionTooltip(tt, triggerEl)` → Calculate smart tooltip position (avoid viewport edges)

**Browser sync**:

- `syncBrowserHighlight(compId)` → Highlight matching item in component browser

**Checklist functions** (NEW):

- `renderChecklist()` → Render checklist items from CHECKLIST_ITEMS array
- `updateChecklistProgress()` → Update progress bar and category counters based on checked state
- Checked state persists in `localStorage` with key `'checklistChecked'`

### Event Handling (`events.js`)

**`initEvents()`** sets up all event listeners:

1. **Layout tabs**: Click → update `state.activeLayout`, re-render tabs/mockup/browser, hide tooltip
2. **Component hover**: Mouseover → add `hovered` class, show tooltip, sync browser highlight
3. **Mockup mouseleave**: Remove hover state, hide tooltip
4. **Browser item click**: Scroll to component in mockup, show tooltip
5. **Search input**: Debounced update of `state.searchQuery`, re-render browser
6. **Browser toggle**: Toggle `state.browserOpen`, show/hide sidebar, toggle button `active` class
7. **Outlines toggle**: Toggle `state.outlinesOn`, add/remove `show-outlines` class on mockup
8. **Dummy toggle**: Toggle `state.dummyMode`, re-render mockup
9. **Hero BG picker**: Click swatch → update `state.heroBg`, apply background
10. **Checklist checkbox** (NEW): Change → update localStorage, toggle visual state, update progress

**Hover state tracking**:
- Local `currentHovered` variable tracks currently hovered DOM element (not component ID)
- Prevents redundant tooltip updates when mouse moves within same component

---

## Code Conventions

### JavaScript Style

- **ES6 modules**: `import`/`export`, no CommonJS
- **No transpilation**: Modern syntax only (const/let, arrow functions, template literals, destructuring, optional chaining)
- **No semicolons**: Consistent ASI (Automatic Semicolon Insertion) style
- **String templates**: Prefer template literals for HTML generation
- **No JSX**: Plain string concatenation/interpolation
- **Arrow functions**: Preferred for callbacks and utilities
- **Object shorthand**: Use when possible (`{ state }` not `{ state: state }`)

### Naming Conventions

- **Files**: Lowercase with hyphens where needed (rare, mostly single words)
- **Functions**: camelCase (e.g., `renderMockup`, `initEvents`, `makeHelpers`)
- **Constants**: SCREAMING_SNAKE_CASE for exported data objects (e.g., `COMPONENTS`, `LAYOUTS`)
- **Variables**: camelCase (e.g., `currentHovered`, `activeLayout`)
- **CSS classes**: kebab-case (e.g., `wf-navbar`, `browser-item`, `layout-tab`)
- **Data attributes**: kebab-case (e.g., `data-comp`, `data-layout`, `data-comp-id`)

### HTML Generation

All HTML is generated via template literal strings:

```js
export function renderTabs() {
  const el = document.getElementById('layoutTabs');
  el.innerHTML = LAYOUTS.map(l => `
    <button class="layout-tab${state.activeLayout === l.id ? ' active' : ''}"
            role="tab" aria-selected="${state.activeLayout === l.id}"
            data-layout="${l.id}">${l.label}</button>
  `).join('');
}
```

**Patterns**:
- Use `.map().join('')` for lists
- Inline ternaries for conditional classes/attributes
- No escaping of user input (no user-generated content in this app)
- ARIA attributes for accessibility (role, aria-selected, aria-label)

### CSS Architecture

**Single file**: All styles in `css/style.css`

**Organization** (in order):
1. Reset & base
2. Skip link (accessibility)
3. Header
4. Controls bar (tabs + toggle buttons)
5. Layout structure (anatomy-body, mockup-area, comp-browser)
6. Component browser (sidebar)
7. Wireframe elements (wf-* classes)
8. Tooltip
9. Footer
10. Responsive (media queries at bottom)

**CSS Custom Properties** (defined in `:root`):

```css
--bg, --surface-1, --surface-2
--border-subtle, --border, --border-strong
--text-primary, --text-secondary, --text-muted
--accent, --accent-bright, --accent-dim
--font
--radius-sm, --radius, --radius-lg
--ease-snap, --dur
```

**Naming**:
- `.header-*` → Header/navbar components
- `.controls-*` → Controls bar elements
- `.layout-*` → Layout tab buttons
- `.ctrl-btn` → Toggle buttons
- `.anatomy-*` → Main body structure
- `.comp-*` → Component browser
- `.browser-*` → Browser sub-elements
- `.mockup-*` → Mockup area
- `.wf-*` → Wireframe elements (wf = wireframe)
- `.tt-*` → Tooltip elements (tt = tooltip)
- `.checklist-*` → Checklist tab elements (NEW)

**Wireframe classes**:
- `.wf-navbar`, `.wf-hero`, `.wf-footer` → Major sections
- `.wf-comp` → Individual hoverable component wrapper
- `.wl`, `.wl-8`, `.wl-12`, etc. → Wireframe lines (wl = wireframe line, number = height)
- `.wf-img-box` → Wireframe image placeholder
- `.wf-filled-btn`, `.wf-outline-btn` → Button styles
- `.wf-badge-pill`, `.wf-avatar`, `.wf-icon-sq` → UI primitives

**Checklist classes** (NEW):
- `.checklist-container` → Main wrapper
- `.checklist-category` → Category group
- `.checklist-item` → Individual checklist item
- `.checklist-checkbox` → Custom styled checkbox
- `.checklist-progress-bar` / `.checklist-progress-fill` → Progress indicator
- `.checklist-item.checked` → Checked state (opacity reduced, text struck-through)

**State classes**:
- `.active` → Active tab, active browser item, active toggle button
- `.hovered` → Hovered component in mockup
- `.hidden` → Hidden element (tooltip, browser sidebar when closed)
- `.show-outlines` → Mockup with visible component boundaries
- `.missing` → Browser item for component not in current layout (dimmed)
- `.checked` → Checked checklist item (NEW)

### Accessibility

- **Skip link**: Hidden "Skip to content" link (visible on focus)
- **ARIA attributes**: `role`, `aria-label`, `aria-selected`, `aria-labelledby`
- **Keyboard navigation**: Tab, Enter, Space work on interactive elements
- **Focus styles**: Visible focus indicators on buttons and links
- **Semantic HTML**: `<header>`, `<main>`, `<footer>`, `<aside>`, `<nav>` (implied by role)

### Performance Considerations

- **No virtual DOM**: Full innerHTML replacement on render (acceptable for ~40 components)
- **Debounced search**: 150ms debounce on search input
- **Event delegation**: Some events use delegation (layoutTabs click, browserList click)
- **Minimal reflows**: Tooltip positioning is the main reflow trigger

---

## Component System

### Adding a New Component

1. **Define in `data.js`**:

```js
export const COMPONENTS = {
  // ... existing components
  'my-component': {
    name: 'My Component Name',
    also: ['Alternate Name'],
    desc: 'Short description.',
    variants: ['Variant 1', 'Variant 2'],
    tip: '"Example prompt."',
    category: 'content', // or navigation, layout, forms, media, feedback
  },
};
```

2. **Add to layout** in appropriate layout function (`layouts.js` or `layouts2.js`):

```js
<div class="wf-comp" data-comp="my-component">
  <!-- component HTML using helpers -->
</div>
```

3. **Register in `LAYOUT_COMPONENTS`** (in `data.js`):

```js
export const LAYOUT_COMPONENTS = {
  landing: ['navbar', 'hero-section', 'my-component', ...],
  // ...
};
```

4. **Style in `style.css`** (if needed):

```css
.wf-my-component {
  /* styles */
}
```

### Adding a New Layout

1. **Create layout function** in `layouts.js` or `layouts2.js`:

```js
export function myLayout(opts = {}) {
  const H = makeHelpers(opts.dummy);
  return `
    ${sharedNavbar(H)}
    <!-- layout HTML -->
    ${sharedFooter(H)}
  `;
}
```

2. **Register in `LAYOUTS`** array (`data.js`):

```js
export const LAYOUTS = [
  // ... existing
  { id: 'mylayout', label: 'My Layout' },
];
```

3. **Add component list** to `LAYOUT_COMPONENTS`:

```js
export const LAYOUT_COMPONENTS = {
  // ... existing
  mylayout: ['navbar', 'hero-section', 'footer', ...],
};
```

4. **Import and register** in `render.js`:

```js
import { myLayout } from './layouts2.js'; // or layouts.js

const LAYOUT_FNS = {
  // ... existing
  mylayout: myLayout,
};
```

---

## Website Checklist Feature

### Overview

The **Website Checklist** tab provides an interactive checklist of 45+ essential items for website launch. Designed for non-technical users to understand what's needed beyond just UI components.

### Data Structure

**Categories** (`CHECKLIST_CATEGORIES`):
- 🔍 SEO & Metadata (10 items)
- ⚙️ Technical Setup (8 items)
- 📊 Analytics & Tracking (3 items)
- ⚖️ Legal & Compliance (4 items)
- ⚡ Performance (5 items)
- 🔒 Security (4 items)
- ♿ Accessibility (5 items)
- 📱 Social Media (3 items)

**Item Structure**:
```js
{
  id: 'favicon',                    // Unique ID for localStorage persistence
  category: 'technical',            // Category key
  label: 'Favicon',                 // Short label
  desc: '32×32px icon shown in browser tabs',  // One-line description
  tip: 'Add <link rel="icon"...>...'           // Detailed implementation tip
}
```

### State Persistence

- **Storage**: `localStorage` with key `'checklistChecked'`
- **Format**: JSON array of checked item IDs: `["favicon", "robots-txt", "ssl", ...]`
- **On page load**: `renderChecklist()` reads from localStorage and pre-checks items
- **On checkbox change**: Event handler updates localStorage immediately

### UI Features

1. **Progress Bar**: Shows overall completion percentage
2. **Category Counters**: Each category shows "X / Y" checked items
3. **Visual States**:
   - Unchecked: Full opacity, no strikethrough
   - Checked: 60% opacity, strikethrough on label
4. **Custom Checkbox**: CSS-only, checkmark (✓) appears when checked
5. **Responsive**: Stacks on mobile, adjusts padding

### Adding New Checklist Items

1. **Add to `CHECKLIST_ITEMS`** in `data.js`:

```js
{
  id: 'my-item',
  category: 'security',  // Must match CHECKLIST_CATEGORIES key
  label: 'My Security Item',
  desc: 'Short description under 60 chars',
  tip: 'Detailed implementation tip. Include code examples if relevant.'
}
```

2. **No other changes needed**: rendering is automatic

### Adding New Categories

1. **Add to `CHECKLIST_CATEGORIES`**:
```js
mycategory: { label: 'My Category', icon: '🎯' }
```

2. **Add items** with `category: 'mycategory'`

3. **Update CSS** (optional) if you want category-specific styling

### Styling Notes

- Checkbox hover: Border changes to accent color
- Checkbox checked: Background = accent gradient, white checkmark
- Item hover: Background lightens
- Tip section: Left border accent color, subtle background
- Progress bar: Animated gradient fill (accent → accent-bright)

---

## Design System

### Neorgon Dark Theme

- **Background**: `#040714` (near-black blue)
- **Accent**: `#d946ef` (magenta/purple)
- **Text**: `#f9f9f9` (near-white), `#cacaca` (secondary), muted with alpha
- **Surface levels**: Subtle white overlays (3%, 6% opacity)
- **Borders**: White with varying opacity (7%, 10%, 22%)

### Wireframe Visual Style

- **Lines**: White with 15% opacity, rounded corners
- **Images**: Dashed border boxes with gradient patterns
- **Buttons**: Filled (white bg) or outline (border only)
- **Hover state**: Magenta glow, scale transform
- **Active component**: Magenta border, box-shadow glow

### Layout Structure

```
┌─────────────────────────────────────────┐
│ Header (sticky, gradient background)   │
├─────────────────────────────────────────┤
│ Controls (tabs + toggles)              │
├──────────────┬──────────────────────────┤
│              │                          │
│  Component   │   Mockup Area            │
│  Browser     │   (wireframe)            │
│  (sidebar)   │                          │
│              │                          │
│              │                          │
├──────────────┴──────────────────────────┤
│ Footer                                  │
└─────────────────────────────────────────┘
```

**Floating Tooltip**: Positioned next to hovered component (smart viewport edge detection)

---

## Common Tasks

### Testing Changes Locally

1. Start server: `make serve`
2. Open http://localhost:8820
3. Make changes to JS/CSS/HTML
4. Hard refresh browser (Cmd+Shift+R / Ctrl+Shift+R)

**No hot reload**: Manual refresh required after changes.

### Modifying Component Data

Edit `js/data.js` → `COMPONENTS` object. Changes appear on next page load.

### Changing Wireframe Appearance

1. **Wireframe lines**: Edit helper functions in `layouts.js` → `makeHelpers()`
2. **Dummy content**: Edit conditional branches in helper functions (when `dummy === true`)
3. **Component structure**: Edit layout functions in `layouts.js` / `layouts2.js`
4. **Styles**: Edit `css/style.css` (search for `.wf-` classes)

### Updating Styles

Edit `css/style.css`. Use existing CSS custom properties where possible.

**Dark theme**: All colors should work on dark background. Use white with opacity for borders/lines.

### Debugging

1. **Console logs**: Add `console.log()` statements (no linter to complain)
2. **Breakpoints**: Use browser DevTools debugger
3. **State inspection**: `console.log(state)` in events.js
4. **Component data**: `console.log(COMPONENTS['component-id'])` in render.js

---

## Deployment

### GitHub Pages

- **Branch**: `main` (root directory)
- **Custom domain**: Configured via `CNAME` file (uianatomy.neorgon.com)
- **No build step**: Files served directly as-is

### Updating Live Site

```bash
git add .
git commit -m "Description"
git push origin main
```

Changes go live in ~1 minute (GitHub Pages propagation).

### SEO & Metadata

- **Meta tags**: In `index.html` `<head>` (title, description, OG tags, Twitter card)
- **Structured data**: JSON-LD schema (WebApplication) in `<head>`
- **Sitemap**: `sitemap.xml` (manually maintained)
- **Robots**: `robots.txt` (allow all)
- **Canonical**: `<link rel="canonical">`

---

## Important Gotchas

### 1. ES Module Imports Must Be Explicit

```js
// ❌ Wrong
import { renderTabs } from './render';

// ✅ Correct
import { renderTabs } from './render.js';
```

File extensions are **required** in ES module imports.

### 2. Circular Import: events.js ↔ render.js

`events.js` imports from `render.js`, and `render.js` imports from `events.js` (indirectly via state). This works because imports are references, but be careful when refactoring.

### 3. No Escaping of HTML

The `escHtml` utility exists but is **not used** in current code. All HTML is hardcoded or comes from trusted data (no user input). If you add user-generated content, **escape it** using `escHtml()`.

### 4. State Is Not Reactive

Changing `state.activeLayout` **does not** automatically re-render. You must call render functions manually:

```js
// ❌ Wrong
state.activeLayout = 'corporate';

// ✅ Correct
state.activeLayout = 'corporate';
renderTabs();
renderMockup();
renderBrowser();
```

### 5. Tooltip Positioning Edge Cases

Tooltip positioning logic (`positionTooltip`) assumes:
- Tooltip width is 300px (hardcoded)
- Tooltip height is read from `offsetHeight` (may be 0 on first render)
- Viewport edges have 8px buffer

If tooltip appears in wrong position, check these assumptions.

### 6. Component IDs Must Match

The same component ID must be used in:
1. `COMPONENTS` object keys
2. `data-comp` attributes in layout HTML
3. `LAYOUT_COMPONENTS` arrays

Mismatches will break hover → tooltip → browser sync.

### 7. Layout Function Names

Layout functions must be registered in `render.js` → `LAYOUT_FNS` object with keys matching `LAYOUTS[].id` in `data.js`.

```js
// data.js
{ id: 'landing', label: 'Landing' }

// render.js
const LAYOUT_FNS = {
  landing: landingLayout,  // ← must match
};
```

### 8. Dummy Mode Helpers

Wireframe helpers (`makeHelpers`) must handle **both** dummy mode and wireframe mode. If you add a new helper, support both cases:

```js
const myHelper = (opts) => dummy
  ? `<div>Real content</div>`
  : `<div class="wf-placeholder"></div>`;
```

### 9. CSS Specificity

Wireframe components use many contextual selectors (e.g., `.wf-navbar .wf-comp`). When adding styles, check existing selectors to avoid unintended overrides.

### 10. Mobile Responsiveness

Responsive breakpoints are at bottom of `style.css`. Most layouts work on mobile, but some complex wireframes may need adjustments. Test at 375px width minimum.

---

## Browser Support

**Target**: Modern evergreen browsers (Chrome, Firefox, Safari, Edge)

**Required features**:
- ES6 modules
- CSS custom properties
- Flexbox
- Optional chaining (`?.`)
- Template literals
- Arrow functions

**No polyfills or transpilation**. If a browser doesn't support ES6 modules natively, the app won't load.

---

## Related Files

- **Architecture diagram**: `docs/architecture.svg` (generated from `docs/architecture.mmd`)
- **OG image**: `og-preview.jpg` (1200x630px, used in social media previews)
- **Logo**: `energon-classic-logo.png` (header logo)
- **Favicon**: `favicon.ico`
- **Dummy image**: `dummy-image.png` (used in dummy mode)

---

## Future Considerations

### If Adding Tests

- Use **Playwright** or **Puppeteer** for E2E tests (hover interactions are hard to unit test)
- Test: layout switching, hover → tooltip, browser item click → scroll, search filtering
- Snapshot test rendered HTML if you want to catch regressions

### If Adding Build Step

- **Why**: Minification, bundling, TypeScript, CSS preprocessing
- **Tools**: Vite (fastest), esbuild, Rollup
- **Keep it simple**: This project's simplicity is a feature, not a bug

### If Adding State Persistence

- Use **localStorage** to save `state.activeLayout`, `state.browserOpen`, etc.
- Load on init in `app.js`, save on state changes in `events.js`

### If Making Component Data Editable

- Move `COMPONENTS` to a JSON file
- Add admin UI or CMS integration
- Validate schema on load (e.g., with Zod)

---

## Questions & Debugging Tips

### Why is my component not appearing?

1. Check `data-comp="component-id"` attribute is present in layout HTML
2. Check component ID is in `LAYOUT_COMPONENTS[layoutId]` array
3. Check layout is being rendered (try `console.log` in layout function)

### Why is tooltip not showing?

1. Check component ID exists in `COMPONENTS` object
2. Check `data-comp` attribute matches component ID exactly
3. Check browser console for errors
4. Check tooltip element exists in HTML (`id="tooltip"`)

### Why is browser item not clickable?

1. Check item has `data-comp-id` attribute
2. Check component is present in current layout (not `.missing` class)
3. Check corresponding element exists in mockup with matching `data-comp` attribute

### Why is search not working?

1. Check debounce is working (150ms delay)
2. Check `state.searchQuery` is being updated
3. Check search logic in `renderBrowser()` (matches name or also array)

### How do I change the color scheme?

Edit CSS custom properties in `:root` in `style.css`. Change `--accent` for accent color, `--bg` for background, etc.

### How do I add a new category?

1. Add to `CATEGORIES` object in `data.js`
2. Use category key in component definitions
3. Styles for category labels are in `.browser-category-label` (no per-category styling)

---

## Philosophy & Principles

1. **Simplicity over complexity**: No framework, no build step, no dependencies (except Python for dev server)
2. **Vanilla is fine**: Modern JavaScript is powerful enough for small projects
3. **Performance is good enough**: ~40 components is not a scaling problem
4. **Accessibility matters**: ARIA attributes, semantic HTML, keyboard navigation
5. **Dark theme by default**: Neorgon brand style, easy on the eyes
6. **Educational focus**: Code should be readable by learners and AI tools
7. **Responsive but desktop-first**: Primary use case is desktop designers/developers
8. **Fast iteration**: No compile step means instant feedback

---

## Summary for Quick Reference

| Task | Command / Location |
|------|---------------------|
| Start dev server | `make serve` (port 8820) |
| Stop server | `make kill` |
| Add component | Edit `data.js` → `COMPONENTS` |
| Add layout | Edit `layouts.js` or `layouts2.js`, register in `render.js` |
| Edit styles | Edit `css/style.css` |
| Edit HTML structure | Edit `index.html` |
| Change state | Mutate `state` object in `state.js`, call render functions |
| Debug | Browser DevTools, console.log |
| Deploy | `git push origin main` |

**Entry points**:
- App initialization: `js/app.js` → `init()`
- State: `js/state.js` → `state` object
- Data: `js/data.js` → `COMPONENTS`, `LAYOUTS`, etc.
- Rendering: `js/render.js` → `render*()` functions
- Events: `js/events.js` → `initEvents()`

**Key patterns**:
- Update state → call render function (no reactivity)
- `data-comp="component-id"` for hoverable components
- Template literal strings for HTML generation
- Event delegation where possible
- CSS custom properties for theming

---

*Last updated: Based on codebase snapshot with git commit e316b02 (OG Image)*
