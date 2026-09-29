---
name: product-excellence
description: >
  Standardized pre-launch design QA that ends in a prioritized fix list, split into
  straightforward fixes and findings that need designer input.
  Runs four passes over the rendered product: (1) visual rhythm and responsiveness,
  (2) accessibility, (3) persona-based CUJ walkthroughs, (4) launch readiness
  (share card, link integrity, production re-verify, regression guard). Use whenever the user says
  "product excellence", "pre-launch review", "launch check", "is this ready to ship",
  "review before I deploy/publish", "visual rhythm check", "persona walkthrough", or
  is about to launch, deploy, publish, or share any page, site, feature, or prototype.
  Even if the user asks for only one of the three checks by name, this skill is the
  process to follow — run the requested pass with this skill's standards, and offer
  the remaining passes.
---

# Product Excellence Review

A pre-launch gate. The goal is not a critique essay — it is a **prioritized list of
findings the user can paste straight back into their coding agent as fix prompts**,
split into two categories: straightforward fixes, and findings that need designer
input before anyone touches them. Every finding must be specific enough that a fresh session
with no context could act on it.

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
   is the product. Serve it (dev server or static server) and drive the actual page
   in a browser. Interactions must be exercised, not read about: click the buttons,
   open the panels, hover the cards, submit the inputs.

## Tooling: what this skill needs

The process is platform-agnostic — it runs on any agent that can drive a browser.
Five capabilities, and where they come from:

| Capability | Claude Code | Other agents (Cursor, Codex, …) |
|---|---|---|
| Serve + render the page | `preview_start` | dev/static server + a browser tool (Playwright, Chrome DevTools MCP) |
| Run JS in the page — "**browser eval**" | `preview_eval` | Playwright `page.evaluate`, DevTools MCP evaluate |
| Resize the viewport | `preview_resize` | `page.setViewportSize`, DevTools emulation |
| Screenshot / accessibility snapshot | `preview_screenshot` / `preview_snapshot` | `page.screenshot`, a11y tree dump |
| Console + network logs | `preview_console_logs` / `preview_network` | DevTools MCP, Playwright tracing |

The reference files say "browser eval" for the run-JS capability — map it to
whatever your platform calls it. If the platform has no browser tool at all, write
a small Playwright script and run it from the shell: every check in this skill is
plain browser JavaScript and pastes into `page.evaluate` unchanged.

## The four passes

Run all four, in this order (each pass reuses the running preview). Read the
reference file when you begin each pass. If the repo has a `qa-checks.md`
(saved acceptance checks from a previous review), run its newest checklist
**before** pass 1 and report regressions first.

1. **Visual rhythm & responsiveness** — read [references/visual-rhythm.md](references/visual-rhythm.md);
   if the surface animates, also read [references/motion.md](references/motion.md)
2. **Accessibility** — read [references/accessibility.md](references/accessibility.md)
3. **Persona CUJ walkthrough** — read [references/cuj-walkthrough.md](references/cuj-walkthrough.md)
4. **Launch readiness** — read [references/launch-readiness.md](references/launch-readiness.md):
   share card and tab presence, link integrity, production re-verify,
   saved acceptance checks, browser-risk scan

Collect findings as you go with evidence attached (viewport, selector, measured
value, or walkthrough step). A finding without evidence is an opinion; don't ship
opinions. Sort each finding into Category A (fix) or Category B (needs design input)
as you record it — the routing rules are under
[Two categories of findings](#two-categories-of-findings).

## House standards

These were mined from months of real design-iteration sessions — the fixes that
kept being requested, promoted into defaults. Treat them as the bar:

- **Viewport matrix:** 1440×900 (laptop), 1280×720 (small laptop), 375×812 (mobile),
  plus one 200%-zoom check (emulate: 720×450 viewport) — zoomed browsing is how a
  large share of 50+ users actually read. If the layout is fixed-height
  (`body { overflow: hidden }`), also test a tall screen (≥1000px height) —
  fixed-height heroes tuned at ~750px break there.
- **Measure, don't eyeball.** Read computed CSS (font-size, padding, color,
  line-height) via browser eval or your platform's inspect tool — it is exact and
  cheap. Screenshots are for gestalt and layout only; never read a color or a font
  size off a screenshot.
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
  another is a P1. Same for terminology and interaction patterns — one name per
  thing, one behavior per gesture (sweep procedure in
  [references/visual-rhythm.md](references/visual-rhythm.md)).
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

## Two categories of findings

Every finding lands in exactly one category. The test is **how many good answers
the problem has**:

- **Category A — Fix.** One obvious answer. The defect and its correct fix are
  both determined, any competent implementer would land in the same place, and a
  fresh agent session could apply it from the fix prompt alone. Mobile overflow,
  contrast failures, dead links, console errors, off-scale spacing where a scale
  already exists, a nav `<button>` that should be an `<a href>`, a missing
  `og:image` when the product already has a hero image to use.
- **Category B — Needs design input.** Several answers, and the design isn't
  settled enough to pick one. The defect is real and measured, but solving it
  could go two or three defensible ways — cut the content, reflow it, or rewrite
  it shorter; add a section, or fold it into an existing one — and which way it
  goes is a call the designer makes, not the review. Anything requiring copy or
  imagery that doesn't exist yet, a new component or state, or a naming/IA
  decision lands here too: the fix has to be designed before it can be written.

Routing rules, applied in order:

1. If the target value is derivable from the design source of truth or a sibling
   page, there's only one answer → **A**, however bad the bug is. "Match the
   sibling page's 24px card padding" is A, not B.
2. If you can write down two or three different fixes that would each be
   defensible, or the fix needs copy, imagery, or a screen state that doesn't
   exist yet → **B**.
3. If the minimum correct fix is A but a better fix would be B, file it once as A
   and note the B upgrade inside it as an optional direction. Never split one
   defect across both categories.
4. Don't use B to duck work. A finding is B only if you can state the specific
   question the designer has to answer, and name the options. If you can't do
   both, it's A.

Priorities work identically in both categories — P0 is a launch blocker whether
or not a human has to unblock it. A P0 in Category B means the launch is blocked
on a design decision, not on engineering time; the verdict must say so.

## Output format

ALWAYS use this exact structure:

```
# Product Excellence Review — <target>
Reviewed at: <viewports tested> · Console: <clean / N errors> · Scope: <pages>
Verified on: <local / production / both> · Regressions: <0 of N saved checks / no saved checks>
Browser risk: <uneven-support features found, each with/without fallback — or "none">
Findings: <N> — <n> straightforward fixes (A), <n> need design input (B) · Blockers: <n> P0 (<n> need a design call)

## Verdict
<Ship / Ship after P0s / Hold> — one sentence why. (A Ship verdict verified
only locally is "Ship, then confirm on production". If any P0 is Category B,
name it here: the launch is waiting on a design decision, not on a fix.)

## Findings
| # | Cat | Pri | Effort | Pass | Finding |
|---|-----|-----|--------|------|---------|
(one row per finding — all findings in one index, Cat is A or B, A rows first,
then B; within a category ordered by priority, then impact)

## A. Fixes — paste-ready

### A1. [P0·S] <short title>
- **Evidence:** <viewport / selector / measured value / walkthrough step>
- **Why it matters:** <one sentence, tied to a user or persona>
- **Fix prompt:** ↓ paste-ready

> In <file>, <specific instruction with selectors, values, and acceptance
> criteria. Self-contained — assume the reader has zero context.>

(repeat per Category A finding)

## B. Needs design input — decisions before fixes

### B1. [P1·M] <short title>
- **Evidence:** <viewport / selector / measured value / walkthrough step>
- **Why it matters:** <one sentence, tied to a user or persona>
- **Decision needed:** <the one question for the designer, phrased so it can be
  answered in a sentence or a sketch>
- **Options:**

| Option | What it costs | What it buys |
|---|---|---|
(2–3 real variations, each implementable — no "do it properly" non-options)

- **Recommendation:** <which option, and why — have an opinion>
- **Unblocks with:** <the exact input needed: a line of copy, a token value, a
  spec for one state, an image at a named size>
- **Fix prompt (once decided):** ↓ paste-ready, written against the recommendation

> In <file>, <instruction with the decided value as a named placeholder, e.g.
> `<APPROVED_HEADLINE>`, plus selectors and acceptance criteria.>

(repeat per Category B finding)

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

**Effort tags:** every finding also carries **S** (copy/CSS one-liner), **M**
(component logic, an hour-ish), or **L** (architectural — a build step, a
routing change). Impact decides priority; effort decides the cut line — it
lets the user say "all P0s plus the P1-smalls before I share this" instead of
relitigating each item. For Category B, effort tags the implementation after
the decision, not the deciding.

**Fix prompts are the deliverable.** Each must name the file, the selector or
element, the current value, the target value, and how to verify ("at 375px,
`scrollWidth` equals viewport width"). Write them so they work in a brand-new
agent session, whatever the platform. A Category B fix prompt is the same thing
with the undecided value left as a named placeholder — it becomes runnable the
moment the decision lands.

## Local environment vs. production

Reviewing on a local/static server means some failures are artifacts, not bugs.
Before filing a P0 on a broken flow, check *which code path actually ran*: did a
network request fire at all (check the network log)? Is a backend (serverless
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

When the user says "fix it," **Category A is the work order**. Category B is not
yours to resolve — picking between the variations is the designer's call. Apply
every Category A fix, then list the Category B items again as open decisions with
their recommendations. Apply a Category B fix only once the user answers its
question (or says "take your recommendations") — and when they do, record the
decision in the commit or the design doc so the choice is traceable, not buried
in a diff.

1. **Read the repo's standing rules first** (CLAUDE.md, AGENTS.md, .cursor/rules,
   contributing docs — whichever the repo uses).
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
   (new build step, new file layout), record it in the project's agent instructions
   file (CLAUDE.md / AGENTS.md / .cursor/rules) so the next session doesn't undo it.
6. **Save the acceptance checks.** Write the criteria you just verified to
   `qa-checks.md` at the repo root (format in
   [references/launch-readiness.md](references/launch-readiness.md)) so the next
   review starts by catching regressions instead of rediscovering them.

## Scaling the review

- Small change (one component): run all four passes but scoped to the changed
  surface plus one sibling page for consistency.
- Full launch (new page/site): full viewport matrix, all interactive elements,
  2–3 personas × 2 CUJs each.
- If the user asks for just one pass, run it at full depth and note which of the
  other three passes you'd recommend before launch and why.
