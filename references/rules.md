# Rules, budgets, and the pre-ship gate

> Merged from: web-animation-skills `accessible-animation` / `60fps-animation` (iart-ai, MIT), genjutsu `motion-principles` (AThevon, MIT), auteur `motionqa` gate concept (agiwhitelist, MIT), greensock gsap-skills (MIT). Full credits in ATTRIBUTION.md.

The contract in SKILL.md says *what*; this file says *how to check it*. Hard rules are embedded in every recipe — this is where they're tested.

## 1. Reduced motion, tiered — never a kill switch

The OS setting signals that **vestibular motion causes nausea and migraines**. The correct response removes dangerous motion and keeps orientation — removing everything (teleporting elements, dead spinners) is also a failure.

| Tier | Motion | Reduced behavior |
| --- | --- | --- |
| **1 — remove** | Parallax, large-area slides, scale/zoom of large elements, 3D rotation, auto-carousels, marquees, smooth-scroll hijack, pinned scroll scenes, custom cursor, tilt | Gone (`transform: none` / `animation: none` / loop never starts / library never instantiated) |
| **2 — soften** | Small slides, pop-ins, entrances | Cross-fade ≤ 200ms, no displacement |
| **3 — keep** | Opacity fades, color transitions, focus rings, loading spinners, progress indicators | Alive (shortened if long) |

### The three layers of gating

**CSS** — global safety net (0.01ms, never 0s — `animationend`/`transitionend` listeners depend on the event firing):

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

…plus **intentional tiered overrides** so Tier 3 survives (the global net alone over-removes). Elaborate effects opt in via the positive query: `@media (prefers-reduced-motion: no-preference)`.

**JS** — CSS can't see timelines, canvas, or WebGL:

```js
const REDUCE = "(prefers-reduced-motion: reduce)";
export const prefersReducedMotion = () =>
  typeof window !== "undefined" && window.matchMedia?.(REDUCE).matches;
export function onReducedMotionChange(cb) {        // users toggle without reloading
  const mq = window.matchMedia(REDUCE);
  const h = (e) => cb(e.matches);
  mq.addEventListener("change", h);
  return () => mq.removeEventListener("change", h);
}
```

**Libraries**: GSAP → `gsap.matchMedia().add({ motionOK, motionReduce }, ...)` (reverts conditional setups on change); Motion → `<MotionConfig reducedMotion="user">` (kills transform/layout animations app-wide, one line); Lenis & smooth-scroll libs → never instantiate; React hook below.

```tsx
export function useReducedMotion() {
  const [reduced, setReduced] = useState(false);   // SSR-safe: motion default, sync in effect
  useEffect(() => {
    const mq = window.matchMedia("(prefers-reduced-motion: reduce)");
    const on = () => setReduced(mq.matches);
    on(); mq.addEventListener("change", on);
    return () => mq.removeEventListener("change", on);
  }, []);
  return reduced;
}
```

Other a11y invariants: focus indicators never hidden by animations; animated text meets contrast at rest (transition frames are exempt, the endpoints aren't); auto-motion > 5s has a visible pause (WCAG 2.2.2 Level A — the media query does not exempt you); motion never carries state the DOM doesn't (keyboard gets the same information).

## 2. The 60fps budget

The render pipeline is **layout → paint → composite**. Per frame you may only touch the last one:

| Animating | Cost | Replace with |
| --- | --- | --- |
| `width` / `height` | layout + paint | `scaleX/Y` (visual) · FLIP (real size) · grid `0fr→1fr` |
| `top` / `left` / `margin` | layout + paint | `transform: translate()` |
| `box-shadow` | paint (large blur region) | `::after` holding the shadow, animate its `opacity` |
| `filter: blur()` | paint (heavy) | cross-fade two layers via `opacity` |
| `background-position` | paint | `translate()` on a child layer |
| `color` / `background-color` | paint | usually fine (short, small areas) |
| **`transform`, `opacity`** | **composite only** | — the whole budget |

Supporting disciplines:

- **Batch reads, then writes.** Every read (`offsetWidth`, `getBoundingClientRect`, `getComputedStyle`) after a write forces sync reflow; interleave them in a loop and you have layout thrashing. Read all → mutate all.
- **`will-change: transform` just-in-time, removed when idle.** It pre-promotes a layer; blanket-applying it to many elements costs GPU memory and *hurts*. One `translateZ(0)` hack does the same but can't be undone cleanly — prefer `will-change`.
- **One transform per element** — compose `translate + scale + rotate` in a single `transform` (or independent compositor-friendly `translate`/`scale`/`rotate` properties); two systems fighting over `style.transform` (e.g. Motion + a CSS transition) double-animate.
- **Verify compositor-only**: DevTools → Performance record → frames show **no purple Layout / green Paint bands** during the animation window. Or headless trace and assert no `Layout`/`Paint` events.
- Test throttled: 4× CPU slowdown in DevTools is the floor, not the ceiling.

## 3. Layout stability (CLS = 0)

- Reveal animations never change layout: the element occupies final space from the first frame (`opacity`/`transform` from hidden states — the container reserves size).
- `@starting-style` entrances, skeletons with final dimensions, and `min-height` reservations for late-loading content.
- Late fonts: `size-adjust` / metric overrides so swap doesn't reflow animated text.
- Measure: Lighthouse CLS — animation-caused shifts count like any other.

## 4. The verification loop (freeze, screenshot, both paths)

"It runs" is not "it ships". Every animated deliverable gets a **freeze harness** — a way to land on a deterministic state for screenshots:

```html
<script>
  const q = new URLSearchParams(location.search);
  const reduce = q.get("reduce") === "1" || matchMedia("(prefers-reduced-motion: reduce)").matches;
  const t = q.get("t"), s = q.get("state");
  // build motion conditionally on `reduce`; apply state `s`;
  // freeze time: tl.pause(); tl.seek(+t)          (GSAP timeline)
  // anim.currentTime = +t * 1000                  (WAAPI)
  // el.style.animationDelay = -t + "s"; el.style.animationPlayState = "paused"  (CSS)
  // svg.pauseAnimations(); svg.setCurrentTime(+t) (SMIL)
  // dl.setFrame(+t)                               (Lottie)
  window.__ready = true;
</script>
```

```bash
npx playwright screenshot --wait-for-timeout=500 "file://$PWD/demo.html?t=0.6" normal.png
npx playwright screenshot --reduced-motion=reduce --wait-for-timeout=500 \
  "file://$PWD/demo.html?t=0.6&reduce=1" reduced.png
```

Check the pair: normal reads intentional (easing visible mid-frame, stagger feathering, nothing clipped); reduced shows Tier 1 gone, Tier 2 as short fades, Tier 3 alive — nothing teleporting, no dead spinners.

## 5. The motion quality gate (ship / no-ship)

Answer every line before calling motion done:

| # | Gate | Pass condition |
| --- | --- | --- |
| 1 | Purpose | Every animation names one of the five purposes (craft.md §1); slop list clean |
| 2 | Durations | Within bands; nothing user-waits-on > 500ms; exits shorter than enters |
| 3 | Easing | Chosen from the library, never accidental `linear`; enter ≠ exit families |
| 4 | Composite-only | Per-frame property audit: `transform`/`opacity` (recipes' cross-fade tricks sanctioned); DevTools shows no per-frame Layout/Paint |
| 5 | Reduced motion | Tiered in CSS **and** JS-gated for timelines/canvas; live-toggle listener where feasible; pause controls for > 5s autoplay |
| 6 | Stability | CLS 0; skeletons reserve; no reveal changes layout |
| 7 | State integrity | Keyboard/screen-reader parity; focus never animated away; state never lives only in the animation |
| 8 | Budget | One primary motion at a time; motion budget declared and cut to; one marquee max; signature moment ≤ 1 |
| 9 | Verified | Freeze-frame screenshots exist for start/mid/end, both motion paths; pinned scenes end within 2 viewports |
| 10 | Cleanup | `will-change` removed when idle; rAF loops pause offscreen; ScrollTriggers/observers/intersection observers disposed; WebGL resources disposed |

A failed gate is a fix, not a debate. Two failed gates in the same category means the implementation choice itself is wrong — step back to the decision tree in `tech.md`.
