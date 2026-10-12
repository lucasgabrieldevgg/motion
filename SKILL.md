---
name: motion
description: "Animation by code, done beautifully and professionally. A two-track skill. TRACK 1 - CRAFT (the 70%): motion as physics and choreography - duration bands (micro 100-250ms, UI 200-300ms, page 300-500ms), a named easing library with real cubic-bezier values, spring tuning (stiffness/damping presets), stagger and orchestration laws, enter/exit asymmetry, state continuity, and a motion-slop death list (perpetual loops, scale(0), linear everything, the same fade-up on every section). TRACK 2 - TOOLBOX: every web animation technology, each with working patterns - modern CSS (transitions, @keyframes, scroll-driven animation-timeline, View Transitions API, @starting-style, allow-discrete), WAAPI + requestAnimationFrame + FLIP, GSAP (timeline, ScrollTrigger, stagger distribution, SplitText, MorphSVG, Flip, quickTo), Motion / framer-motion (AnimatePresence, layout/layoutId, variants, motion values, useScroll), SVG animation (stroke draw-on, path morphing, offset-path), Canvas 2D and three.js/WebGL (DPR-aware, particles), Lottie (dotLottie) and Rive. Plus ~30 production recipes - reveals, hovers, magnetic buttons, toasts, accordions, modals, skeletons, scroll progress, parallax, marquees, count-ups, text effects, custom cursors, tilt, confetti - each in BOTH vanilla and React, each with its reduced-motion guard and perf note. Hard rules throughout: transform/opacity only per frame, tiered prefers-reduced-motion, 60fps + CLS 0, verify by freeze-frame screenshot before shipping. Use for ANY animation task: 'animate this', 'make it feel alive', 'add scroll effects', 'why is my animation janky', 'recreate this motion reference', 'build a page transition', 'fix easing', 'add Lottie/Rive', 'make a landing page cinematic' - even when the user just says 'it feels dead'."
version: 1.0.0
license: MIT for the synthesis; upstream content keeps its own licenses - see ATTRIBUTION.md
---

# motion — animation as a physical, disciplined craft

`motion` builds animation the way a motion designer at a top studio would: purpose first, physics second, technology last — and it knows exactly which of the eight web animation stacks serves each kind of movement.

## The motion contract

1. **Every animation answers a purpose** — feedback, continuity, orientation, hierarchy, or rationed delight. Motion that answers none of these is decoration and gets cut.
2. **Composite-only per frame.** Animate `transform` and `opacity`; everything else (shadow, blur, size) animates via cross-faded pseudo-elements or layers. Layout properties never animate.
3. **Duration bands are law.** Micro-interaction 100–250ms · UI transition 200–300ms · page transition 300–500ms · nothing a user waits on exceeds 500ms · scroll-scrub and ambient motion are progress-based (no duration).
4. **Enter eases out, exit eases in, exit is shorter.** Elements decelerate into place and accelerate away. An entry with `ease-in` feels broken; a 400ms exit feels sluggish.
5. **`prefers-reduced-motion` honored as tiers, in every deliverable** — Tier 1 (vestibular) removed, Tier 2 (movement) softened to ≤200ms fades, Tier 3 (opacity/color/feedback) kept. JS-driven motion is gated via `matchMedia` too; CSS alone never covers canvas or timelines.
6. **Never ship unverified motion.** Freeze-frame screenshots of start/mid/end states — including the reduced-motion path — before calling anything done.

## Routing table

Read only what the task needs:

| When | Read |
| --- | --- |
| Choosing durations, easing, springs, choreography, motion tokens — or cutting motion slop | `references/craft.md` |
| Choosing the technology, or using a specific stack (CSS/WAAPI/GSAP/Motion/SVG/Canvas/WebGL/Lottie/Rive) | `references/tech.md` |
| Building UI motion: reveals, hovers, micro-interactions, transitions, loaders | `references/recipes-ui.md` |
| Building ambient/scroll motion: parallax, progress, backgrounds, cursors, text effects, confetti | `references/recipes-ambient.md` |
| Gating accessibility, performance budgets, and the pre-ship verification loop | `references/rules.md` |

## The one-glance technology matrix

| Need | First call |
| --- | --- |
| Hover, toggle, simple state change | CSS `transition` |
| Enter from `display: none`, dialogs, popovers | CSS `@starting-style` + `allow-discrete` |
| Scroll-linked reveal / progress / parallax | CSS `animation-timeline` (in `@supports`), else JS |
| Page transitions (SPA or MPA) | View Transitions API |
| Sequenced timeline, cinematic scroll, pinned scenes | GSAP + ScrollTrigger |
| React UI state, exit animations, shared layout, drag | Motion (`motion` / `framer-motion`) |
| Interruptible, physics-based UI motion | Motion springs |
| Layout change animation (reorder, resize, move) | FLIP (WAAPI) or Motion `layout` |
| Icon draw-on, shape morph, path following | SVG techniques (± GSAP MorphSVG / flubber) |
| Particles, generative backgrounds, thousands of movers | Canvas 2D (simple) / three.js (3D, shaders) |
| Designer-authored After Effects animation | Lottie (dotLottie) |
| Interactive animated characters / state machines | Rive |

Rule of thumb: if one `@keyframes` plus one `animation-timeline` can express it, stay in CSS. The moment you need imperative control, sequencing, or runtime values, reach for a library — and pick the one whose strength matches the job, not the one you know best.

## The loop in one breath

**Intent** (what purpose does this motion serve? which of the five?) → **Choose** (technology matrix; free before paid, native before library) → **Choreograph** (duration band, easing family, stagger law, enter/exit asymmetry — on paper if the scene is complex) → **Implement** (recipe as starting point, tokens for values) → **Guard** (reduced-motion tiers, composite-only audit) → **Verify** (freeze-frame screenshots, both motion paths; DevTools shows no per-frame Layout/Paint).

## Hard rules

- Reduced motion is tiered, never a kill switch — teleporting elements is also an accessibility failure. Fades and focus rings survive.
- Auto-playing motion longer than 5s needs a visible pause control (WCAG 2.2.2, Level A) — regardless of the media query.
- Smooth-scroll libraries (Lenis & co.) are not instantiated at all under reduced motion.
- A page gets a motion budget: declare how many animated moments it has; one primary motion at a time, supporting motion yields.
- Verification is part of the deliverable: `?t=` / `?state=` freeze harness + screenshots, normal AND reduced path. "It runs" is not "it ships".

## Out of scope

- AI-generated video assets and sprite-atlas pipelines (a media-production problem, not UI motion).
- Whole-site art direction and design systems (the sibling design skills' job) — this skill animates what they design.
- Video editing, Remotion-style rendered films, and Manim math animations.

## Sources

This skill is an original synthesis (MIT) merging upstream skills — `AThevon/genjutsu` (MIT), `iart-ai/web-animation-skills` (MIT), `greensock/gsap-skills` (official, MIT), `199-biotechnologies/motion-dev-animations-skill` (MIT), `agiwhitelist/auteur` (MIT), `oil-oil/oil-motion` (MIT, driver-taxonomy concept), `scriptscrypt/animation-reverse-engineering` (credited method source), plus discovery via `frankxai/awesome-motion-design-agent-skills` (CC0). All recipes are original dual-stack implementations; npm versions live-verified. Full credits and licenses: `ATTRIBUTION.md`.
