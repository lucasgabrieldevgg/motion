# motion

A big skill for **animation by code** — beautiful, professional, physical. Born from merging the best animation skills found on GitHub (the official GreenSock agent skills included) into one method-first discipline: craft before recipes, recipes before libraries.

```
Motion is not decoration. It is the part of an interface
that communicates when words can't: what changed, where things
came from, what the user just did.
```

## What's inside

| Piece | What it does |
| --- | --- |
| **Motion craft** (`references/craft.md`) | The 70%: five purposes, frequency law, duration scale, named easing library with real bezier values, spring presets, choreography laws (asymmetry, stagger, one-director, state continuity), motion tokens, the 14-item slop death list |
| **Technology map** (`references/tech.md`) | Eight stacks with decision tree: modern CSS (scroll-driven, View Transitions, @starting-style), WAAPI + rAF + FLIP, GSAP (ScrollTrigger, quickTo, do-nots), Motion/framer-motion (AnimatePresence, layout, motion values), SVG (draw-on, morph), Canvas 2D & three.js, Lottie & Rive — npm versions live-verified |
| **UI recipes** (`references/recipes-ui.md`) | 17 recipes, each vanilla + React with guard lines: reveals (fade-up, stagger, clip wipe, blur-in, split-text, masked), micro-interactions (press, hover lift, magnetic, toggle, like burst), toasts, accordions, modals, page transitions (FrozenRouter), skeleton→content, spinner→success morph |
| **Ambient recipes** (`references/recipes-ambient.md`) | 13 recipes: scroll progress, parallax, horizontal scroll, pinned scenes, count-up, marquee, scramble, typewriter, aurora background, canvas particles, custom cursor, 3D tilt, confetti |
| **Rules & gates** (`references/rules.md`) | Tiered reduced-motion (never a kill switch), the 60fps property-cost budget, CLS discipline, the freeze-frame verification harness, the 10-line ship gate |

Plus a Cursor rules export at `exports/motion.mdc`.

## Install

**Claude Code / GLM / any SKILL.md host:**
```bash
git clone https://github.com/lucasgabrieldevgg/motion.git ~/.claude/skills/motion
```

**Cursor:** copy `exports/motion.mdc` to `.cursor/rules/motion.mdc`.

**ClawHub / skills.sh:** the folder is standard skill layout — publish as-is.

## The method (why craft is 70%)

Durations, easings, and springs are physics; choreography is taste; both are learned before any library. A page earns one signature moment; everything else is working motion answering one of five purposes. The slop death list exists because the default failure mode of AI-generated animation is exactly the same as AI-generated design: perpetual motion, the same fade-up everywhere, `linear` easing, `scale(0)`, parallax on paragraphs. The recipe catalog exists to serve the loop, not replace it — every recipe wires back to the tokens and laws in `craft.md`.

## Verification

Animation libraries change fast. Package versions in `references/tech.md` (gsap 3.15.0, motion 14.1.0, three 0.186.1, dotlottie 0.81.0, rive 4.36.0, lenis 1.3.26...) were live-checked against the npm registry on 2026-10-13. Browser-feature claims (scroll-driven animations, View Transitions, @starting-style) carry their own support caveats inline — re-verify against your target browsers before relying on a feature table.

## Attribution

`motion` is an original synthesis (MIT) merging upstream material from `AThevon/genjutsu` (MIT), `iart-ai/web-animation-skills` (MIT), `greensock/gsap-skills` (MIT), `199-biotechnologies/motion-dev-animations-skill` (MIT), `agiwhitelist/auteur` (MIT), `oil-oil/oil-motion` (MIT, concept), `scriptscrypt/animation-reverse-engineering` (credited method source), and `frankxai/awesome-motion-design-agent-skills` (CC, discovery aid), plus original recipe authorship and live version verification. See [`ATTRIBUTION.md`](ATTRIBUTION.md).

## License

MIT for this repo's original content. Upstream material retains its own licenses — see ATTRIBUTION.md.
