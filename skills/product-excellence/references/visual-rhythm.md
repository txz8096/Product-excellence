# Pass 1 — Visual Rhythm & Responsiveness

The question this pass answers: **does the page hold its composure at every size,
and does the spacing feel like one system rather than many accidents?**

## Procedure

Work through the viewport matrix in this order — desktop first (that's where rhythm
is designed), mobile last (that's where it breaks).

### At each viewport (1440×900 → 1280×720 → 375×812)

1. **Screenshot for gestalt.** Look for: sections that feel cramped or stranded,
   uneven gutters, elements that visually drift off the grid, images at awkward
   sizes.
2. **Overflow check (programmatic).** Run in `preview_eval`:
   ```js
   (() => {
     const de = document.documentElement;
     const offenders = [...document.querySelectorAll('*')].filter(el =>
       el.scrollWidth > de.clientWidth + 1 &&
       getComputedStyle(el).overflowX !== 'hidden'
     ).slice(0, 10).map(el => el.className || el.tagName);
     return { pageScrollWidth: de.scrollWidth, viewport: de.clientWidth, offenders };
   })()
   ```
   `pageScrollWidth > viewport` at 375px is an automatic P0.
3. **Text integrity.** Headlines: no single-word orphan lines (fix:
   `text-wrap: balance`). Body: no mid-word breaks in CJK-free text unless
   `hyphens` is intentional; no ellipsis truncation hiding meaningful content
   (`text-overflow: ellipsis` on a stat line that users need to read is a finding).
   Long unbroken strings (URLs, file paths, code) need `word-break` or `overflow-wrap`.
   If a line overflows despite `white-space: normal`, check for the inverse
   problem: generated markup (JSX `.map()`, template loops) often emits adjacent
   inline elements with **no whitespace between them**, leaving the line zero
   break opportunities — it can never wrap, only clip. The fix is explicit space
   text nodes between segments, not CSS.
4. **Fixed-height layouts.** If `body` or the hero locks height, test 1440×1080.
   A layout tuned at ~750px height that pins content to the bottom will strand or
   clip on tall screens.

### Spacing-scale audit (once, at desktop)

Inspect the repeated components (cards, list rows, section headers) with
`preview_inspect` and record padding / gap / margin values. You are looking for:

- **A scale, not a scatter.** Values should cluster on a step scale (e.g. 8/16/24/
  40). Three cards with padding 24, 22, and 25 is a rhythm finding even though no
  single card looks wrong.
- **Consistent section rhythm.** Vertical space between sibling sections should be
  equal or intentionally ranked. Measure the actual gaps; don't trust the CSS to
  tell you (margins collapse, absolute children escape).
- **Label-to-content proximity.** A caption should sit closer to its image than to
  the next section (Gestalt proximity). Recurrent user note: "there should be more
  spacing" between text and unrelated elements above/below.

### Typography rhythm

For each text role (h1, h2, body, meta/label, code) record computed font-family,
size, line-height, letter-spacing. Findings:

- Same role, different values across the page or across sibling pages → consistency
  finding, cite both values.
- Body line-height outside 1.5–1.6, body measure above ~720px, meta text under 11px.
- `clamp()` sizing: check both ends of the clamp actually render well (evaluate at
  375px and 1440px, not just the middle).

### Responsive behavior between breakpoints

Don't only test the three matrix sizes — drag through the awkward middle (~700–900px)
where two-column layouts decide whether to wrap. Watch for: two-column grids where
one column orphans, nav that wraps to two lines, images that jump size classes.

### Layout shift & pre-JS render

A page that moves under the reader is a rhythm bug, even when every static
screenshot looks right.

- **Measure CLS during load** in `preview_eval`:
  ```js
  (() => new Promise(res => {
    let cls = 0, worst = null;
    new PerformanceObserver(list => {
      for (const e of list.getEntries()) if (!e.hadRecentInput) {
        cls += e.value;
        const src = e.sources && e.sources[0] && e.sources[0].node;
        if (src) worst = (src.className || src.tagName).toString().slice(0, 40);
      }
    }).observe({ type: 'layout-shift', buffered: true });
    setTimeout(() => res({ cls: +cls.toFixed(3), worstShifter: worst }), 3000);
  }))()
  ```
  (Reload first — buffered entries only cover the current page load.)
  Above ~0.1 is a finding; name the shifting element.
- **Check the usual causes directly:** images/iframes/video without intrinsic
  `width`/`height` (or CSS `aspect-ratio`), web fonts without
  `font-display: swap`, content injected above existing content after load.
- **Blank-before-JS:** if the app is client-rendered, note what exists before
  the bundle executes (view the served HTML body). A pure white page until JS
  arrives is a finding whose priority scales with bundle size and audience
  network quality.

### Content stress test

The happy-path copy is the best case. Stress the repeated components with
hostile-but-legal content via `preview_eval` (swap text into the live DOM,
screenshot, then reload to reset):

- Longest realistic string per slot: a 40-character name in a card title, a
  three-line label in a chip, an unbroken URL in body text.
- List extremes: zero items (is there an empty state, or a stranded heading?)
  and 3× the designed count (does the layout scroll, wrap, or explode?).
- Widest numbers where numbers render ("$1,234,567.89" in a slot designed
  around "$5"; 4-digit counts in badges).
- If the product will be translated, +30% text length is the German test.

A component that survives gets a "what's working" mention — clamping behavior
is worth protecting from future "simplification."

### Cross-page consistency check

Open one established sibling page side by side. Compare: header/nav treatment, card
anatomy (padding, tag row, hover state), type scale, accent-color usage, footer.
Every divergence is a finding unless the design source of truth sanctions it.

### Language & pattern consistency

Two consistency classes hide from visual comparison and need their own sweep:

- **Terminology:** the same thing carries the same name everywhere — no "Resume"
  on one surface and "CV" on another, no "Sign in" vs "Log in", no feature
  renamed mid-product. Collect the nouns and verbs the UI uses for its key
  objects and actions (buttons, nav, headings, empty states) and diff them.
  Same rule for micro-conventions: external-link arrows (↗ vs →), date formats,
  capitalization of the same label.
- **Interaction patterns:** the same gesture means the same thing everywhere.
  If one external link opens a new tab, all should; if one card is fully
  clickable, sibling cards shouldn't require hitting a small link inside; if a
  list numbers its items in one place, the same list elsewhere shouldn't drop
  the numbers. Inventory the repeated patterns (links, cards, chips, dialogs)
  and check each behaves identically at every occurrence.

P1 when the inconsistency can mislead (a renamed action reads as a different
action); P2 when it just reads as sloppiness.

## Evidence format

Every finding cites: viewport, selector, measured value, expected value.
"`.topic__name` clips at 375px (scrollWidth 402 > 375)" — good.
"Mobile looks cramped" — not a finding.
