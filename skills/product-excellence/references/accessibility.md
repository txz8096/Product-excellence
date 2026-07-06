# Pass 2 — Accessibility

The question this pass answers: **can someone with imperfect vision, no mouse, a
screen reader, or motion sensitivity actually use this?** WCAG 2.1 AA is the bar.

## Contrast (the highest-yield check)

Most failures in this codebase's history came from **opacity on small text**, not
from bad color picks. `rgba(...)` and `opacity: 0.55` composite against the real
background — judge the *effective* color.

Run this in `preview_eval` to compute effective contrast for all text nodes:

```js
(() => {
  const lum = ([r,g,b]) => { const f=c=>{c/=255;return c<=.03928?c/12.92:((c+.055)/1.055)**2.4};
    return .2126*f(r)+.7152*f(g)+.0722*f(b); };
  const parse = s => (s.match(/[\d.]+/g)||[]).map(Number);
  const blend = (fg,a,bg) => fg.map((c,i)=>Math.round(c*a+bg[i]*(1-a)));
  const bgOf = el => { for(let e=el;e;e=e.parentElement){
    const c=parse(getComputedStyle(e).backgroundColor);
    if(c.length>=3&&(c[3]===undefined||c[3]>0.9)) return c.slice(0,3);} return [255,255,255]; };
  const out=[];
  document.querySelectorAll('body *').forEach(el=>{
    if(!el.innerText||!el.innerText.trim()||el.children.length>0) return;
    const cs=getComputedStyle(el); if(cs.visibility==='hidden'||cs.display==='none')return;
    let fg=parse(cs.color); const bg=bgOf(el);
    let alpha=(fg[3]??1)*parseFloat(cs.opacity||1);
    fg=blend(fg.slice(0,3),alpha,bg);
    const L1=lum(fg),L2=lum(bg);
    const ratio=(Math.max(L1,L2)+.05)/(Math.min(L1,L2)+.05);
    const px=parseFloat(cs.fontSize), bold=parseInt(cs.fontWeight)>=700;
    const large=px>=24||(px>=18.66&&bold), req=large?3:4.5;
    if(ratio<req) out.push({t:el.innerText.trim().slice(0,40),
      sel:(el.className||el.tagName).toString().slice(0,40),
      px, ratio:+ratio.toFixed(2), req});
  });
  return out.slice(0,25);
})()
```

Interpret with judgment: decorative/duplicated text can pass at lower contrast;
primary content and anything a persona must read cannot. Findings cite the measured
ratio and the required one, and the fix prompt gives a concrete replacement (a
darker color value or a higher opacity that clears the bar — e.g. "raise
`opacity: 0.55` to `0.85`, ≈5.0:1").

**Check states, not just the resting render.** Hover, focus, disabled, selected,
and error states each have their own colors and each can fail independently —
a button that passes at rest and washes out on hover is still a finding. Trigger
the state (or read its CSS rule) and run the same math. Disabled controls are
WCAG-exempt but still flag them at P2 if barely legible: exempt isn't invisible.

## Beyond contrast — the checklist

- **Font size floors.** Body ≥ 16px, meta/labels ≥ 11px. Anything smaller needs a
  reason and extra contrast headroom.
- **Keyboard path.** Tab through the page: every interactive element reachable, in
  a sensible order, with a *visible* focus state (`:focus-visible` styling — the
  default outline removed with nothing added is a P1). Modals/panels: Escape
  closes, focus returns to the trigger.
- **Touch targets.** Interactive elements ≥ 44×44px effective hit area at mobile
  width, with enough spacing that adjacent targets aren't mis-tappable.
- **Hover-dependence.** Anything revealed only on hover (tooltips, magnifiers,
  action buttons) needs a touch/keyboard equivalent and a resting-state cue.
  This is both an a11y finding and a discoverability finding — file once, tag both.
- **Zoom and reflow.** At 200% browser zoom (emulate: halve the viewport, e.g.
  720×450 for a 1440×900 layout), text is still readable and nothing overlaps or
  clips. At 320px-wide reflow (400% equivalent), content is one column with no
  horizontal scroll. Fixed-height layouts are the usual casualty here.
- **Forms** (any input, not just long forms): every field has a programmatically
  associated label (not placeholder-as-label); validation doesn't fire while the
  user is still typing; error messages say *what's wrong and how to fix it*, are
  announced (`aria-live` or focus move), and never wipe the user's input.
- **Images and media.** Classify each image before judging its `alt`: decorative
  (`alt=""`), informative (describe the content), or functional (describe the
  action, not the picture — "Search", not "magnifying glass"). Videos that
  autoplay are muted and pausable.
- **Structure.** One `h1`; heading levels don't skip; landmarks (`nav`, `main`,
  `footer`) present; links have discernible names (no bare "here" / icon-only
  links without `aria-label`).
- **Color is not the only channel.** Anything communicated by color alone (status,
  active tab, chosen option) gets a second signal — weight, icon, underline, label.
- **Motion.** `prefers-reduced-motion: reduce` disables or tones down animation.
  Verify with `preview_resize`'s color-scheme emulation counterpart: run
  `matchMedia('(prefers-reduced-motion: reduce)')` handling exists in CSS or JS.
- **Dark mode** (only if the product claims it): re-run the contrast script with
  the dark scheme emulated.

## Async and AI-generated content

Products with AI features get three extra checks on every async surface:

- **Loading resolves.** Simulate the backend being slow or absent (block the
  request in devtools, or test on a static server): the loading state must
  resolve into content or a status within ~10 seconds — an infinite spinner is
  a P0 on a core path. If reviewing locally without the backend, verify whether
  a timeout/error path *exists in code*; "works when the API answers" is not
  the same as "handles the API not answering."
- **Errors offer a way forward.** The failure message should be in the product's
  voice and point somewhere useful (retry, alternate content), and the input
  must remain usable after the error.
- **Confidence is honest.** AI-generated content is labeled as such
  ("powered by AI, can make mistakes") somewhere the reader will see it.

## Priorities

Contrast failure or keyboard dead-end on the **primary content/CUJ path** → P0.
Same failure on secondary/meta content → P1. Missing polish (focus ring style,
decorative alt) → P2.

## Honesty rule

This pass is automated measurement plus judgment — it is **not** assistive-
technology testing. Never phrase the report as "WCAG compliant" or "accessible";
say what was checked and what passed. If the product's audience makes AT usage
likely (public-facing, enterprise procurement, older users), recommend a real
screen-reader session as a follow-up rather than implying this pass covered it.
