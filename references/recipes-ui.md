# UI motion recipes — reveals, micro-interactions, transitions, loaders

> Merged and adapted from web-animation-skills `micro-interaction` / `page-transition-animation` (iart-ai, MIT), genjutsu `css-native` / `framer-motion` (AThevon, MIT), motion-dev-animations-skill (199-biotechnologies, MIT), greensock gsap-skills (MIT). Full credits in ATTRIBUTION.md.

Every recipe ships three things: **vanilla** (zero-dependency), **React** (Motion / `motion/react`), and the two guard lines — reduced motion and perf. Durations/easings reference the tokens in `craft.md §7`. Values are starting points calibrated to the duration bands; adjust to the product, not to taste alone.

## A. Reveals

### 1. Fade-up on scroll into view

The workhorse. One axis of movement + fade, 300–400ms, `--ease-enter`.

**Vanilla** (progressive enhancement: content visible if JS fails):

```html
<div class="reveal">...</div>
```

```css
.reveal { opacity: 1; }
@media (prefers-reduced-motion: no-preference) {
  .reveal { opacity: 0; translate: 0 1.5rem;
            transition: opacity 400ms var(--ease-enter), translate 400ms var(--ease-enter); }
  .reveal.in { opacity: 1; translate: 0 0; }
}
```

```js
const io = new IntersectionObserver((es) => {
  for (const e of es) if (e.isIntersecting) { e.target.classList.add("in"); io.unobserve(e.target); }
}, { threshold: 0.2 });
document.querySelectorAll(".reveal").forEach((el) => io.observe(el));
```

**React**:

```tsx
<motion.div initial={{ opacity: 0, y: 24 }} whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, amount: 0.2 }}
  transition={{ duration: 0.4, ease: [0.16, 1, 0.3, 1] }}>
```

- **Reduced motion**: content is visible by default (vanilla) / Motion's `MotionConfig reducedMotion="user"` or `initial={reduced ? {opacity:0} : ...}` with a 150ms fade.
- **Perf**: `opacity` + `translate` are composite-only; IO fires once, then unobserves.
- CSS-only alternative where supported: `animation-timeline: view()` inside `@supports` — zero JS, but gate it and keep content visible by default.

### 2. Staggered grid / list reveal

40–60ms per item, direction = meaning (forward for lists, center-out for grids). Total ≤ 400ms.

**Vanilla** (sibling-index, no JS):

```css
@media (prefers-reduced-motion: no-preference) {
  .stagger > * { opacity: 0; animation: item-in 400ms var(--ease-enter) both;
                 animation-delay: calc(sibling-index() * 50ms); }
}
@keyframes item-in { from { opacity: 0; translate: 0 1rem; } to { opacity: 1; translate: 0 0; } }
```

**React** (variants propagate):

```tsx
const list = { hidden: {}, show: { transition: { delayChildren: stagger(0.05) } } };
const item = { hidden: { opacity: 0, y: 12 }, show: { opacity: 1, y: 0 } };
<motion.ul variants={list} initial="hidden" whileInView="show" viewport={{ once: true }}>
  {items.map((i) => <motion.li key={i.id} variants={item} />)}
</motion.ul>
```

- **Reduced motion**: whole block under `no-preference` query; items render visible.
- **Perf**: hundreds of items → cap the stagger (`from: "center"`) or reveal in chunks (`ScrollTrigger.batch` for mass reveals).

### 3. Clip-path wipe (image/banner reveal)

The curtain. Content reveals edge-to-edge — horizontal for wide media, vertical for tall.

```css
.wipe { clip-path: inset(0 100% 0 0); }        /* hidden */
.wipe.in { clip-path: inset(0 0 0 0);
           transition: clip-path 600ms cubic-bezier(0.77, 0, 0.175, 1); }
```

```tsx
<motion.div initial={{ clipPath: "inset(0 100% 0 0)" }} whileInView={{ clipPath: "inset(0 0% 0 0)" }}
  viewport={{ once: true }} transition={{ duration: 0.6, ease: [0.77, 0, 0.175, 1] }} />
```

- **Reduced motion**: show final state, no wipe.
- **Perf**: `clip-path` is paint-only (no layout) — heavier than transform but safe at 600ms; pair with a scale 1.08→1.00 counter-zoom on the inner image to fake parallax.

### 4. Blur-in text reveal

Premium editorial feel: text resolves into focus while rising slightly.

```css
@media (prefers-reduced-motion: no-preference) {
  .blur-in { animation: blur-in 500ms var(--ease-enter) both; }
}
@keyframes blur-in {
  from { opacity: 0; filter: blur(8px); translate: 0 0.5rem; }
  to   { opacity: 1; filter: blur(0);   translate: 0 0; }
}
```

```tsx
<motion.h1 initial={{ opacity: 0, filter: "blur(8px)", y: 8 }}
  animate={{ opacity: 1, filter: "blur(0px)", y: 0 }}
  transition={{ duration: 0.5, ease: [0.16, 1, 0.3, 1] }} />
```

- **Reduced motion**: plain fade, no blur, ≤200ms.
- **Perf**: `filter` animates paint — fine for one hero headline, never for body text or many elements.

### 5. Split-text character reveal

Headline theater: characters rise with a small stagger (20–30ms). GSAP SplitText (free since 3.13) or hand-rolled:

**Vanilla/GSAP**:

```js
const split = new SplitText("h1", { type: "chars" });
gsap.from(split.chars, { yPercent: 100, opacity: 0, stagger: 0.025,
                         duration: 0.6, ease: "power3.out" });
```

**React, no dependency** (aria-safe):

```tsx
function SplitReveal({ text }: { text: string }) {
  const reduced = useReducedMotion();
  return (
    <h1 aria-label={text}>
      {text.split("").map((c, i) => (
        <motion.span key={i} aria-hidden style={{ display: "inline-block" }}
          initial={reduced ? { opacity: 0 } : { y: "0.6em", opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          transition={{ delay: i * 0.024, duration: 0.5, ease: [0.16, 1, 0.3, 1] }}>
          {c === " " ? "\u00A0" : c}
        </motion.span>
      ))}
    </h1>
  );
}
```

- **Reduced motion**: whole-word fade instead of per-char.
- **Perf**: wraps `aria-hidden` chars inside an `aria-label` heading so screen readers hear the sentence, not the letters.

### 6. Masked image reveal (curtain + counter-zoom)

Image scales down from 1.15 as its mask opens — perceived parallax with zero scroll listeners:

```css
.masked { overflow: clip; }
.masked img { scale: 1.15; }
.masked.in img { scale: 1; transition: scale 800ms var(--ease-enter); }
.masked.in { clip-path: inset(0); transition: clip-path 800ms var(--ease-enter); }
```

- **Reduced motion**: image at `scale: 1`, visible.
- **Perf**: both properties are compositor/paint-safe; one transition, no rAF.

## B. Micro-interactions

### 7. Button press feedback

The tactile floor: compress on press, spring back on release.

**Vanilla**:

```css
.btn { transition: scale 120ms var(--ease-snappy); }
.btn:active { scale: 0.96; }
@media (prefers-reduced-motion: reduce) { .btn { transition-duration: 0.01ms; } }
```

**React**:

```tsx
<motion.button whileTap={{ scale: 0.96 }}
  transition={{ type: "spring", stiffness: 500, damping: 30 }} />
```

- Instant — no delay between finger and reaction. Never animate position here, only scale.

### 8. Card hover lift (cheap shadow)

Hover shadow animates as **pseudo-element opacity**, not `box-shadow`:

```css
.card { position: relative; transition: translate 200ms var(--ease-snappy); }
.card:hover { translate: 0 -4px; }
.card::after {
  content: ""; position: absolute; inset: 0; border-radius: inherit;
  box-shadow: 0 12px 28px rgb(0 0 0 / 0.18); opacity: 0;
  transition: opacity 200ms var(--ease-snappy); pointer-events: none;
}
.card:hover::after { opacity: 1; }
```

```tsx
<motion.div whileHover={{ y: -4 }} transition={{ type: "spring", stiffness: 400, damping: 30 }}
  className="card" />   /* shadow via the same ::after trick in CSS */
```

- The blurred shadow rasterizes once; hover only flips compositor opacity.
- Motion stays under the cursor: lift ≤ 6px, never horizontal drift on click targets.

### 9. Magnetic button (pointer driver)

Element leans toward the cursor, max ~8px, springs home on leave.

**Vanilla** (rAF-throttled):

```js
const btn = document.querySelector(".magnetic");
btn.addEventListener("pointermove", (e) => {
  const r = btn.getBoundingClientRect();
  const x = (e.clientX - r.left - r.width / 2) * 0.25;   // 25% of offset, capped by design
  const y = (e.clientY - r.top - r.height / 2) * 0.25;
  btn.style.translate = `${x}px ${y}px`;
});
btn.addEventListener("pointerleave", () => { btn.style.translate = "0 0"; });
```

```css
.magnetic { transition: translate 300ms var(--ease-overshoot); }
```

**React**:

```tsx
const x = useMotionValue(0), y = useMotionValue(0);
const sx = useSpring(x, { stiffness: 200, damping: 15 }), sy = useSpring(y, { stiffness: 200, damping: 15 });
<motion.button style={{ x: sx, y: sy }}
  onPointerMove={(e) => { const r = e.currentTarget.getBoundingClientRect();
    x.set((e.clientX - r.left - r.width / 2) * 0.25); y.set((e.clientY - r.top - r.height / 2) * 0.25); }}
  onPointerLeave={() => { x.set(0); y.set(0); }} />
```

- **Reduced motion**: disable entirely — this is pointer-tracking, Tier 1.
- **Perf**: motion values write straight to the DOM; no re-render per move.

### 10. Animated toggle/switch

Thumb travels 100% with overshoot; track color crossfades:

```css
.switch { inline-size: 44px; block-size: 24px; border-radius: 999px;
          background: #ccc; transition: background 200ms var(--ease-snappy); }
.switch[aria-checked="true"] { background: var(--accent); }
.thumb { block-size: 20px; inline-size: 20px; border-radius: 50%; background: white;
         translate: 2px 0; transition: translate 250ms var(--ease-overshoot); }
.switch[aria-checked="true"] .thumb { translate: 22px 0; }
```

```tsx
<motion.button role="switch" aria-checked={on}
  animate={{ backgroundColor: on ? accent : "#ccc" }}>
  <motion.span animate={{ x: on ? 22 : 0 }}
    transition={{ type: "spring", stiffness: 500, damping: 30 }} />
</motion.button>
```

- Keyboard-operable (`role="switch"`, space toggles) — the animation never carries the state.

### 11. Like / heart burst

Press → scale punch + particles once (signature micro-moment, rationed):

```tsx
<motion.button whileTap={{ scale: 1.4 }} animate={{ scale: filled ? 1 : 0.8 }} onClick={toggle}
  transition={{ type: "spring", stiffness: 500, damping: 12 }}>
  <AnimatePresence>{filled && <Particles key="p" />}</AnimatePresence>
</motion.button>
```

Particles: 6–8 dots, `animate={{ x: rand, y: rand, opacity: 0 }}`, 400ms, fire once. Under reduced motion: color/state change only, no burst.

## C. Transitions & containers

### 12. Toast (enter, auto-exit, stacked)

```css
.toast { transition: opacity 200ms var(--ease-enter), translate 200ms var(--ease-enter),
                      display 200ms allow-discrete; }
@starting-style { .toast { opacity: 0; translate: 0 1rem; } }
```

```tsx
<AnimatePresence mode="popLayout">           /* exiting toasts stop holding space */
  {toasts.map((t) => (
    <motion.li key={t.id} layout
      initial={{ opacity: 0, y: 16 }} animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, scale: 0.95 }} transition={{ duration: 0.2 }} />
  ))}
</AnimatePresence>
```

- **Reduced motion**: fade only. **Perf**: `popLayout` + `layout` animates removal with transforms.

### 13. Accordion (height: auto, zero JS measurement)

```css
.acc { display: grid; grid-template-rows: 0fr; transition: grid-template-rows 300ms var(--ease-snappy); }
.acc.open { grid-template-rows: 1fr; }
.acc > .content { overflow: clip; min-height: 0; }
```

```tsx
<motion.div animate={{ height: open ? "auto" : 0 }} style={{ overflow: "clip" }}
  transition={{ duration: 0.3, ease: [0.2, 0, 0, 1] }} />
```

- Chromium can also do it natively: `interpolate-size: allow-keywords` + `transition: height`.
- One accordion open at a time = one director rule.

### 14. Modal / dialog (native, animated both ways)

```html
<dialog class="modal">...</dialog>
```

```css
.modal { opacity: 1; scale: 1;
  transition: opacity 250ms var(--ease-enter), scale 250ms var(--ease-enter),
              display 250ms allow-discrete, overlay 250ms allow-discrete; }
.modal[open] { /* final state */ }
@starting-style { .modal { opacity: 0; scale: 0.96; } }   /* never scale(0) */
```

- `<dialog>` gives focus trap, Escape, top layer — free. `overlay` in the transition list animates the top-layer entry.
- React equivalent: `AnimatePresence` + `motion.div` with `initial={{opacity:0, scale:0.96}}`, `exit={{opacity:0, scale:0.98}}` (exit shorter/subtler — asymmetry law).

### 15. Page transition (Next.js App Router)

The classic failure — `AnimatePresence` exit never fires because the router unmounts the old route instantly. Fix: `template.tsx` + pathname key + **FrozenRouter** (freezes router context so the outgoing tree keeps rendering its own content during exit):

```tsx
// app/template.tsx
<AnimatePresence mode="wait" initial={false}>
  <PageTransition key={pathname}>       {/* key = usePathname() */}
    <FrozenRouter>{children}</FrozenRouter>
  </PageTransition>
</AnimatePresence>

// page-transition.tsx
<motion.div initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -12 }} transition={{ duration: 0.3, ease: [0.22, 1, 0.36, 1] }} />
```

```tsx
// frozen-router.tsx — snapshot LayoutRouterContext
export function FrozenRouter({ children }) {
  const context = useContext(LayoutRouterContext ?? {});
  const frozen = useRef(context).current;
  return <LayoutRouterContext.Provider value={frozen}>{children}</LayoutRouterContext.Provider>;
}
```

Native alternative (no Motion): the **View Transitions API** — `startViewTransition()` on navigation, or `next-view-transitions` for MPA-style. `mode="wait"` = exit completes first; default mode = crossfade.

### 16. Skeleton shimmer → content (state continuity)

Skeleton crossfades into real content — the swap itself is the animation, so no jump cut:

```css
.skeleton { background: linear-gradient(90deg, #eee 25%, #f5f5f5 50%, #eee 75%);
            background-size: 200% 100%; animation: shimmer 1.4s linear infinite; }
@keyframes shimmer { to { background-position: -200% 0; } }
```

```tsx
<AnimatePresence mode="wait">
  {loading ? <motion.div key="sk" className="skeleton" exit={{ opacity: 0 }} />
            : <motion.div key="content" initial={{ opacity: 0 }} animate={{ opacity: 1 }} />}
</AnimatePresence>
```

- **Reduced motion**: static skeleton (no shimmer loop — 2.2.2 territory), plain swap fade.
- **Perf**: shimmer is a paint animation on one element — acceptable for loading states, which are essential feedback (Tier 3).

### 17. Spinner → success morph (one object, one story)

The button keeps its identity; its content morphs spinner → checkmark:

```tsx
<motion.button disabled={state !== "idle"} onClick={submit}>
  <AnimatePresence mode="wait" initial={false}>
    {state === "idle"   && <motion.span key="i" exit={{opacity: 0}}>Send</motion.span>}
    {state === "loading" && <motion.span key="l" className="spinner" initial={{opacity:0}} exit={{opacity:0}} />}
    {state === "done"   && <motion.svg key="d" viewBox="0 0 24 24" initial={{opacity:0, scale:0.8}}
        animate={{opacity:1, scale:1}}><motion.path d="M5 13l4 4L19 7" strokeWidth={2.5}
        fill="none" stroke="currentColor" strokeLinecap="round"
        initial={{ pathLength: 0 }} animate={{ pathLength: 1 }} transition={{ duration: 0.3 }} /></motion.svg>}
  </AnimatePresence>
</motion.button>
```

The checkmark **draws itself** (`pathLength` 0→1). Under reduced motion: states swap with a 150ms fade, no draw.
