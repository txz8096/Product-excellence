# Preview Measurement & Interaction Pitfalls

Traps that produce false findings (or hide real ones) when driving a page
through preview tools. Each of these burned a real review once.

## Visibility checks

- **`offsetParent === null` does NOT mean hidden for `position: fixed` elements.**
  Fixed-position elements always report `offsetParent: null`, even when fully
  visible. A tooltip/modal check built on `offsetParent` will report "never
  appears" when it's on screen. Use instead:
  ```js
  const r = el.getBoundingClientRect();
  const cs = getComputedStyle(el);
  const visible = r.width > 0 && cs.display !== 'none' &&
    cs.visibility !== 'hidden' && parseFloat(cs.opacity) > 0;
  ```
- **Animation timing:** a screenshot mid fade-in shows "illegibly faint text"
  that settles at full opacity. Before filing a contrast/visibility finding on
  freshly rendered content, re-measure after the animation window (~1s) —
  computed `opacity` at rest is the fact, the screenshot is the impression.
- **`<noscript>` content is context-dependent.** Served normally, its children
  are inert text (invisible to `querySelectorAll` and the a11y tree). Injected
  via `innerHTML` (some bundlers do this), they parse as real elements. Don't
  assume hidden SEO blocks behave the same across build variants.

## Driving React (or similar) apps

- **`preview_fill` may not update framework state.** Setting `.value` directly
  doesn't fire React's onChange, so submit buttons stay disabled and your
  "click did nothing" is a broken test, not a broken product. Use the native
  setter + input event:
  ```js
  const setter = Object.getOwnPropertyDescriptor(
    window.HTMLInputElement.prototype, 'value').set;
  setter.call(input, 'text');
  input.dispatchEvent(new Event('input', { bubbles: true }));
  ```
  Symptom to watch for: after "submitting," the view is unchanged and the
  input still holds focus — verify the button wasn't disabled before claiming
  the flow is broken.
- **Synthetic `mouseenter` won't reach React listeners** (it doesn't bubble;
  React delegates via `mouseover`). To test hover-driven UI, dispatch
  `new MouseEvent('mouseover', { bubbles: true })`.
- **Programmatic `el.focus()` usually won't match `:focus-visible`** (the
  browser's input-modality heuristic sees script, not keyboard). To verify a
  focus-visible fix, confirm the rule exists in a stylesheet and rely on real
  Tab behavior; don't file "no focus style" from a script-focus measurement
  alone — check what the rule computes to.

## Layout

- **Adjacent inline elements with no whitespace between them cannot wrap.**
  JSX `.map()` output glues siblings together with zero break opportunities;
  with `nowrap` spans the whole line becomes one unbreakable run that clips.
  If a line refuses to wrap despite `white-space: normal` on the parent,
  inspect for missing text nodes between children — the fix is explicit
  `{" "}` separators, not more CSS.
- **Full-page screenshots ≠ viewport.** Some screenshot tools capture the full
  document or an upscaled canvas; don't infer viewport-relative positions from
  the image. Bounding rects from `preview_eval` are ground truth.
