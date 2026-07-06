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

### Cross-page consistency check

Open one established sibling page side by side. Compare: header/nav treatment, card
anatomy (padding, tag row, hover state), type scale, accent-color usage, footer.
Every divergence is a finding unless the design source of truth sanctions it.

## Evidence format

Every finding cites: viewport, selector, measured value, expected value.
"`.topic__name` clips at 375px (scrollWidth 402 > 375)" — good.
"Mobile looks cramped" — not a finding.
