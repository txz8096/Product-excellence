# Pass 3 — Persona-Based CUJ Walkthrough

The question this pass answers: **when a real person with a real goal lands here,
do they succeed — and what does it cost them in confusion?** Passes 1 and 2 check
the surface; this pass checks the experience.

## Step 1 — Construct the personas (2–3)

Derive them from what the product is *for*, not from demographics. If the user has
already defined personas (in the repo, a case study, or conversation), use those.
Otherwise build each persona from four dimensions:

- **Goal:** what they came to accomplish (be concrete: "decide in 90 seconds
  whether this designer is worth a phone screen", not "learn about the site").
- **Expertise:** domain fluency — do they know the jargon on this page?
- **Context:** device, time pressure, how they arrived (link from where?).
- **Skepticism/patience:** what makes them bounce.

Pick personas that *disagree* — a skimmer and a deep-reader, an expert and a
novice — so the walkthrough stresses different parts of the page. Name them and
state the four dimensions in the report; findings will cite them.

**One persona carries an access need, woven in naturally.** Not a "disability
persona" bolted on — a real person whose context includes a constraint: the
62-year-old director who browses at 200% zoom and distrusts unlabeled icons;
the engineer who navigates entirely by keyboard because of RSI; the parent
holding a baby, driving the phone one-handed with their thumb. Their walkthrough
*uses* the constraint at every step (the zoomed persona sees a quarter of the
layout at a time; the keyboard persona lives or dies by focus order). This is
how accessibility findings get user impact attached instead of a WCAG number.

Example (portfolio site): a hiring manager skimming 40 portfolios on a laptop
between meetings; a design peer on mobile who follows craft details; a recruiter
who needs keywords and contact info fast, doesn't know the domain jargon, and
reads at 200% zoom on an aging laptop.

## Step 2 — Enumerate the CUJs (2 per persona, max ~5 total)

Always include:
1. **First-visit orientation:** land cold → understand what this is, who it's for,
   and what to do next — within ~10 seconds of scanning.
2. **The core task:** the one job the product exists to do, end to end.

Add where relevant: a return visit ("find the thing I saw last time"), an edge
entry (deep link into an inner page — does it orient without the homepage?), and
the skeptic's probe ("prove this claim").

## Step 3 — Walk each CUJ against the real render

Drive the actual preview, one step at a time, **as the persona**. At every step
record five things:

1. **Wants:** what the persona is trying to do right now.
2. **Sees:** what's actually visible (use snapshot/screenshot — not the source
   code, and not what you know is there because you read the HTML).
3. **Questions:** what they'd ask at this moment. Unanswered questions are
   findings. ("What does 'Nexus' mean?" from the non-domain persona is real.)
4. **Next action discoverable?** Could they find the next step without hovering
   at random or scrolling on faith? Hover-only reveals, unlabeled icons, and
   below-the-fold-only CTAs fail this.
5. **Missing or confusing:** information they need that isn't there, jargon that
   isn't earned, labels that mislead, states that don't explain themselves
   (empty states, loading, errors).

Honesty rules for the simulation:
- The persona reads at scan speed. If key information only exists inside a long
  paragraph, the skimmer persona does not have it.
- The persona doesn't know the site's internal vocabulary until the page teaches
  it. Track when each jargon term is first used vs. first explained.
- The mobile persona never hovers.
- If *you* had to search the DOM to answer one of the persona's questions, the
  persona couldn't answer it at all — that's a finding, not a success.

## Step 4 — Score learnability and cognitive load per CUJ

After walking each CUJ, summarize: did the persona complete it? How many
confusion events (questions 3 and 5 above) occurred? Where would they have
bounced? A CUJ that completes but accumulates 4+ confusion events is a P1 even
though "it works".

Also score three cognitive-load signals across the journey:

- **Memory demands:** what must the persona hold in their head between steps?
  (A code shown on screen A and typed on screen B; a plan name they must
  remember to compare.) Anything the UI could carry for them is a finding.
- **Decision fatigue:** count the choices per screen on the core path. Eight
  near-identical options with no default or grouping is a finding even if each
  option is individually clear.
- **Reading level of critical copy:** error messages, CTAs, and onboarding
  should read at roughly conversational level. Flag critical copy that requires
  domain fluency the persona doesn't have — decorative copy can be fancy;
  instructions can't.

## Converting walkthrough notes into findings

- CUJ dead-end or wrong-destination (persona cannot complete, or completes the
  wrong thing believing it's right) → **P0**.
- Needed information missing/undiscoverable, jargon unexplained on the persona's
  path, misleading label → **P1**.
- Friction that slows but doesn't block (extra scroll, weak affordance with a
  workaround) → **P2**.

Each finding cites the persona and step: "Recruiter persona, CUJ 1 step 2:
'AutoTFX' appears in the hero with no gloss; she can't map it to a skill she's
hiring for." The fix prompt then says exactly what to add and where.
