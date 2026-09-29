# Motion Review (Pass 1 extension)

Run this whenever the surface under review animates — entrances, hovers, view
transitions, loading states, gesture-driven UI. Adapted from Emil Kowalski's
`review-animations` skill (github.com/emilkowalski/skills, animations.dev);
his framing applies here: **default to flagging — approval is earned.** The
strongest fix for a bad animation is usually deletion, not tuning.

## The standards

1. **Justified motion.** Every animation answers "why does this animate?" —
   spatial continuity, state indication, feedback, or softening a jarring
   change. "Looks nice" is not a reason on elements users see constantly.
2. **Frequency-appropriate.** Actions triggered 100+ times a day (and anything
   keyboard-initiated) get no animation; frequent actions get minimal motion;
   occasional actions get standard motion; rare or first-run moments may earn
   delight. Ask the persona: how often do *they* hit this?
3. **UI stays under 300ms.** Longer needs a justification, not a vibe.
4. **Easing has intent.** Entrances/exits use `ease-out` or a custom
   cubic-bezier; default CSS easings are weak, and `ease-in` on UI (slow start,
   fast end — feels unresponsive) is an automatic finding.
5. **Physical correctness.** Popovers/dropdowns/tooltips scale from their
   trigger (`transform-origin`), not from center; nothing enters from
   `scale(0)` — use `scale(0.9–0.97)` + opacity. Modals stay centered.
6. **Interruptible.** Rapidly re-triggered motion (toasts, toggles, hover
   in/out) must retarget from its current state, not restart keyframes.
   Test it: trigger twice fast and watch for the snap-to-start.
7. **GPU-only properties.** Animate `transform` and `opacity` only; animating
   layout properties (width, height, top, margin) is a performance finding.
8. **Reduced motion is gentler, not necessarily zero.** `prefers-reduced-motion`
   must be honored; hover-driven motion gated behind
   `@media (hover: hover) and (pointer: fine)` so touch devices don't get
   sticky hover states.
9. **Asymmetric timing.** User-initiated actions can ease in view; system
   responses and exits should be faster than entrances.
10. **Cohesion.** Motion matches the product's personality — a calm editorial
    site and a playful consumer app should not share the same spring.

## Instant-finding triggers

Any of these is a finding on sight: `transition: all` · entrance from
`scale(0)` or a pure fade on spatial UI · `ease-in` on UI · animation on a
keyboard-shortcut path · UI duration over 300ms unjustified ·
`transform-origin: center` on a trigger-anchored popover · keyframe animation
on rapidly re-triggered elements · layout-property animation · missing
`prefers-reduced-motion` handling · ungated `:hover` motion on touch.

## How to check from the preview

- Scan stylesheets for the triggers above (`transition: all`, `ease-in`,
  keyframes touching layout properties) — same technique as the hover/focus
  rule scan in the main pass.
- `document.getAnimations()` via browser eval lists live animations with
  durations and easing; check durations against the 300ms bar.
- Interruptibility: fire the trigger twice in quick succession and screenshot
  or measure mid-flight — a restart from the initial state is the tell.
- Reduced motion: emulate `prefers-reduced-motion: reduce` and confirm motion
  is toned down, not just present.
- When something feels off but you can't name it, slow it down (set
  `animation-duration`/`transition-duration` ×10 via browser eval) and watch
  what actually happens — then set it back.

## Fixing (order of preference, for fix prompts)

Delete the animation → reduce it (duration, distance, properties) → fix the
easing → fix the origin → make it interruptible → move it to GPU properties →
make timing asymmetric → only then polish (stagger, blur, springs). Fix
prompts should start at the top of this ladder, not the bottom.

## Priority mapping

Category: every fix on the ladder above is **Category A (fix)** — deleting,
shortening, re-easing, and honoring `prefers-reduced-motion` each have one obvious
answer. **Category B (needs design input)** is for motion that has to be invented
rather than corrected: a transition that needs new choreography, or whether a
surface should animate at all when that's a feel call with several answers.

Motion that blocks or fights the user (uninterruptible animation on a core
path, seizure-range flashing) → **P0**. Feel-breaking on frequent actions —
`ease-in`, >300ms on common UI, keyboard-path animation, layout-property jank,
missing reduced-motion → **P1**. Origin nits, missing stagger, cohesion
mismatches → **P2**.
