# Product Excellence Skill

Vibe coding has a lopsided ratio: building something takes about 30% of the
time, and raising it to the quality you'd actually ship takes the other 70%.
On product teams, that 70% has a name — product excellence — and it runs on
review passes, checklists, and hard-won instincts about what breaks before
launch.

This skill packages those instincts into a repeatable process for Claude Code:
a standardized pre-launch design QA on any web surface that ends in a
prioritized, paste-ready fix list instead of a critique essay.

It was built by a designer from months of real design-iteration sessions:
the fixes that kept being requested got promoted into defaults, and the
measurement mistakes that produced false findings got documented so they
never happen twice.

## What it does

Say `run a product excellence review on <page>` (or "pre-launch check", "is
this ready to ship") and Claude drives the **rendered page** — real browser,
real viewports, real clicks — through four passes:

1. **Visual rhythm & responsiveness** — viewport matrix (laptop / small laptop /
   mobile / 200% zoom / tall screens), programmatic overflow detection, word-break
   and orphan checks, spacing-scale audits (values must sit on a scale, not a
   scatter), cross-page consistency against the design source of truth. If the
   surface animates, motion gets reviewed against Emil Kowalski's standards.
2. **Accessibility** — measured contrast (compositing opacity into effective
   colors, checking hover/focus/disabled states, not just the resting render),
   keyboard paths and focus visibility, touch targets, zoom reflow, forms,
   heading/landmark structure, reduced motion, and async/AI states (loading must
   resolve, errors must offer a way forward).
3. **Persona CUJ walkthrough** — 2–3 personas derived from what the product is
   for (one always carries an access need, woven in naturally), walked through
   the core journeys step by step: what they want, what they actually see, what
   questions they'd have, whether the next action is discoverable, what's
   missing or confusing. Cognitive load gets scored, not just task completion.
4. **Launch readiness** — the product's life outside the viewport: the
   share-card unfurl and browser tab (`og:` tags, favicon, title), link
   integrity (every href resolves), a production re-verify step (local
   verdicts are conditional), a browser-risk scan for unevenly-supported
   features, and a regression guard — verified acceptance checks get saved to
   `qa-checks.md` so the next review catches regressions before hunting new
   findings.

### The output contract

Every review ends in the same structure: a ship / ship-after-P0s / hold verdict
(with a "verified on: local vs production" qualifier and a browser-risk line),
findings ordered **P0 (launch blocker) → P1 (fix before sharing) → P2 (polish)**
and tagged **S/M/L for effort** so the cut line is negotiable ("all P0s plus the
P1-smalls"), and — the actual deliverable — a **self-contained fix prompt per
finding** that names the file, selector, current value, target value, and how to
verify. Each prompt works in a brand-new Claude Code session with zero context.
A closing "what's working" section keeps good patterns from being "fixed" later.

Findings require evidence (viewport, selector, measured value, or walkthrough
step). A finding without evidence is an opinion, and the skill doesn't ship
opinions.

### Fix mode

Say "fix it" and the fix prompts become work orders: read the repo's standing
rules first, fix in source (not in generated output), re-verify each finding
against its own acceptance criterion, then sync the design doc and any
machine-readable content copies (JSON-LD, llms.txt, meta) the fixes touched.

## What's inside

| File | What it holds |
|---|---|
| [skills/product-excellence/SKILL.md](skills/product-excellence/SKILL.md) | The process: scoping, the three passes, house standards, output format, fix mode |
| [references/visual-rhythm.md](skills/product-excellence/references/visual-rhythm.md) | Pass 1 checklist + overflow-detection script |
| [references/motion.md](skills/product-excellence/references/motion.md) | Animation bar, adapted from [Emil Kowalski's review-animations](https://github.com/emilkowalski/skills) (animations.dev) |
| [references/accessibility.md](skills/product-excellence/references/accessibility.md) | Pass 2 checklist + a contrast script that composites opacity |
| [references/cuj-walkthrough.md](skills/product-excellence/references/cuj-walkthrough.md) | Pass 3: persona construction, walkthrough protocol, honesty rules |
| [references/launch-readiness.md](skills/product-excellence/references/launch-readiness.md) | Pass 4: share card, link integrity, production re-verify, saved acceptance checks, browser-risk scan |
| [references/preview-pitfalls.md](skills/product-excellence/references/preview-pitfalls.md) | Measurement traps that each burned a real review once (`offsetParent` lies about fixed elements, `preview_fill` vs. React state, `:focus-visible` heuristics…) |

## Install

**Manual (Claude Code):**

```bash
git clone https://github.com/txz8096/product-excellence.git
cp -r product-excellence/skills/product-excellence ~/.claude/skills/
```

Or per-project: copy into `<repo>/.claude/skills/` instead.

**With the skills installer:**

```bash
npx skills@latest add txz8096/product-excellence
```

## Notes on honesty

Two rules are load-bearing and worth knowing before you trust a report:

- The accessibility pass is automated measurement plus judgment — it is **not**
  assistive-technology testing, and reports never claim "WCAG compliant."
- Reviews on a local/static server distinguish environment artifacts from
  product bugs: "the API is down locally" is not a finding; "there is no
  timeout or error path even if production works" is.

## Credits

- Motion standards adapted from [Emil Kowalski](https://animations.dev)'s
  `review-animations` skill, under his "default to flagging; approval is
  earned" philosophy.
- Repo layout inspired by [mattpocock/skills](https://github.com/mattpocock/skills).

## License

MIT
