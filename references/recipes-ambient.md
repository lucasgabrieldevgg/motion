# Ambient & scroll motion recipes — progress, parallax, backgrounds, cursors, text, celebration

> Merged and adapted from genjutsu `css-native` / `gsap` / `framer-motion` / `canvas-generative` (AThevon, MIT), web-animation-skills `accessible-animation` / `60fps-animation` (iart-ai, MIT), greensock gsap-skills (MIT). Full credits in ATTRIBUTION.md.

Ambient motion is scroll-linked, pointer-linked, or autonomous — never a user-waits-on interaction, so different laws apply: progress-based (no fixed duration), below-conscious cycles for loops, and **Tier-1 vestibular sensitivity at maximum** (large-area movement is the first thing removed under reduced motion).

## A. Scroll motion

### 18. Reading progress bar

```css
/* Zero-JS where supported */
.progress { position: fixed; inset: 0 0 auto 0; height: 3px; background: var(--accent);
            transform-origin: left; transform: scaleX(0); }
@supports (animation-timeline: scroll()) {
  .progress { animation: grow linear both; animation-timeline: scroll(root block); }
}
@keyframes grow { to { transform: scaleX(1); } }
```

```tsx
const { scrollYProgress } = useScroll();
const scaleX = useSpring(scrollYProgress, { stiffness: 120, damping: 24 });
<motion.div className="progress" style={{ scaleX }} />
```

- **Reduced motion**: progress indicator itself is orientation, not motion — keep it, drop the spring (map 1:1). The CSS version is scrub-driven and already linear.
- **Perf**: `scaleX` only; the spring smoothing avoids the raw scrollbar jitter.

### 19. Parallax layers (depth without disease)

Two layers, small differential, never on reading text:

```css
@supports (animation-timeline: scroll()) {
  .layer-bg  { animation: drift linear both; animation-timeline: view(); animation-range: cover; }
  .layer-fg  { animation: drift-fg linear both; animation-timeline: view(); animation-range: cover; }
}
@keyframes drift   { to { translate: 0 8%; } }    /* background slower: moves less */
@keyframes drift-fg { to { translate: 0 -8%; } }
```

```tsx
const { scrollYProgress } = useScroll({ target: ref, offset: ["start end", "end start"] });
const yBg = useTransform(scrollYProgress, [0, 1], ["0%", "12%"]);
const yFg = useTransform(scrollYProgress, [0, 1], ["0%", "-8%"]);
```

- **Reduced motion: remove entirely (Tier 1)** — `translate: none` under the media query; the composition must survive flat.
- **Perf**: transforms only; differential ≤ 15% of section height or it reads as a funhouse.

### 20. Horizontal scroll section (GSAP)

The pinned panel gallery — the one pattern CSS can't do alone:

```js
const panels = gsap.utils.toArray(".panel");
const scrollTween = gsap.to(".track", {
  x: () => -(track.scrollWidth - window.innerWidth),
  ease: "none",                                  // NEVER ease the container
  scrollTrigger: { trigger: ".wrapper", pin: true, scrub: 1,
                   end: () => "+=" + track.scrollWidth },
});
// elements INSIDE react to horizontal position:
gsap.to(".panel-art", { scale: 1.15, scrollTrigger: { trigger: ".panel-art",
  containerAnimation: scrollTween, start: "left center", end: "right center", scrub: true } });
```

- **Reduced motion**: skip the pin entirely — render a normal vertically-scrolling grid, or a horizontally scrollable `overflow-x` region the user drives.
- **Perf**: ` scrub: 1` smooths 1s behind the scrollbar; test on a 4×-throttled CPU; `invalidateOnRefresh: true` + function values for resize.

### 21. Pinned scrub scene (the cinematic tell)

One sticky moment max per page — declare it in the motion budget:

```js
gsap.timeline({ scrollTrigger: { trigger: ".scene", pin: true, scrub: 1, start: "top top",
                                 end: "+=150%" } })
  .fromTo(".s1", { opacity: 1 }, { opacity: 0 })
  .fromTo(".s2", { opacity: 0, y: 40 }, { opacity: 1, y: 0 }, "<")
  .to(".badge", { scale: 1.2, rotate: 8 }, "<0.2");
```

Rules: total pin length ≤ 2 viewports (users feel hijack past that); content readable at every scrub position (it can pause anywhere); reduced motion = static final composition, no pin.

## B. Numbers & text

### 22. Count-up

```tsx
const ref = useRef<HTMLSpanElement>(null);
const inView = useInView(ref, { once: true });
const reduced = useReducedMotion();
useEffect(() => {
  if (!inView) return;
  const controls = animate(0, value, {
    duration: reduced ? 0 : 1.2, ease: [0.16, 1, 0.3, 1],
    onUpdate: (v) => { if (ref.current) ref.current.textContent = Math.round(v).toLocaleString(); },
  });
  return () => controls.stop();
}, [inView, value]);
<span ref={ref} />;
```

- Vanilla: WAAPI on a registered `@property` (`--num`) with `counter-reset: var(--num)` — or a small rAF tween writing `textContent`.
- **Perf/A11y**: final value is real text (SEO + screen readers), animation is cosmetic. Reduced motion: instant.

### 23. Marquee (one per page, pauseable)

```css
.marquee { overflow: clip; }
.marquee-track { display: flex; gap: 2rem; width: max-content;
  animation: marquee var(--speed, 22s) linear infinite; }
@keyframes marquee { to { translate: calc(-50% - 1rem) 0; } }   /* duplicated content */
.marquee:hover .marquee-track, .marquee:focus-within .marquee-track { animation-play-state: paused; }
@media (prefers-reduced-motion: reduce) { .marquee-track { animation: none; } }
```

- Content duplicated exactly (seamless loop = `-50%` shift). Hover **and** `focus-within` pause it (keyboard users too); if it's ever unhoverable, add a real pause button (2.2.2). Linear is correct here — scrub-logic.
- React/Motion: `x: ["0%", "-50%"]` loop, same pause rules.

### 24. Text scramble (decode effect)

```tsx
const CHARS = "!<>-_\\/[]{}—=+*^?#";
function Scramble({ text }: { text: string }) {
  const [out, setOut] = useState(text);
  const reduced = useReducedMotion();
  useEffect(() => {
    if (reduced) return setOut(text);
    let frame = 0;
    const id = setInterval(() => {
      frame++;
      setOut(text.split("").map((c, i) =>
        i < frame / 2 ? c : CHARS[Math.random() * CHARS.length | 0]).join(""));
      if (frame / 2 >= text.length) clearInterval(id);
    }, 30);
    return () => clearInterval(id);
  }, [text, reduced]);
  return <span aria-label={text} aria-hidden>{out}</span>;
}
```

- Rationed to one hero moment. **Reduced motion**: plain text, immediately.

### 25. Typewriter

```tsx
const [n, setN] = useState(0);
useEffect(() => {
  if (reduced) { setN(text.length); return; }
  const id = setInterval(() => setN((v) => (v >= text.length ? (clearInterval(id), v) : v + 1)), 28);
  return () => clearInterval(id);
}, [text]);
<span>{text.slice(0, n)}<motion.span animate={{ opacity: [1, 0] }}
  transition={{ repeat: Infinity, duration: 0.8 }} className="caret" /></span>
```

- Blinking caret stops after attention settles (~3s then hide). Reduced: full text, static.

## C. Backgrounds & ambient

### 26. Animated gradient / aurora background

```css
.aurora { position: absolute; inset: -50%; filter: blur(60px); opacity: 0.35;
  background:
    radial-gradient(40% 40% at 30% 30%, oklch(0.7 0.15 250 / 0.8), transparent),
    radial-gradient(40% 40% at 70% 60%, oklch(0.65 0.18 310 / 0.8), transparent),
    radial-gradient(40% 40% at 50% 80%, oklch(0.7 0.12 170 / 0.8), transparent);
  animation: aurora 26s ease-in-out infinite alternate; }
@keyframes aurora { to { rotate: 360deg; scale: 1.15; } }
@media (prefers-reduced-motion: reduce) { .aurora { animation: none; } }
```

- Cycle ≥ 20s (below attention), static under reduce, `filter: blur` set once (not animated), and **never your only color story** — slop list #9 if it's the whole design.

### 27. Canvas particles (DPR-aware, budgeted, pausable)

```js
const ctx = setupCanvas(canvas, w, h);          // tech.md §6 setup
const N = Math.min(120, (w * h) / 12000);       // density budget, not ambition
const particles = Array.from({ length: N }, () => spawn());
let visible = true, raf, last;
new IntersectionObserver(([e]) => { visible = e.isIntersecting; }).observe(canvas);

function frame(t) {
  if (visible) {
    const dt = Math.min((t - last) / 1000, 0.05); last = t;
    ctx.clearRect(0, 0, w, h);
    for (const p of particles) { step(p, dt); draw(p); }
  }
  raf = requestAnimationFrame(frame);           // loop continues; work pauses
}
if (!matchMedia("(prefers-reduced-motion: reduce)").matches) {
  raf = requestAnimationFrame((t) => { last = t; frame(t); });
} else { drawStaticFrame(); }                   // one beautiful still
```

- Reduced motion = **one static frame** (the loop never starts). Loop pauses offscreen. Interaction-optional: particles drift; pointer adds gentle repulsion at most.

### 28. Custom cursor / cursor glow (pointer driver)

```tsx
const x = useMotionValue(-100), y = useMotionValue(-100);
const sx = useSpring(x, { stiffness: 300, damping: 28 }), sy = useSpring(y, { stiffness: 300, damping: 28 });
useEffect(() => {
  const move = (e: PointerEvent) => { x.set(e.clientX); y.set(e.clientY); };
  window.addEventListener("pointermove", move, { passive: true });
  return () => window.removeEventListener("pointermove", move);
}, []);
<motion.div className="cursor" style={{ x: sx, y: sy }} aria-hidden />;
```

- **Reduced motion: disable the whole feature** — pointer-following is Tier 1 (implies self-motion).
- **Perf**: spring-smoothed motion values (no re-renders); GSAP path is `quickTo`. Never hide the native cursor without a fully-functional replacement (focus visibility is non-negotiable).

### 29. 3D tilt card

```tsx
const rx = useMotionValue(0), ry = useMotionValue(0);
const srx = useSpring(rx, { stiffness: 300, damping: 25 }), sry = useSpring(ry, { stiffness: 300, damping: 25 });
<div style={{ perspective: 800 }}>
  <motion.div style={{ rotateX: srx, rotateY: sry, transformStyle: "preserve-3d" }}
    onPointerMove={(e) => {
      const r = e.currentTarget.getBoundingClientRect();
      ry.set(((e.clientX - r.left) / r.width - 0.5) * 10);      // ≤ 10deg
      rx.set(-((e.clientY - r.top) / r.height - 0.5) * 10);
    }}
    onPointerLeave={() => { rx.set(0); ry.set(0); }}>
    <div style={{ transform: "translateZ(30px)" }}>Content floats</div>
  </motion.div>
</div>
```

- Reduced motion: disable (large-area 3D is Tier 1). Vanilla: CSS `perspective` + `pointermove` writing `--rx/--ry` custom properties into `transform: rotateX(var(--rx)) rotateY(var(--ry))`.

### 30. Confetti (celebration, once)

```tsx
function Confetti({ fire }: { fire: boolean }) {
  const reduced = useReducedMotion();
  if (reduced || !fire) return null;
  return (
    <div className="confetti" aria-hidden>
      {Array.from({ length: 80 }, (_, i) => {
        const a = Math.random() * Math.PI * 2, v = 200 + Math.random() * 300;
        return <motion.span key={i} className="piece"
          initial={{ x: 0, y: 0, opacity: 1 }}
          animate={{ x: Math.cos(a) * v, y: Math.sin(a) * v - 150, rotate: Math.random() * 720, opacity: 0 }}
          transition={{ duration: 0.9, ease: [0.16, 1, 0.3, 1] }} />;
      })}
    </div>
  );
}
```

- Fires **once** per achievement (a success moment, not a mood). Reduced motion: none — the success message carries the moment. Canvas variant for 500+ pieces.
