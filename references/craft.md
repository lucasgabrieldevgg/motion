# Motion craft — physics, choreography, taste

> Merged from: genjutsu `motion-principles` + `tells` (AThevon, MIT), web-animation-skills `micro-interaction` (iart-ai, MIT), motion-dev-animations-skill (199-biotechnologies, MIT), auteur (agiwhitelist, MIT), oil-motion driver taxonomy (oil-oil, MIT). Full credits in ATTRIBUTION.md.

Motion is not decoration. It is the part of an interface that communicates *when* words can't: what changed, where things came from, what the user just did, what the system is doing. This file is the physics and the taste; the technology lives in `tech.md`, the recipes in `recipes-ui.md` and `recipes-ambient.md`.

## 1. Why anything moves — the five purposes

Every animation in a shipped product must answer one of these, or it gets cut:

| Purpose | What it communicates | Example |
| --- | --- | --- |
| **Feedback** | "Your input registered" | Button compresses on press; toggle springs |
| **Continuity** | "This is the same object, moved" | List item morphs into detail view; toast slides from the edge |
| **Orientation** | "You are here; that came from there" | Drawer slides from its edge; breadcrumb animates direction |
| **Hierarchy** | "Look here first" | Hero staggers in before secondary content |
| **Delight** | "This product has a hand" | A single signature moment — rationed, never ambient |

Delight is the smallest budget and the most abused. A page earns one signature moment; everything else is working motion. When a design "feels alive" in a good way, it is almost always feedback + continuity done consistently, not more animation.

## 2. The frequency law

**The more often an animation plays, the shorter and subtler it must be.**

| Plays per day | Budget |
| --- | --- |
| ~1000× (hover, focus, toggle) | 100–150ms, opacity/scale only, no choreography |
| ~10× (modal open, tab switch) | 200–300ms, one axis of movement + fade |
| ~1× ever (onboarding reveal, launch hero) | 400–600ms+ full choreography allowed |

A button hover that runs 600ms with staggered choreography is hostility with good intentions. An onboarding reveal at 100ms flat is a wasted moment. The law runs both ways.

## 3. The duration scale

| Context | Duration | Why |
| --- | --- | --- |
| Micro-interaction (toggle, hover, focus) | 100–150ms | Instant feedback, no perceived delay |
| UI transition (modal, drawer, toast, tab) | 200–300ms | Smooth but never sluggish |
| Page / route transition | 300–500ms | Establishes spatial narrative |
| Signature moment (rare, 1× per page) | 400–800ms | The one place choreography is allowed |
| Scroll-driven / scrub / 3D | progress-based | Tied to input, no fixed duration |
| Ambient loops (backgrounds, marquees) | 8–30s cycles | Below conscious attention |

Hard ceiling: **nothing a user waits on exceeds 500ms.** Modals, dropdowns, tooltips, tab panels — the user asked for content; motion is the escort, not the show.

## 4. The easing library

Named curves, real values. Memorize three; recognize the rest.

| Name | Value | Use |
| --- | --- | --- |
| **snappy** | `cubic-bezier(0.2, 0, 0, 1)` | Default for UI state changes; fast start, smooth land |
| **enter** (decel) | `cubic-bezier(0.16, 1, 0.3, 1)` | Elements arriving; strong deceleration into place |
| **standard** (move) | `cubic-bezier(0.4, 0.2, 0.2, 1)` → use `(0.4, 0, 0.2, 1)` | Moving between two visible states, ease-in-out |
| **overshoot** | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Playful arrivals (toggles, likes); the only sanctioned bounce |
| **exit** (accel) | `cubic-bezier(0.55, 0, 1, 0.45)` | Elements leaving; gets out of the way |
| **dramatic** (expo-out) | `cubic-bezier(0.16, 1, 0.3, 1)` w/ longer duration, or GSAP `expo.out` | Cinematic reveals; distance sells it |
| **linear** | — | Scroll-scrub mapping only. Never for UI state. |

Laws:

- **Element enters → ease-out family. Element exits → ease-in family. Moves between states → ease-in-out. Scroll-synced → linear** (any easing on scrub breaks the 1:1 input mapping).
- **Exit is always more subtle than enter.** Enter: 300ms ease-out with movement. Exit: 150–200ms ease-in, opacity-forward, half the distance.
- Linear is a scrub value, not a personality. The moment `linear` appears on an entrance, someone stopped designing.
- GSAP equivalents: `power2.out` ≈ standard ease-out, `power3.out`/`expo.out` ≈ enter/dramatic, `back.out(1.7)` ≈ overshoot.

### Springs (when to prefer them)

Springs beat durations when motion must be **interruptible** (drag release, gestures) or feel **physical** (toggles, press release). Tune three numbers:

- `stiffness` — speed of response. Higher = faster settle.
- `damping` — how much bounce is absorbed. Lower = more overshoot.
- `mass` — almost never touch it; it slows the response.

Presets that ship:

| Feel | Motion (framer-motion) | Intuition |
| --- | --- | --- |
| **Snappy UI** (default) | `stiffness: 400, damping: 30` | Modal, drawer, tab |
| **Tactile press** | `stiffness: 500, damping: 30` | Button release, toggle snap |
| **Bouncy reveal** | `stiffness: 300, damping: 12–18` | Like burst, toast, success state |
| **Drag follow** | `stiffness: 800+, damping: 35–40` | Tracks finger closely, tiny lag |
| **Soft float** | `stiffness: 120, damping: 14` | Custom cursor, ambient hover |

Springs ignore `duration` — if a deterministic duration matters (a reveal that must finish before content is readable), use a tween.

## 5. Choreography laws

### 5.1 Asymmetry

Arrival is theater, departure is logistics. Enter with full intent (distance + fade + easing); exit with opacity-forward, half duration, half distance. Users forgive a slow entrance once; they never forgive a slow exit.

### 5.2 Stagger laws

- **30–80ms per item** — below 30ms reads as one blob, above 80ms reads as waiting.
- Direction is meaning: forward (list order), reverse (newest-first), center-out (grids, hero collections).
- **Feather**: aim for 2–4 items mid-transition at any instant. A stagger where every item waits its turn is a queue, not choreography.
- Cap the total: `stagger × count ≤ 400ms` for waiting contexts; truncate with `stagger: { each, from: "center" }` or by grouping.
- CSS can do it dependency-free: `animation-delay: calc(sibling-index() * 60ms)`.

### 5.3 One director at a time

A screen has **one primary motion** at any moment; everything else yields (shorter, subtler, or waits). Two simultaneous full choreographies fight for the eye and both lose. This is the "motion budget" made operational: declare the page's animated moments up front, and cut to budget.

### 5.4 Spatial continuity

Things come from where they belong: drawers from their edge, toasts from the nearest edge, modals from center (scale 0.96 + fade, never `scale(0)`), shared elements along their real path (`layoutId`, FLIP). An element that teleports or arrives from a direction with no spatial logic breaks the user's spatial model — the cost is disorientation, which is exactly what animation exists to prevent.

### 5.5 State continuity

`loading → success → error` is **one object that morphs**, never three jump cuts. Keep the element mounted and animate between states so the eye tracks the same thing. A button that becomes its own success state (spinner → checkmark morph) communicates causality; a button that swaps for a different element communicates nothing.

### 5.6 The classic twelve, translated for UI

| Classic principle | UI translation |
| --- | --- |
| Squash & stretch | Press compresses to `scale: 0.96` (subtle, never cartoon) |
| Anticipation | Rare in UI; a 50ms pre-motion on interactive elements at most |
| Follow-through / overlapping | Overshoot easing on arrival; related elements offset 40–60ms |
| Slow in / slow out | Ease-out enter, ease-in exit — the whole easing section |
| Arc | Movement follows content logic (diagonal for cards, straight for drawers) |
| Secondary action | Parent moves; badge/shadow counter-moves 60ms later |
| Timing | The duration scale and frequency law above |

The rest (exaggeration, appeal) belong to animation film, not interfaces. Interfaces are rationed realism.

## 6. Drivers — what makes it move

Classify before choreographing (taxonomy from oil-motion, MIT):

| Driver | Behavior | Examples |
| --- | --- | --- |
| **time** | autonomous; plays on mount or loop | hero entrance, ambient background |
| **state** | discrete segments on user/system state | modal open, toggle, loading |
| **scroll** | continuous progress mapping | parallax, progress bar, pinned scene |
| **pointer** | follows cursor position | magnetic button, custom cursor, tilt |
| **drag** | 1:1 tracking + release physics | sliders, reorder, pull-to-refresh |
| **touch / gesture** | mobile equivalents of pointer/drag | swipe, sheet dismiss |
| **data / audio** | external signal drives values | live chart, audio-reactive viz |

And the time-control question for driven motion: **scrub** (input maps to timeline position, frame stops when input stops), **segment-play** (input selects a state, segment plays; reverse input rewinds, never hard-cuts), or **autonomous** (time advances; interaction pauses/plays). Ambiguity here is the root cause of "it feels wrong" scroll experiences.

## 7. Motion tokens

Values are design tokens, not per-component improvisation:

```css
:root {
  --dur-micro: 120ms;
  --dur-ui:    240ms;
  --dur-page:  400ms;
  --ease-snappy:    cubic-bezier(0.2, 0, 0, 1);
  --ease-enter:     cubic-bezier(0.16, 1, 0.3, 1);
  --ease-exit:      cubic-bezier(0.55, 0, 1, 0.45);
  --ease-overshoot: cubic-bezier(0.34, 1.56, 0.64, 1);
  --spring-snappy:  stiffness 400 / damping 30;   /* Motion */
  --stagger:        60ms;
}
```

A product whose buttons, drawers, and toasts each invent their own timing is a product that feels assembled. Two durations + three easings cover 90% of any design system.

## 8. The slop death list

Each of these is a *tell* — motion nobody asked for, produced by reflex. Ship none of them (catalog concept from genjutsu `tells`; bans cross-checked with auteur's list):

| # | Slop | Why it dies |
| --- | --- | --- |
| 1 | **Perpetual motion** — everything loops, floats, or pulses forever | Constant motion = zero information; it also burns battery and CPU |
| 2 | **The same fade-up on every section** | Reads as a template within 10 seconds; each reveal should fit what it reveals |
| 3 | **`scale(0)` entrances/exits** | Vanishing into a black hole; start/end at `0.95` + opacity |
| 4 | **`linear` easing on UI state** | Feels like a conveyor, not a hand |
| 5 | **Waiting animation over 500ms** | Users wait for content, not choreography |
| 6 | **Bounce on everything** | Overshoot is for playful arrivals; a bouncing settings panel is a joke with no punchline |
| 7 | **Parallax on reading text** | Vestibular trigger + legibility cost; parallax is for layers, never paragraphs |
| 8 | **More than one marquee per page** | At that point it's a screensaver |
| 9 | **Glow/blob motion** (animated glow, wobbling blobs as default texture) | Decorative filler; carries no information |
| 10 | **Autoplay carousel with no pause** | WCAG 2.2.2 Level A failure, motion nobody consented to |
| 11 | **`transition: all`** | Animates properties you didn't choose, including ones you'll add later |
| 12 | **Scroll-hijack with no escape** | The user's scroll wheel is not your play button; pinned scenes must end promptly and never trap |
| 13 | **Hover animations that move content the user is trying to read/click** | The hover target must stay under the cursor |
| 14 | **Animation that runs while content is already readable** | If the text is there, the escort finished its job — end it |

One sanction: a deliberate, argued exception (a brand moment, a game) is voice. A default that produced itself is slop. The test is whether you can name what the motion communicates — "it looks cool" is not an answer.

## 9. When there's no design — reverse-engineer

Given a reference (a video, a Dribbble, a screen recording): **eyeballing a video at 1× lies about easing, stagger order, overlap, and timing — always dissect first** (pipeline from animation-reverse-engineering, scriptscrypt):

1. **Classify**: timeline choreography (time-driven: entrances, sweeps) vs interaction-driven (input maps to progress) — hybrids get both analyses.
2. **Acquire**: download, probe fps/duration (`ffprobe`), tile a contact sheet at ~2fps to map transition windows.
3. **Dissect**: frame stacks of just the moving region, near native fps; measure easing from frame-by-frame deltas — never guess.
4. **Analyse**: which properties move, stagger order and feather, asymmetry, handoff overlaps, what explicitly does *not* move.
5. **Port** to the production stack; **verify** by driving to the same states and screenshotting against the dissected frames.

Reference hunting grounds: 60fps.design, Mobbin, Awwwards, Godly — and the craft accounts (@emilkowalski_, @raunofreiberg, @jh3yy). Ethics: study technique, never clone a branded identity 1:1.
