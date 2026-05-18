# Sevkaan Storefront — New Sections Design

**Date:** 2026-05-19
**Branch:** `feature/storefront-sections`
**Scope:** `index.html` only (root product listing page)
**Status:** Approved by user, ready for implementation plan

## Goal

Add 4 new commerce surfaces to the root storefront page:

1. Bestseller carousel
2. "By budget" tiles
3. Bundle hero banner
4. Floating "Best deals" countdown overlay (the FUEL ON GAMES widget)

No backend, no build system. `index.html` is a single static file using Tailwind CDN
plus a custom `<style>` block and a small inline `<script>`. All new work extends those
same in-file mechanisms. Theme, fonts, and existing components stay unchanged.

## Constraints & Context

- File: `/Users/tejasdeck/influencer-portal/index.html`
- Theme: dark `#0b0b0f`, headings use `.grot` (Bricolage Grotesque), accent `sk-grad`
  (orange→red gradient), giftcards use the `.gc` component (routes to `card.html` on click
  via existing delegated click handler).
- Page wrapper: `<main class="max-w-[1400px] mx-auto px-6 py-12 space-y-20">`.
- No layout reflow: page stays single-column. The deals widget floats over content.
- No tests / no framework. Verification = open in DIA browser, visual check.
- New CSS appended to the existing `<style>` block. New JS appended as / within a
  `<script>` block before `</body>` (existing script wires `.gc` and `.bento-tile`).

## Section Order (inside `<main>`)

```
Hero                  (existing — unchanged)
Bestseller rail       NEW
Sevkaan picks         (existing — unchanged)
By budget             NEW
Full catalog          (existing — unchanged)
Bundle banner         NEW   (id="bundles" anchor target)
Catch up / clips      (existing — unchanged)
```

Plus the **Best deals overlay**: a `position:fixed` element rendered once, outside
`<main>` flow (sibling of `<header>`/`<main>`), page-global.

Rationale: bestseller leads commerce right after the hero; budget bridges
picks→catalog (shop by price before the big grid); bundle upsells after the catalog;
clips remain the closing section.

## Component Specs

### 1. Bestseller rail

- Reuses the existing `.gc` card component and the existing horizontal-scroll +
  `‹ ›` arrow pattern from the "Sevkaan picks" section. No autoplay.
- Section header: `<h2 class="grot">Bestsellers <span>· top 6</span></h2>` matching the
  picks section's header markup, with the same two round arrow buttons.
- 6 curated brand cards (Steam, Riot, PlayStation, Xbox, Roblox, Fortnite — reusing the
  existing `.gc-*` brand classes and logos already present in the catalog).
- Each card gets a rank badge `#1`…`#6`. Reuse `.disc` badge styling but a new modifier
  `.rank` that pins it **top-left** (existing `.disc` is top-right) so it does not collide
  with a discount badge.
- Click → `card.html` (covered by the existing `document.querySelectorAll('.gc')` handler;
  no new wiring needed).
- Arrow buttons: a small JS scroll handler (scroll the rail container by ~one card width).
  The existing picks arrows are currently non-functional; this spec adds a reusable
  `scrollRail(btn, dir)` helper and wires the new bestseller arrows. (Wiring the existing
  picks arrows is optional and out of scope unless trivial in the same helper.)

### 2. By budget

- New `.budget-tile` component (NOT `.gc`). Responsive grid:
  `grid grid-cols-2 lg:grid-cols-4 gap-5`.
- 4 tiles: `$100`, `$200`, `$500`, `$1000`.
- Each tile: large amount in `.grot`, sub-label "Best picks under $X", a right arrow
  glyph, subtle gradient border, hover lift consistent with `.gc:hover`.
- Click target: placeholder anchor href `#budget-100`, `#budget-200`, `#budget-500`,
  `#budget-1000`. No real filtering logic — these are landing stubs (confirmed with user).
- Section header matches the catalog/picks header style ("Shop by budget").

### 3. Bundle banner

- Single full-width attention banner. `id="bundles"` (scroll target for the overlay CTA).
- Bold treatment using the existing `sk-grad` orange→red gradient + `grain` texture
  (both already defined in `<style>`).
- Content (one headline bundle):
  - Name: **"Triple Threat"**
  - Includes: Steam $10 + Valorant $25 + PSN $25
  - Pricing: cumulative `~~$60~~` → **`$40`**, label "Save $20 · 33% off"
  - 3 mini brand chips (small inline representations of the 3 included cards)
  - One CTA button: `Grab the bundle →` linking to `card.html` (placeholder).
- No carousel / no multiple bundles (confirmed: one hero banner only).

### 4. Best deals overlay (FUEL ON GAMES widget)

- Single element, `position:fixed`, pinned to the **right edge**, vertically centered
  (`top:50%; transform:translateY(-50%)`). High `z-index` (above content, below nothing
  critical; header is `z-50`, use `z-40` so it never covers the sticky header).
- **No layout reflow** — floats over page content, always in viewport while scrolling.
- Visual (matches the reference screenshot):
  - Purple gradient background, ember/spark accent (CSS radial dots / pseudo-element,
    no external image required).
  - Gold title **"FUEL ON GAMES"**.
  - **"ENDS IN:"** label + live countdown with 4 cells: `DAYS HRS MIN SEC`,
    each cell a boxed number with a unit label, mono font.
  - Yellow CTA button **"See the deals"**.
  - Close **✕** at top-right.
- Countdown:
  - Target = a JS `const DEALS_END` set to `Date.now() + ~36h` at page load
    (a fixed offset constant; no persistence).
  - `setInterval` 1s updates the 4 cells.
  - On reaching zero: freeze all cells at `00`, swap title to "Deals live now",
    keep the CTA enabled.
- Collapse behavior (user-selected: collapse to reopen tab):
  - ✕ hides the card and shows a thin vertical tab `⚡ DEALS` pinned to the right edge
    (`writing-mode: vertical-rl`).
  - Clicking the tab re-expands the card and hides the tab.
  - State held in a JS variable only (resets on reload — no localStorage; YAGNI).
- CTA action: smooth-scroll to `#bundles` (the bundle banner). `html { scroll-behavior:
  smooth }` is already set globally.
- Accessibility: ✕ and tab are real `<button>`s with `aria-label`s; the countdown region
  has `aria-live="off"` (decorative, avoid screen-reader spam).
- Mobile: below `sm`, reduce width and keep it pinned; acceptable to overlap content
  (matches reference behavior). No separate mobile redesign in scope.

## Files Changed

- `index.html` — add 3 in-`<main>` sections, 1 floating overlay element, supporting CSS
  in the existing `<style>` block, supporting JS in a `<script>` block before `</body>`.

No other files. `card.html`, `index-grouped.html`, `crazy8s.html` untouched.

## CSS Additions (new classes, appended to existing `<style>`)

- `.disc.rank` — rank badge variant, pinned top-left.
- `.budget-tile`, `.budget-tile:hover` — budget grid tile.
- `.bundle-banner` and inner helpers (chips, struck price).
- `.deals-overlay`, `.deals-overlay .x`, `.deals-overlay .cd`, `.deals-overlay .cd b`,
  `.deals-overlay .cta`, `.deals-tab`, `.ember` (spark accent), plus a `@keyframes`
  for a subtle ember drift (optional, low cost).

No existing class is modified or removed.

## JS Additions (one `<script>` block, additive)

- `scrollRail(dir)` helper + wiring for the bestseller arrows.
- Countdown ticker (`DEALS_END` const, `setInterval`, zero-state handling).
- Overlay collapse/expand toggle (✕ → tab, tab → card).
- CTA → smooth scroll to `#bundles`.

The existing `.gc` and `.bento-tile` click handlers are left intact and continue to
cover the new bestseller cards.

## Out of Scope (YAGNI)

- Real budget filtering / a budget landing page.
- Multiple bundles / bundle carousel.
- Persisting overlay-dismissed state across reloads (localStorage).
- Wiring the pre-existing (currently dead) "Sevkaan picks" arrows, unless free via the
  shared `scrollRail` helper.
- Any change to `card.html` or other pages.
- Backend, cart logic, real pricing.

## Verification

- Open `index.html` in DIA (`open -a Dia index.html`) — user preference is DIA only,
  never Chrome/Playwright.
- Visual checks:
  - All 4 new surfaces render in the specified order, theme-consistent.
  - Bestseller arrows scroll the rail; cards route to `card.html`.
  - Budget tiles render 4-up and have working placeholder anchors.
  - Bundle banner shows struck $60 → $40 and CTA scrolls to itself / bundles anchor.
  - Overlay floats fixed right, countdown ticks every second, ✕ collapses to tab,
    tab re-expands, CTA scrolls to the bundle banner.
  - No layout reflow of existing sections; hero and existing sections visually unchanged.
- Run `superpowers:requesting-code-review` on the diff before calling done (per user
  global instructions).
