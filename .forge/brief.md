# Brief: ui-anatomy-site: add quiz mode + layout prompt export; plan standalone palette/patterns site

Started 2026-08-09 19:37. Maintained by the `task` skill; read by `debrief` and `writeup`.

## Problem

The site taught UI vocabulary by exposure only, hover, read, hope it sticks. Nothing tested
recall, and the stated purpose ("know what to call things when prompting AI design tools") had
no payoff artifact: per-component tips existed, but no way to turn a whole layout into a prompt.
Separately: picking colors for new projects meant "pick an unused Tailwind accent". No palette,
no coherence check, no story for why the colors work.

## Approach

Quiz mode: a controls-bar toggle swaps the component browser for a quiz panel over the *live*
wireframe: alternating "name the highlighted part" (4 choices, same-category distractors) with
"click the named part". Tooltips are suppressed while active (they leak answers); score/streak
persist in localStorage following the checklist precedent. On mobile the panel becomes a fixed
bottom sheet (the sidebar is display:none ≤768px). Prompt export: a modal showing a prose prompt
for the current layout: components extracted from the rendered DOM in page order, first-sentence
descriptions, hero-bg style when non-solid, editable textarea + clipboard copy. Both features
derive from the DOM rather than LAYOUT_COMPONENTS so they cannot drift from what's on screen.
Palette tool: plan only, for a standalone site executed later via /new-project.

## Rejected

- Quiz as a separate `learning/`-style page: loses the spatial association with the wireframe
  the user was just exploring; the recall question must point at the same pixels they studied.
- Prompt generated from `LAYOUT_COMPONENTS` data: can drift from the actual render; the DOM is
  the truth.
- Palette tool inside ui-anatomy (dilutes single-purpose identity) or as skill-only (no visual
  preview: the whole point is *seeing* colors together). Standalone site won: one job per site
  is the fleet pattern, and the wireframe preview becomes its differentiator.

## Decisions

<!-- Appended by: brief.sh note "<what you learned>" -->

- `2026-08-09 19:58` Rejected: quiz as separate learning/ page (loses spatial link to wireframe); prompt from LAYOUT_COMPONENTS data (drifts from render). Both quiz pool and prompt read the rendered DOM instead.

- `2026-08-09 19:58` Found pre-existing, NOT fixed: tests/test-layouts.html 404s (imports ./js/layouts2.js relative to tests/, and stale names fortunelayout/saasLayout vs layouts4.js exports); dummy-image.png 404 referenced by hero-bg picker swatch; 2 console errors on load are these, unrelated to this work.

- `2026-08-09 19:58` Palette tool decision: standalone site (Palette Forge, paletteforge.neorgon.com, UI Lab hub section, port 8860), plan at docs/plans/2026-08-09-palette-forge-site.md, queued as prompt #12. /color keeps theory role; site is the instrument.

## Measured

Verified in headless Chromium (Playwright) against localhost:8820, 2026-08-09:
- name-mode correct answer → Score 1/1 · Streak 1; wrong answer → 1/2, streak reset, correct
  choice revealed green
- find-mode wrong click → "That's the Footer: the Subheadline is now highlighted", both
  elements marked; correct click → flash-green, Score 2/4
- stats survive reload (localStorage `quizStats` = 4 answered / 2 correct)
- layout switch mid-quiz regenerates the question, zero stale highlight marks
- end-quiz restores the component browser, re-enables its toggle, tooltips work again
- prompt modal on SaaS: 20 component lines in page order; hero-bg line ("Night") appears only
  when a non-solid hero background is selected; copy button shows "Copied ✓"; Escape closes
- mobile 390px: quiz panel computed style position:fixed, bottom:0 (bottom sheet)

## Open

- `tests/test-layouts.html` is broken pre-existing (404s on `tests/js/layouts2.js`; stale import
  names). Not fixed: flagged for the user.
- `dummy-image.png` 404 referenced by the hero-bg "Photo" swatch. Pre-existing, not fixed.
- Palette Forge is a plan, not a site: `docs/plans/2026-08-09-palette-forge-site.md`, prompt
  queue #12. Open questions in the plan: final name, fleet-JSON generator location, PNG tiles.
- User deferred (chose not to build, may want later): semantic HTML/ARIA layer per component,
  more layouts (dashboard / mobile app / settings), visual variant previews, compare view.

_Closed 2026-08-09 19:59._
