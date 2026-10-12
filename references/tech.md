# The technology map — eight stacks, one decision

> Merged from: genjutsu `css-native` / `gsap` / `framer-motion` / `canvas-generative` / `threejs-r3f` (AThevon, MIT), web-animation-skills `svg-animation` / `lottie-animation` / `page-transition-animation` / `60fps-animation` (iart-ai, MIT), greensock official `gsap-skills` (MIT), motion-dev-animations-skill (199-biotechnologies, MIT). Package versions live-verified 2026-10-13. Full credits in ATTRIBUTION.md.

Version snapshot (npm, 2026-10-13): `gsap` 3.15.0 · `motion` / `framer-motion` 14.1.0 (same library, both names publish the same version) · `@gsap/react` 2.1.2 · `three` 0.186.1 · `@react-three/fiber` 9.8.1 · `@lottiefiles/dotlottie-web` 0.81.0 · `@lottiefiles/dotlottie-react` 0.20.0 · `lottie-web` 5.13.0 · `@rive-app/react-canvas` 4.36.0 · `lenis` 1.3.26 · `flubber` 0.4.2. Re-verify before pinning anything.

## The decision tree

```
What does the motion need?
├─ A state changes (hover, open, toggle)          → CSS transition
├─ Element appears/disappears (display:none)       → CSS @starting-style + allow-discrete
├─ Scroll should drive it                         → CSS animation-timeline (@supports-gated), else JS
├─ Page or route changes                          → View Transitions API
├─ It must sequence 3+ steps / pin / scrub cinematically → GSAP timeline + ScrollTrigger
├─ React state drives it, or it needs exit animations / shared layout / drag → Motion
├─ Physics + interruption matter (drag release)   → Motion springs
├─ Layout itself changes (reorder, resize)        → FLIP (WAAPI) or Motion layout
├─ Icons draw / shapes morph / path following     → SVG (± GSAP MorphSVG / flubber)
├─ Thousands of particles / generative art        → Canvas 2D
├─ 3D, shaders, real lights                       → three.js (+ R3F in React)
├─ Designer made it in After Effects              → Lottie (dotLottie)
└─ Interactive animated character / state machine  → Rive
```

Bundles to know: CSS 0 KB · GSAP ~25 KB core + plugins · Motion ~30–50 KB tree-shaken · three.js (large; code-split it) · dotlottie-web small WASM. If the page has fewer than ~3 animations, the answer is CSS, full stop.

## 1. Modern CSS (the default stack)

**Transitions** for state pairs. **@keyframes** for autonomous loops and entrances. Beyond those:

### Scroll-driven animations (zero JS)

```css
/* Progress: tied to page scroll */
.progress-bar {
  animation: grow linear both;
  animation-timeline: scroll(root block);
}

/* Reveal: tied to element visibility in the scrollport */
.reveal {
  animation: fade-up linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 80%;
}
```

- `scroll(<scroller> <axis>)`: scroller `nearest | root | self`, axis `block | inline | x | y`.
- `view()` ranges: `cover | contain | entry | exit | entry-crossing | exit-crossing`, sliced with `animation-range`.
- **Gate it**: `@supports (animation-timeline: scroll())` and keep content visible by default — Firefox has no release-channel support and Safari shipped only in 26; about one visitor in six sees nothing.
- `animation-fill-mode: both` (never `forwards`) for scroll-driven work, or scrolling back can lock the end state.

### View Transitions API (page transitions, SPA and MPA)

```js
// SPA: wrap the DOM update
document.startViewTransition(() => updateContent());
```

```css
::view-transition-old(root) { animation: fade-out 200ms ease-in; }
::view-transition-new(root) { animation: fade-in 300ms ease-out; }

.hero-image { view-transition-name: hero; }        /* named elements morph */
::view-transition-group(hero) { animation-duration: 400ms; }

@view-transition { navigation: auto; }             /* MPA: cross-document, both pages */
.view-transition-class(card) { ... }               /* group styling */
```

### @starting-style + allow-discrete (enter/exit from `display: none`)

```css
.dialog {
  opacity: 1; transform: translateY(0);
  transition: opacity 300ms var(--ease-enter), transform 300ms var(--ease-enter),
              display 300ms allow-discrete;
  @starting-style { opacity: 0; transform: translateY(-1rem); }
}
.dialog[hidden] { opacity: 0; transform: translateY(-1rem); display: none; }
```

Pairs natively with `<dialog>` and `[popover]` (add `overlay` to the transition list for top-layer). Without `allow-discrete`, `display` never animates — the exit is skipped entirely.

### Also in the kit

- **Sibling stagger without JS**: `animation-delay: calc(sibling-index() * 60ms)` — handles unknown list lengths.
- **`height: auto` animation**: `interpolate-size: allow-keywords` (Chromium 129+) or the cross-browser grid trick `grid-template-rows: 0fr → 1fr` (child needs `overflow: hidden; min-height: 0`).
- **Anchor positioning** for tooltips/popovers: `anchor-name` + `position-anchor` + `position-try-fallbacks`, animated via `@starting-style`.
- **`@property`** registers custom properties so they interpolate (`--num: <number>` → count-ups in pure CSS).
- **Container queries** adapt animation to container size; `cqw` units in keyframes for container-relative motion.

### CSS do-nots

| Bad | Good | Why |
| --- | --- | --- |
| `transition: all` | list properties | `all` catches every property you add later |
| animate `width/height/top/left/margin` | `transform` / grid tricks | Layout recalcs every frame |
| scroll-driven without `@supports` | gate + visible default | 1-in-6 visitors see a wall |
| `@starting-style` without `allow-discrete` | pair them | Exit silently never runs |
| `forwards` on scroll-driven | `both` | End-state lock on scroll back |

## 2. WAAPI + rAF (zero-dependency JS)

`element.animate()` gives you a timeline object without any library:

```js
const anim = el.animate(
  [{ transform: "translateY(20px)", opacity: 0 }, { transform: "none", opacity: 1 }],
  { duration: 240, easing: "cubic-bezier(0.16, 1, 0.3, 1)", fill: "both" }
);
anim.pause(); anim.currentTime = 1200; anim.play(); anim.cancel();  // full control
```

- Runs on the compositor when only `transform`/`opacity` animate.
- `fill: "both"` + `commitStyles()` when you need the end state written to style.
- Perfect for **scrub**: `anim.currentTime = progress * anim.effect.getComputedTiming().duration`.

**FLIP** — animate layout changes with transforms only (First, Last, Invert, Play):

```js
function flip(el, mutate, duration = 300) {
  const first = el.getBoundingClientRect();
  mutate();                                   // the real DOM change
  const last = el.getBoundingClientRect();
  const dx = first.left - last.left, dy = first.top - last.top;
  const sx = first.width / last.width, sy = first.height / last.height;
  el.animate(
    [{ transformOrigin: "top left", transform: `translate(${dx}px, ${dy}px) scale(${sx}, ${sy})` },
     { transformOrigin: "top left", transform: "none" }],
    { duration, easing: "cubic-bezier(0.2, 0, 0, 1)" }
  );
}
// many elements: read ALL rects first, then mutate, then play — never interleave
```

**rAF loops** — always compute with delta time, pause when offscreen:

```js
let raf, last;
function frame(t) {
  const dt = (t - last) / 1000; last = t;
  update(dt); raf = requestAnimationFrame(frame);
}
raf = requestAnimationFrame((t) => { last = t; frame(t); });
```

- `requestAnimationFrame`, never `setTimeout`/`setInterval`, for anything visual.
- **Batch reads then writes**: every `offsetWidth`/`getBoundingClientRect()` after a style write forces synchronous reflow; interleaved loops are layout thrashing.
- Stop the loop when the canvas/section is offscreen (`IntersectionObserver`) — invisible rAF still burns battery.

## 3. GSAP (timeline, scroll, plugins)

Since 3.13 **every former Club plugin is free in the public `gsap` package** (SplitText, MorphSVG, DrawSVG, ScrollSmoother, Flip, InertiaPlugin, CustomEase...). There is no paywall anymore.

```js
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);           // always at top level
```

```js
const tl = gsap.timeline({ defaults: { duration: 0.8, ease: "power2.out" } });
tl.to(".a", { y: -20 })
  .to(".b", { y: -20 }, "<0.1")               // "<0.1" = 0.1s after previous start
  .to(".c", { y: -20 }, "<0.1");

gsap.to(".grid-item", {                        // distributed stagger
  scale: 1, stagger: { each: 0.05, from: "center", grid: "auto", axis: "x" }
});

gsap.to(".panel", {                            // pinned scrub scene
  x: "-300%", ease: "none",
  scrollTrigger: { trigger: ".container", pin: true, scrub: 1,
                   end: () => "+=" + document.querySelector(".container").scrollWidth }
});

ScrollTrigger.batch(".card", {                 // mass reveal, one trigger for many
  onEnter: (els) => gsap.to(els, { opacity: 1, y: 0, stagger: 0.1 }),
  start: "top 85%",
});
```

- **Mouse followers**: `gsap.quickTo()` reuses one tween per property instead of creating tweens per event — the difference between smooth and jank.
- **React**: `useGSAP()` from `@gsap/react` (scope + auto-cleanup), never bare `useEffect`.
- ScrollTrigger + horizontal scroll: `containerAnimation` links child triggers to the horizontal tween.
- **ScrollTrigger.refresh()** only when layout actually changes; debounce resize.

GSAP do-nots (each one is a real bug): **ease on a `containerAnimation` parent** breaks scroll mapping (always `ease: "none"`); **ScrollTrigger on a child tween inside a triggered timeline** is ignored (one trigger per timeline, or standalone tweens); **`setState`/state updates in `onUpdate`** re-render 60×/s — mutate a ref or use `quickSetter`; **`from()` after another tween** needs `immediateRender: false`; **animating `width/height`** reflows every frame — `scaleX/scaleY`, or the Flip plugin for true size.

## 4. Motion (`motion` / `framer-motion`)

Two package names, one library: `motion` re-exports `framer-motion` (both at 14.1.0). **Read `package.json` and follow what's there**; never migrate a project between the two unasked.

```tsx
import { motion, AnimatePresence, stagger } from "motion/react";   // or "framer-motion"

<AnimatePresence mode="wait">                 // exit animations need this + a stable key
  {open && (
    <motion.div key="panel"
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: 8 }}
      transition={{ duration: 0.2, ease: [0.16, 1, 0.3, 1] }} />
  )}
</AnimatePresence>
```

- **`AnimatePresence` modes**: `wait` (exit finishes first — page transitions), `sync` (crossfade), `popLayout` (removed items stop holding space — toasts/lists).
- **Variants propagate** to motion children automatically; orchestrate with `delayChildren: stagger(0.08)` — `staggerChildren` is deprecated since Motion 12.22, pass the `stagger()` function instead.
- **`layout` / `layoutId`**: auto-FLIP for reorder/resize; shared-element morph across components. Gotcha: plain `layout` distorts `borderRadius` and text during scale — counter-scale children, prefer `layout="position"`, or keep `borderRadius` as a motion value.
- **Motion values** (`useMotionValue`, `useTransform`, `useSpring`) update the DOM directly, no re-renders — the tool for scroll-linked and cursor-follow effects.
- **Scroll**: `useScroll()` / `useScroll({ target, offset: ["start end", "end start"] })` + `useTransform`.
- **`MotionConfig reducedMotion="user"`** disables transform/layout animations globally under the OS preference — one line that makes a whole app tier-aware.
- Springs: `whileTap={{ scale: 0.96 }}` + `transition={{ type: "spring", stiffness: 400, damping: 30 }}`.
- With styled-components/Emotion: `MotionConfig isValidProp={isPropValid}` or `motion.create(StyledDiv)` — motion-only props otherwise leak to the DOM.

Motion do-nots: **no `key` on AnimatePresence children** (exit never fires); **unstable `key` with `layout`** (Math.random() breaks tracking); **`setState` in `onUpdate` without a guard** (infinite re-render — use `useMotionValueEvent`); **nesting motion.divs animating the same transform axis** (conflicts — use variants to coordinate).

## 5. SVG animation

Three drivers — pick by need: **CSS** (declarative, simple), **SMIL** (self-contained, no JS bundle), **GSAP/WAAPI** (control, scrub, morph).

```css
/* Draw-on — pathLength="1" normalizes any path, no measurement: */
.path { stroke-dasharray: 1; stroke-dashoffset: 1;
        animation: draw 1.4s var(--ease-enter) forwards; }
@keyframes draw { to { stroke-dashoffset: 0; } }
```

- Measured variant when you need partial draws: `const len = path.getTotalLength()`.
- Reverse = animate offset back to `len`; direction follows point order.
- **Morphing**: `d` interpolates only between matching command structures. Mismatched shapes → **GSAP MorphSVG** (free, auto-maps points, `convertToPath` for primitives) or **flubber** (standalone, React-friendly).
- **Path following**: CSS `offset-path: path("...")` + `offset-distance` + `offset-rotate: auto` (declarative), GSAP MotionPath (control, scrub, `autoRotate`), or SMIL `<animateMotion>` (self-contained icons).
- **Authoring**: SVGO with `cleanupIds`/`removeViewBox`/`mergePaths`/`convertShapeToPath` disabled when you animate internals; inline the SVG (an `<img>` can only self-animate via SMIL/CSS inside); draw-on needs strokes, not filled outlines.
- Filters (`feDisplacementMap`, `feGaussianBlur`) are paint-heavy — animate sparingly, prefer transform/opacity.

## 6. Canvas 2D & WebGL

**Canvas 2D** for particles, flow fields, generative backgrounds:

```js
function setupCanvas(canvas, w, h) {           // DPR-aware, always
  const dpr = window.devicePixelRatio || 1;
  canvas.width = w * dpr; canvas.height = h * dpr;
  canvas.style.width = `${w}px`; canvas.style.height = `${h}px`;
  const ctx = canvas.getContext("2d");
  ctx.scale(dpr, dpr); return ctx;
}
```

- Resize via `ResizeObserver` (re-size the buffer, re-scale, redraw).
- rAF loop with delta time; **pause the whole loop offscreen** — an invisible canvas at 60fps is pure battery drain.
- Under reduced motion: draw one static frame and never start the loop.
- Particle counts are budgeted: hundreds fine, thousands need WebGL.

**three.js / R3F** when it's genuinely 3D: real lighting, shaders, or 10k+ particles. Discipline: `renderer.setAnimationLoop`; clamp `devicePixelRatio` to 2; `dispose()` geometries/materials/textures on unmount (R3F does most of this for you); gate the loop with `matchMedia` (WebGL ignores CSS entirely); code-split — `three` is the heaviest thing on this page.

## 7. Lottie & Rive (designer-authored motion)

**Lottie** — After Effects → JSON/`.lottie`, played by a runtime. Prefer **dotLottie** (`@lottiefiles/dotlottie-web` 0.81.0, canvas+WASM, 60–80% smaller than raw `.json`) over legacy `lottie-web`:

```js
const dl = new DotLottie({ canvas, src: "/hero.lottie", loop: true, autoplay: true });
dl.play(); dl.pause(); dl.setSpeed(1.5); dl.setFrame(42); dl.setSegment(30, 90);
dl.addEventListener("complete", () => { /* state machine: segment → idle loop */ });
```

- **Scroll-driven Lottie**: `autoplay: false`, map scroll progress to `setFrame(p * (total - 1))`.
- **Theming**: `.lottie` embeds named themes — `setTheme("dark")` instead of re-export per brand.
- **Discipline**: lazy-load offscreen players, pause when not visible, don't autoplay loops under reduced motion (show a static frame), keep AE comps small (size tracks keyframe/path complexity, not duration).
- React: `@lottiefiles/dotlottie-react` (0.20.0), control via `dotLottieRefCallback`.

**Rive** when you need **interactivity and state machines** (characters that respond to input, multi-state UI animations) rather than linear playback — tiny runtimes (`@rive-app/react-canvas` 4.36.0), listeners fire animations, states drive themselves. Lottie plays what happened; Rive *is* the state machine.

## 8. Smooth scroll (the honorary ninth)

Lenis (1.3.26) and friends: never instantiate under reduced motion (it is a Tier-1 vestibular trigger), always provide native fallback, and remember scroll-hijack needs an escape — pinned scenes end promptly, native scrollbar always works.
