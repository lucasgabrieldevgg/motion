# Attribution & Sources

`motion` is an original synthesis commissioned for this repo: the craft core and technology map were merged and rewritten from upstream skills, all ~30 recipes were re-authored for this skill's dual-stack (vanilla + React) format, and package versions were live-verified against the npm registry (13 October 2026) for this project. The synthesis, structure, recipe code, and merged rule set are MIT. Upstream material retains its original license.

## Upstream skills (merged)

| Source | What was merged | License |
| --- | --- | --- |
| `AThevon/genjutsu` — skills/_jutsu/motion-principles, css-native, gsap, framer-motion, canvas-generative, threejs-r3f, tells | The timing table and frequency rule, the easing cheat-sheet concept, the five "never" rules, reduced-motion mandates, scroll-driven CSS patterns (@supports gating, animation-range), @starting-style + allow-discrete, View Transitions patterns, GSAP do-nots (containerAnimation ease, ScrollTrigger child tweens, immediateRender), Motion do-nots (AnimatePresence keys, layout radius distortion, motion values), Canvas DPR-aware setup, the "tells" concept behind the slop death list | MIT |
| `iart-ai/web-animation-skills` — accessible-animation, 60fps-animation, micro-interaction, svg-animation, lottie-animation, page-transition-animation | The tiered reduced-motion model (Tier 1/2/3) with CSS/JS/library gating layers, the property-cost table, FLIP, grid 0fr→1fr height technique, layout-thrashing batching, will-change discipline, spring intuition presets, state-continuity principle, the freeze-harness verification loop (concept), pathLength="1" draw-on, MorphSVG/flubber decision, offset-path, dotLottie playback/scroll/theming, the Next.js FrozenRouter pattern | MIT |
| `greensock/gsap-skills` (official GreenSock agent skills) | quickTo for mouse followers, stagger-over-many-tweens efficiency, ScrollTrigger perf notes (pin sparingly, refresh only on layout change) — plus version-anchor for GSAP 3.15 free-plugins fact | MIT |
| `199-biotechnologies/motion-dev-animations-skill` | The purpose/principles framing (Apple/Jon Ive framing: purposeful/smooth/accessible/elegant), the ENTRANCE/GESTURE/SCROLL/LAYOUT decision-tree shape, quality-standards table (≥60fps, CLS=0, bundle budget), Motion.dev framework-integration notes | MIT |
| `agiwhitelist/auteur` | Motion-related anti-slop bans (same fade-up per section, transition:all, scale(0), scroll listeners, >1 marquee), the motion-budget concept, the motionqa gate concept that shaped the quality gate | MIT |
| `oil-oil/oil-motion` | Driver taxonomy (scroll/pointer/drag/touch/orientation/audio/data/state + scrub/segment-play/autonomous) — concept only; its media-generation pipeline is out of scope and not merged | MIT |
| `scriptscrypt/animation-reverse-engineering` | The reverse-engineering method: species classification, acquire→dissect→port→verify pipeline, measure-don't-guess easing, reference discovery sources (60fps.design, Mobbin), the ethics line | No LICENSE file in the upstream repo — public-source skill; credited here as a merged method source. If the author specifies a license, this table updates |
| `frankxai/awesome-motion-design-agent-skills` | Discovery aid (landscape index of motion skills); no content merged | CC0 1.0 (the list) |

## Original verification work

- **npm version verification**: 13 packages checked live against the npm registry on 2026-10-13 (gsap 3.15.0, motion/framer-motion 14.1.0 dual-publish confirmed, @gsap/react 2.1.2, three 0.186.1, @react-three/fiber 9.8.1, dotlottie-web 0.81.0, dotlottie-react 0.20.0, lottie-web 5.13.0, @rive-app/react-canvas 4.36.0, @rive-app/canvas 2.44.2, lenis 1.3.26, flubber 0.4.2) — reflected in `references/tech.md`.
- **Upstream landscape survey**: 8 candidate repositories cloned and studied before synthesis (January 2026 GitHub search: animation, motion design, micro-interactions, GSAP, framer motion skill queries); video-generation skills identified and excluded by scope decision.
- **Recipe corpus**: all ~30 recipes in `references/recipes-ui.md` and `references/recipes-ambient.md` were written for this repo — merging technique ideas from upstream (pseudo-element shadow, grid accordion, popLayout toasts, pathLength draw) with original dual-stack implementations, token wiring, and guard lines.
- **The structure, routing, quality gates, and wording** are original to this repo.

## Individual credits

- **AThevon** — the genjutsu multi-skill collection that forms this skill's largest merged backbone (motion principles, per-stack skills, the tells concept).
- **iart-ai** — the web-animation-skills collection whose accessibility tiering and 60fps discipline shaped this skill's hard rules.
- **GreenSock** (greensock) — the official agent skills for GSAP, and a decade of the animation engine itself.
- **199-biotechnologies** — the Motion.dev standards skill (principles framing, quality table).
- **agiwhitelist** — auteur's motion-budget and anti-slop gates.
- **oil-oil** — the driver taxonomy.
- **scriptscrypt** — the reverse-engineering method.
- **frankxai** — the awesome list that mapped the landscape.
- The referenced craftspeople whose public work shaped the taste encoded here: Emil Kowalski, Rauno Freiberg (motion values, springs), Joshua Comeau (scroll and pointer techniques), and the maintainers of GSAP, Motion, Lenis, Flubber, dotLottie, and Rive.

Any errors in the merged synthesis are this repo's, not theirs.

If you redistribute `motion`, keep this file and the per-file credits with it.
