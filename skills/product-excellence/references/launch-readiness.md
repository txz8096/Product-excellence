# Pass 4 — Launch Readiness

The question this pass answers: **does this product survive contact with the
world — the link paste, the dead href, the deploy, the next regression?**
Passes 1–3 review the product on the screen; this pass reviews everything
around it. It's short: run it last, against the same preview.

## 1. Share card & tab presence

The real first impression of anything shared by link is the unfurl card in
Slack/LinkedIn/iMessage and the browser tab — rendered before anyone clicks.

Check in `<head>` (read the served HTML, not the source):

- `og:title`, `og:description`, `og:image` present. The image URL resolves
  (fetch it) and is ~1200×630 — other ratios get cropped by most platforms.
- `twitter:card` set (`summary_large_image` when an image exists).
- `<title>` reads sensibly truncated in a cramped tab; not "index" or a slug.
- Favicon exists and resolves.
- Meta description is a sentence worth reading, not boilerplate; on personal/
  product sites it should agree with the on-page pitch (content consistency
  applies to metadata too).

Example finding:

> **[P1·S] Link unfurls with no image.** Evidence: no `og:image` in head;
> pasting the URL renders a bare grey card. Why: the recruiter's first
> impression is the unfurl, not the hero. Fix prompt: add `og:image`
> (1200×630 export of the hero) plus `og:title`/`og:description` matching
> the tab title and meta description.

## 2. Link integrity

- Collect every `href` on every reviewed surface (`preview_eval`:
  `[...document.querySelectorAll('a[href]')].map(a => a.href)`), dedupe, and
  verify each resolves: `curl -sI -o /dev/null -w '%{http_code}'` per URL
  (HEAD, then GET for servers that reject HEAD). Internal `#anchors` must
  match a real element id.
- External links open new tabs consistently (`target="_blank"` +
  `rel="noopener"`) — and per the pattern-consistency rule, all of them or
  none of them.
- Redirect chains land somewhere sensible (a resume link that 302s to a
  revoked doc is a dead link with extra steps).

Priorities: dead link on the core CUJ (resume, contact, primary CTA) → P0;
dead footer/secondary link → P1; redirect oddities → P2.

## 3. Production re-verify

Local verdicts are conditional. Build steps, CDNs, serverless functions, env
vars, and caching all change behavior — "works on the static preview" has
been wrong before.

- The report header carries `Verified on:` — `local` until proven otherwise.
  A **Ship** verdict from local-only review is really "Ship, then confirm".
- After deploy, re-run only the P0 acceptance checks against the live URL
  (each fix prompt already says how to verify — run exactly those), plus one
  cold-load of a deep link and one submit of the core async flow.
- Report the delta: "production re-verify: 6/6 pass" or file the failures as
  new findings marked `prod-only`.

## 4. Saved acceptance checks (regression guard)

Reviews should compound, not restart. Fixes regress — a fact corrected once
has come back before (same claim living in four files; one got missed on the
next edit).

When fix mode completes, write the verified acceptance criteria to
`qa-checks.md` at the repo root (create or append, newest launch first):

```markdown
## <date> — <scope of launch>
- [ ] `.about__stats` scrollWidth ≤ clientWidth at 375px and 1440px
- [ ] Hero, about view, JSON-LD, and llms.txt all state the same career numbers
- [ ] `#topic/<id>` cold-loads the correct view; browser back returns to index
- [ ] Ask flow: loading resolves or errors within 10s; input usable after error
```

Keep each line runnable-as-written (selector + threshold, not vibes). The
**next** review starts by running the newest checklist and reports
`Regressions: 0 of N` in the header — before any new findings.

## 5. Browser-risk scan

The review runs in one Chromium; full cross-browser testing is out of scope,
but naming the risk isn't. Grep the served CSS/JS for features with uneven
support and check each for a fallback:

- View Transitions API · `:has()` · container queries · `@starting-style` ·
  `text-wrap: balance` · `backdrop-filter` · popover/anchor positioning ·
  subgrid

For each hit: is it feature-detected or progressive (page works without it),
or load-bearing (layout/flow breaks without it)? Cosmetic degradation is fine
to ship; load-bearing without a fallback is a P1.

Output is one line in the report header, not findings:

> Browser risk: View Transitions (feature-detected, falls back to instant
> swap ✓) · `text-wrap: balance` (cosmetic ✓). Nothing load-bearing lacks a
> fallback.
