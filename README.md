# Product Excellence Skill

Vibe coding has a lopsided ratio: building something takes about 30% of the
time, and raising it to the quality you'd actually ship takes the other 70%.
On product teams, that 70% has a name — product excellence — and it runs on
review passes, checklists, and hard-won instincts about what breaks before
launch.

This skill gives your coding agent those instincts — Claude Code, Cursor,
Codex, or anything else that can drive a browser. One request, and it drives
your **rendered page** — real browser, real viewports, real clicks — through
the pre-launch review a design team would run, and hands back a ranked list of
what to fix, each item written as a prompt you can paste straight back into
the agent.

You don't need to know what to look for. That's the point.

## What a finding looks like

Every finding has evidence (measured, not vibes), a reason tied to a real
user, and a fix prompt that works in a fresh session with zero context:

> ### 3. [P1·S] Hero stat line is ellipsis-clipped at every viewport
> - **Evidence:** `.about__stats` — scrollWidth 415 > clientWidth 400 at
>   1440×900; renders as "complex too…". Same clip at 375px.
> - **Why it matters:** it's the credential line — the first thing a skimming
>   recruiter reads, and it's visibly truncated on their first scan.
> - **Fix prompt:** ↓ paste-ready
>
> > In index.html, the hero stats element `.about__stats` is clipped by
> > `white-space: nowrap; overflow: hidden; text-overflow: ellipsis` inside
> > the 400px hero column. Remove the nowrap/ellipsis and let it wrap to two
> > lines. Acceptance: at 1440×900 and 375×812 the full text is visible
> > (element scrollWidth ≤ clientWidth).

The report opens with a verdict — **Ship / Ship after P0s / Hold** — and
findings are ranked P0 (blocker) → P1 (fix before sharing) → P2 (polish),
each tagged S/M/L for effort so you can draw a realistic cut line ("all P0s
plus the P1-smalls today"). A closing "what's working" section keeps the good
patterns from being accidentally "fixed" later.

## Quickstart

Install with the skills installer (it asks which agents to target — Claude
Code, Cursor, Codex, and others):

```bash
npx skills@latest add txz8096/product-excellence
```

Or manually, per agent:

```bash
git clone https://github.com/txz8096/product-excellence.git
```

- **Claude Code:** `cp -r product-excellence/skills/product-excellence ~/.claude/skills/`
  (or into `<your-repo>/.claude/skills/` for one project)
- **Cursor:** keep the folder in your repo and add a project rule that says
  *"for any pre-launch / design review request, read
  `skills/product-excellence/SKILL.md` and follow it"*
- **Codex (or any AGENTS.md-based agent):** same folder, plus one line in
  `AGENTS.md` pointing at the SKILL.md

The skill needs an agent that can drive a browser (render, run JS, resize,
screenshot). Claude Code has this built in; for other agents a Playwright or
Chrome DevTools MCP works — and if there's no browser tool at all, the agent
can run every check through a small Playwright script, since they're all
plain browser JavaScript. The SKILL.md includes the capability map.

Then point your agent at anything a browser can render — a local HTML file,
a dev server, a deployed URL:

```
run a product excellence review on index.html
```

It also triggers on "pre-launch check", "is this ready to ship?", and
"review this before I deploy". A full review of a small site takes a few
minutes; the agent resizes viewports, clicks through flows, and measures as
it goes.

When the report is back, say **"fix it"** — the fix prompts become work
orders: the agent applies them in source, re-verifies each against its own
acceptance criterion, and saves the checks to `qa-checks.md` so the *next*
review catches regressions before hunting new findings.

## The four passes

1. **Visual rhythm & responsiveness** — does the page hold its composure at
   laptop, small-laptop, phone, 200% zoom, and tall screens? Overflow is
   detected programmatically, spacing is audited against a scale (three cards
   with padding 24/22/25 is a finding), content gets stress-tested with
   hostile-but-legal strings, and layout shift is measured, not eyeballed.
   Animated surfaces get held to [Emil Kowalski's motion bar](skills/product-excellence/references/motion.md).
2. **Accessibility** — measured contrast that accounts for opacity and checks
   hover/focus/disabled states, keyboard paths with visible focus, touch
   targets, zoom reflow, heading/landmark structure, reduced motion, and
   async/AI states (loading must resolve; errors must offer a way forward).
3. **User walkthroughs** — 2–3 realistic users derived from what the product
   is for (one always carries an access need, woven in naturally), walked
   through the core journeys step by step: what they want, what they actually
   see, what would confuse them, whether the next step is discoverable.
4. **Launch readiness** — the product's life outside the viewport: the
   link-unfurl card and browser tab, every href checked for dead links, a
   production re-verify step, a browser-support risk scan, and the saved
   regression checks.

## What's inside

| File | What it holds |
|---|---|
| [skills/product-excellence/SKILL.md](skills/product-excellence/SKILL.md) | The process: scoping, the four passes, house standards, output format, fix mode |
| [references/visual-rhythm.md](skills/product-excellence/references/visual-rhythm.md) | Pass 1 checklist + overflow and layout-shift scripts |
| [references/motion.md](skills/product-excellence/references/motion.md) | Animation bar, adapted from [Emil Kowalski's review-animations](https://github.com/emilkowalski/skills) (animations.dev) |
| [references/accessibility.md](skills/product-excellence/references/accessibility.md) | Pass 2 checklist + a contrast script that composites opacity |
| [references/cuj-walkthrough.md](skills/product-excellence/references/cuj-walkthrough.md) | Pass 3: persona construction, walkthrough protocol, honesty rules |
| [references/launch-readiness.md](skills/product-excellence/references/launch-readiness.md) | Pass 4: share card, link integrity, production re-verify, regression guard, browser risk |
| [references/preview-pitfalls.md](skills/product-excellence/references/preview-pitfalls.md) | Measurement traps that each burned a real review once (`offsetParent` lies about fixed elements, filling inputs vs. React state, `:focus-visible` heuristics…) |

Where the standards come from: months of real design-iteration sessions with
Claude Code. The fixes that kept being requested got promoted into defaults;
the measurement mistakes that produced false findings got documented so they
never happen twice.

## Notes on honesty

Two rules are load-bearing and worth knowing before you trust a report:

- The accessibility pass is automated measurement plus judgment — it is **not**
  assistive-technology testing, and reports never claim "WCAG compliant."
- Reviews on a local/static server distinguish environment artifacts from
  product bugs: "the API is down locally" is not a finding; "there is no
  timeout or error path even if production works" is. The verdict says what
  it was verified on.

## Credits

- Motion standards adapted from [Emil Kowalski](https://animations.dev)'s
  `review-animations` skill, under his "default to flagging; approval is
  earned" philosophy.
- Repo layout inspired by [mattpocock/skills](https://github.com/mattpocock/skills).

## License

MIT
