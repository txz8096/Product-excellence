---
name: product-excellence
description: >
  Standardized pre-launch design QA that ends in a prioritized, paste-ready fix list.
  Runs three passes over the rendered product: (1) visual rhythm and responsiveness,
  (2) accessibility, (3) persona-based CUJ walkthroughs. Use whenever the user says
  "product excellence", "pre-launch review", "launch check", "is this ready to ship",
  "review before I deploy/publish", "visual rhythm check", "persona walkthrough", or
  is about to launch, deploy, publish, or share any page, site, feature, or prototype.
  Even if the user asks for only one of the three checks by name, this skill is the
  process to follow — run the requested pass with this skill's standards, and offer
  the remaining passes.
---

# Product Excellence Review

A pre-launch gate. The goal is not a critique essay — it is a **prioritized list of
findings the user can paste straight back into Claude Code as fix prompts**. Every
finding must be specific enough that a fresh session with no context could act on it.

## Step 0 — Scope and ground truth

Before reviewing anything:

1. **Identify what's launching.** Which files/pages/routes? If the user didn't say,
   ask or infer from recent git changes. List the surfaces you will review.
2. **Find the design source of truth.** Look for `design.md`, a style guide, design
   tokens, or — most often — the existing sibling pages of the same product.
   Consistency findings are judged against this, not against your own taste. If the
   product has an established voice (colors, type scale, card treatments, hover
   language), the new surface must speak it.
3. **Render it for real.** Never review from source code alone; the page as rendered
   is the product. Start a preview server (`preview_start`, or a static server for
   plain HTML) and drive the actual page. Interactions must be exercised, not read
   about: click the buttons, open the panels, hover the cards, submit the inputs.

## The three passes

Run all three, in this order (each pass reuses the running preview). Read the
reference file when you begin each pass:

1. **Visual rhythm & responsiveness** — read [references/visual-rhythm.md](references/visual-rhythm.md);
   if the surface animates, also read [references/motion.md](references/motion.md)
2. **Accessibility** — read [references/accessibility.md](references/accessibility.md)
3. **Persona CUJ walkthrough** — read [references/cuj-walkthrough.md](references/cuj-walkthrough.md)

Collect findings as you go with evidence attached (viewport, selector, measured
value, or walkthrough step). A finding without evidence is an opinion; don't ship
opinions.

## House standards

These were mined from months of real design-iteration sessions — the fixes that
kept being requested, promoted into defaults. Treat them as the bar:

- **Viewport matrix:** 1440×900 (laptop), 1280×720 (small laptop), 375×812 (mobile),
  plus one 200%-zoom check (emulate: 720×450 viewport) — zoomed browsing is how a
  large share of 50+ users actually read. If the layout is fixed-height
  (`body { overflow: hidden }`), also test a tall screen (≥1000px height) —
  fixed-height heroes tuned at ~750px break there.
- **Measure, don't eyeball.** Use `preview_inspect` for computed CSS (font-size,
  padding, color, line-height) — it is exact and cheap. Screenshots are for gestalt
  and layout only; never read a color or a font size off a screenshot.
- **Console must be clean.** Any console error at load or during interaction is a
  finding, minimum P1.
- **Contrast is the usual offender via opacity.** Small labels and meta text set at
  `opacity: 0.5–0.7` routinely fail WCAG AA. Composite the opacity into the
  effective color before judging.
- **Reading measure:** body text line length caps around 720px; line-height 1.5–1.6
  for body copy. Headlines get `text-wrap: balance` to avoid orphans.
- **Cross-page consistency:** typography, spacing tokens, colors, tag styles, hover
  states, and card/thumbnail treatments must match the sibling pages exactly. "Same
  layout, typography, spacing, tags, hover states" is the standing instruction.
- **Hover-only affordances are a discoverability bug.** If the only cue that
  something is interactive appears on hover, it's dead on arrival — and invisible
  on touch. Require a visible resting-state cue and a touch equivalent.
- **Mobile overflow is a repeat offender.** At 375px: no horizontal scroll, no
  clipped stat lines or badges, images constrained with `object-fit`, grids collapse
  to a clean single column. Check `document.documentElement.scrollWidth` vs
  viewport width — it catches clipping that screenshots hide.
- **Motion:** animations should be opacity/transform only (no layout thrash),
  respect `prefers-reduced-motion`, ease out on entrances, and stay under 300ms
  on UI. Full bar (Emil Kowalski's standards) in
  [references/motion.md](references/motion.md).
- **Pre-existing vs. new:** if an issue predates the change under review (check git
  history or `.bak` files), still list it, but flag it "pre-existing" so the user
  can decide whether it's in scope for this launch.
- **Content consistency counts too:** numbers, tenure claims, section numbering, and
  labels must agree across pages. A stat that says "7 years" on one page and "8" on
  another is a P1.
- **Facts live in more places than the visible UI.** Numbers and claims are usually
  duplicated in machine-readable copies: JSON-LD, meta descriptions, `llms.txt`,
  hidden SEO/noscript blocks, data files. Grep the repo for the old value — a real
  case had the same tenure claim in four places and the visible fix missed two.
- **URL state is part of the product.** Views reachable only through JS clicks
  (no href, no hash/route) can't be shared, bookmarked, middle-clicked, or safely
  backed out of. Check: does each view have a URL? Does browser back return to the
  previous view or exit the site? Nav "links" that are `<button>`s are a finding.
- **AI features must resolve.** Any async/AI-generated content needs three states
  verified: loading resolves within ~10s or shows a status, failure shows a
  friendly message with a path forward (not a spinner forever), and the input
  stays usable after an error. Test by blocking the API in devtools if needed.

## Output format

ALWAYS use this exact structure:

```
# Product Excellence Review — <target>
Reviewed at: <viewports tested> · Console: <clean / N errors> · Scope: <pages>

## Verdict
<Ship / Ship after P0s / Hold> — one sentence why.

## Findings
| # | Pri | Pass | Finding |
|---|-----|------|---------|
(one row per finding, ordered by priority then impact)

### 1. [P0] <short title>
- **Evidence:** <viewport / selector / measured value / walkthrough step>
- **Why it matters:** <one sentence, tied to a user or persona>
- **Fix prompt:** ↓ paste-ready

> In <file>, <specific instruction with selectors, values, and acceptance
> criteria. Self-contained — assume the reader has zero context.>

(repeat per finding)

## What's working
<3–5 bullets. This prevents good patterns from being "fixed" in the next session.>
```

**Priorities:**
- **P0 — launch blocker.** Content unreachable, clipped, or broken; a core CUJ
  dead-ends; contrast failure on primary content; console errors; horizontal
  overflow on mobile.
- **P1 — fix before sharing widely.** Comprehension or trust harmed: inconsistency
  with sibling pages, confusing labels, missing information a persona needs,
  contrast failures on secondary text, hover-only affordances.
- **P2 — polish.** Rhythm stutters, spacing that's off-scale, nice-to-have
  affordances, minor motion issues.

Within a priority, order by (users affected × how early in the journey they hit it).

**Fix prompts are the deliverable.** Each must name the file, the selector or
element, the current value, the target value, and how to verify ("at 375px,
`scrollWidth` equals viewport width"). Write them so they work in a brand-new
Claude Code session.

## Local environment vs. production

Reviewing on a local/static server means some failures are artifacts, not bugs.
Before filing a P0 on a broken flow, check *which code path actually ran*: did a
network request fire at all (`preview_network`)? Is a backend (serverless
functions, API keys) simply absent locally? Preview harnesses can also inject
globals (e.g. a `window.claude` runtime) that change the app's behavior. Rule:
report what you observed, name the environment caveat explicitly, and judge the
product on what's still true regardless — e.g. "even if production works, there
is no timeout or error path" is a valid finding; "the API is down" locally is not.

When measuring, know the traps in [references/preview-pitfalls.md](references/preview-pitfalls.md)
— read it the first time a measurement or interaction result looks contradictory
(an element that's "invisible" but clearly on screen, a click that does nothing,
a fill that doesn't enable the submit button).

## Fix mode

When the user says "fix it," the review's fix prompts become your work orders:

1. **Read the repo's standing rules first** (CLAUDE.md, contributing docs).
   Package-manager choices, lockfile rules, "don't create .bak files," design-system
   boundaries — violating house rules while fixing design bugs is a net negative.
   Check which branch you're on before touching anything.
2. **Fix in source, not in output.** If the page is generated or bundled, find the
   real source. If the "source" is unreadable (a 3MB bundle), unpacking it into an
   editable source tree may itself be one of the fixes — confirm parity in the
   preview before layering other changes on top.
3. **Re-verify each finding against its own acceptance criterion** — the fix prompt
   said how to verify; run exactly that check. Measure again, don't assume.
4. **Sync the design source of truth afterward.** After a fix pass the design doc
   and the shipped page have usually diverged; ask (or follow the user's standing
   choice) about which direction wins, then update the doc to match. Also update
   any machine-readable content copies the fixes touched.
5. **Leave the workflow documented.** If fixing changed how the project is edited
   (new build step, new file layout), record it in the project's CLAUDE.md so the
   next session doesn't undo it.

## Scaling the review

- Small change (one component): run all three passes but scoped to the changed
  surface plus one sibling page for consistency.
- Full launch (new page/site): full viewport matrix, all interactive elements,
  2–3 personas × 2 CUJs each.
- If the user asks for just one pass, run it at full depth and note which of the
  other two passes you'd recommend before launch and why.
