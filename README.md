<div align="center">

# UI Anatomy

Learn UI component names by hovering a live wireframe mockup

[![Live][badge-site]][url-site]
[![HTML5][badge-html]][url-html]
[![CSS3][badge-css]][url-css]
[![JavaScript][badge-js]][url-js]
[![Claude Code][badge-claude]][url-claude]
[![License][badge-license]](LICENSE)

[badge-site]:    https://img.shields.io/badge/live_site-0063e5?style=for-the-badge&logo=googlechrome&logoColor=white
[badge-html]:    https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white
[badge-css]:     https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white
[badge-js]:      https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black
[badge-claude]:  https://img.shields.io/badge/Claude_Code-CC785C?style=for-the-badge&logo=anthropic&logoColor=white
[badge-license]: https://img.shields.io/badge/license-MIT-404040?style=for-the-badge

[url-site]:   https://uianatomy.neorgon.com/
[url-html]:   #
[url-css]:    #
[url-js]:     #
[url-claude]: https://claude.ai/code

</div>

---

## Overview

UI Anatomy is an interactive wireframe simulator where hovering any part of a white-outline website mockup reveals the component's name, alternate names, design variants, and a ready-to-use AI prompt tip. It helps developers, designers, and anyone working with AI tools build better prompts by teaching them the vocabulary of UI components.

The app ships many layout families (core site types plus industry templates such as SaaS, ecommerce, and game studio), each exposing a curated set of hover targets. A separate **Learning Hub** (`learning/`) adds checklists, anti-patterns, prompt recipes, and framework notes.

**Live:** uianatomy.neorgon.com

---

## Features

- **Wireframe explorer**: multiple layouts; hover any region to see its UI vocabulary, variants, and prompt tips
- **50+ component definitions**: name, alternate names, description, variants, and prompt tip per component (see JSON glossary export in the site footer)
- **Component browser**: searchable sidebar for the active layout; syncs with hover and scroll-to-target
- **Quiz mode**: recall practice over the live wireframe — "name the highlighted part" and "click the named part" questions, with score/streak persisted locally
- **AI prompt export**: one click turns the visible layout into a copy-ready prompt (components in page order, with descriptions) for AI design tools
- **Learning Hub**: production checklist (with local progress), common mistakes, AI prompt guide, patterns, design systems, accessibility notes, architecture matrix, and framework code snippets
- **Deep links**: `?layout=` and `?comp=` on the home app; `?tab=` on the learning hub for shareable views

---

## Running locally

ES modules require an HTTP server (not `file://`):

```bash
make serve
# or
python3 -m http.server 8820
```

Then open [http://localhost:8820](http://localhost:8820).

---

## Architecture

![Architecture](docs/architecture.svg)

```
ui-anatomy-site/
├── index.html              # Wireframe app shell + JSON-LD
├── learning/               # Learning Hub (tabs, checklist, prompts, …)
│   ├── index.html
│   ├── hosting-resources.html
│   ├── frameworks-decision-guide.html  # Renders frameworks-decision-guide.md
│   ├── frameworks-decision-guide.md
│   ├── css/style.css
│   └── js/ …
├── css/
│   └── style.css           # Wireframe app styles
├── js/
│   ├── app.js              # Entry: ?layout=, ?comp=, wiring
│   ├── state.js            # Active layout, hovered comp, toggles
│   ├── data.js             # Component map + layout → component ids + layouts list
│   ├── layouts*.js         # Wireframe HTML generators
│   ├── render.js           # Layout nav, mockup, browser, tooltip
│   ├── events.js           # Interactions + glossary export
│   └── utils.js            # debounce, escHtml, prefersReducedMotion
├── docs/
│   ├── architecture.mmd
│   └── architecture.svg
├── CNAME
├── Makefile
├── robots.txt
└── sitemap.xml
```

---

<div align="center">
<sub>Part of <a href="https://neorgon.com/">Neorgon</a></sub>
</div>
