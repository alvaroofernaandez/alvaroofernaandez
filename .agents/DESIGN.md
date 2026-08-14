# DESIGN.md — alvaroofernaandez (GitHub profile)

Source of truth for this repository's design system. The rendered surface is a
single GitHub profile README, so the constraints of GitHub's Markdown sanitizer
are part of the design system, not an afterthought.

## 1. Intent & North Star

**Direction: "Steel and signal", composed as a kakemono.** A centred ink scroll,
printed on marbled ink, with one acid signal in it.

The visitor arrives from a proposal or a shared link, at night, on GitHub's dark
theme, and decides in about four seconds whether this person is serious. The
page answers with a printed object, not a résumé.

The identity blends two sources deliberately: the personal register comes from
the ink typography and the 鋼の心 column (the account's own bio, "heart of
steel"), and the single accent colour is Hagalink's lime, so the personality is
his and the colour is his company's.

**The Japanese register is built from construction, not motifs.** Every
ornament is drawn geometry with a name and a history:

- **Suminagashi (墨流し)** — floating-ink marbling, the ground. Rings spread
  from three drops and a current combs them into flowing contours. Generated
  from harmonics plus a horizontal shear, **never tiled**: a repeating tile
  reads as wallpaper, and marbling never repeats. Each ring is a closed curve
  `r(θ) = R·(1 + Σ aₖ·sin(nₖθ + φₖ + i·drift))`, squashed on y, then displaced
  by `x += comb·sin(y / combLen)`. Rings in one drop share its harmonics so
  they nest like real ink; the phase drifts outward so they never look like
  plain offset curves.
- **Tategaki (縦書き)** — the bio reads top to bottom, centred above the name.
- **Hanko** — the lime square, set at the foot beneath the signature, where a
  seal belongs. It is not a bullet and never repeats.

All ornament is monochrome. Lime appears exactly once, on the hanko.

**Anti-references (explicitly rejected):**

- Badge soup: shields.io pills, streak counters, trophy walls, contribution
  snake, profile-3d. The saturated template.
- The fake terminal (`$ whoami`, blinking cursor, typing SVG). The equally
  saturated anti-template.
- Torii gates, cherry blossoms, rising-sun discs, "brush-style" display faces.
  Japanese-as-costume. The constructed patterns above replace all of it.
- Greeting headings ("Hi, I'm X 👋") and emoji used as icons.
- Third-party stat cards. An external dependency that duplicates the
  contribution graph GitHub already renders directly below.

## 2. Tokens

| Role | Value | Use |
| --- | --- | --- |
| `ink` | `#0B0B0C` | Ground of every asset |
| `bone` | `#EDE7DC` | All type and all ornament |
| `lime` | `#E6FF2B` | The hanko, and nothing else |
| `wash` | `#171B1C` | Radial sumi wash over the ground |

**Opacity ramp on ink** (measured, not guessed):

| Opacity | Role |
| --- | --- |
| `1.00` | The signature |
| `0.88` | The bio glyphs |
| `0.72` | The role line (8.4:1) |
| `0.20` | Suminagashi in the divider |
| `0.13` | Suminagashi in the poster |

Ground texture sits below the text floor on purpose; it carries no information.
Do not drop actual type below `0.52` (4.8:1). The bio was tried at `0.30`,
measured 2.4:1, and failed even the 3:1 large-text minimum.

The two suminagashi values differ because the divider is a thin strip: at the
poster's `0.13` it would disappear. They are tuned to read as the same ink, not
to be the same number.

A radial `wash` of ink at 72% sits **over** the ground and under the type,
darkening the centre so the contours never compete with the signature. Without
it the marbling reads through the letterforms.

### Typography

**One family. Only initials are capitalised.** No all-caps anywhere: not the
signature, not headings, not labels, not tracked eyebrows. Hierarchy comes from
size, tracking, and opacity.

- **Instrument Serif Regular** (SIL OFL 1.1) — signature and role line.
- **Noto Serif JP, wght 500** (SIL OFL 1.1) — the three bio glyphs.

Scale in viewBox units (the poster is 1200 wide and scales to roughly 830px, so
multiply by ~0.69 for rendered pixels):

| Step | Size | Tracking |
| --- | --- | --- |
| Signature | 80 | 0.01em |
| Bio glyph | 50 | — |
| Role line | 26 | 0.05em |

The signature and the role line are set to nearly equal measures (684 and 688
units), so the centred stack closes on clean left and right edges. Preserve that
pairing when the role text changes; it is what makes the block read as centred
rather than merely aligned.

## 3. Layout Primitives

Poster canvas 1200 × 566. **Everything is centred on the midline at x 600.**

| Element | Position |
| --- | --- |
| Bio glyphs | baselines 134 / 192 / 250, size 50 |
| Signature | baseline 376 |
| Role line | baseline 422 |
| Hanko | 16 × 16 at y 460 |

Top and bottom margins are optically equal at roughly 90 units.

The gap between the bio and the signature is measured to the **accent**, not to
the cap line: the acute on "Á" rises above cap height and collided with 心 at
the first spacing.

Ink drops at (250, 250), (640, 300), (995, 215); divider drops at (320, 300)
and (880, 322), placed below the strip so only their broad upper contours cross
it. Drops inside a thin strip produce small tight arcs that read as noise.

Divider canvas 1200 × 80, full bleed, ink ground masked to nothing at both ends.

## 4. Component Patterns

### Poster (`assets/poster.svg`)

Bio, signature, role, hanko, over the suminagashi ground.
Accessibility: `role="img"`, `<title>`, and `aria-label` inside the SVG, plus
`alt` on the `<img>` in the README. Both are required, because the SVG's own
accessibility tree is not exposed through `<img>`.

### Divider (`assets/divider.svg`)

A closing wave band. Its ink ground is masked by a horizontal gradient so it
dissolves at both ends: on GitHub's dark theme it blends into the page, and on
the light theme it reads as a deliberate wash rather than a black bar with hard
edges. It is decorative, so it carries `alt=""` and screen readers skip it.

### README body

Prose and two sections, each headed by a single kanji, a middot, and a lowercase
English word. Verified rendering in Hiragino: 今 and 心 resolve correctly through
GitHub's font stack. Sentence case is used in prose; "only initials capitalised"
governs the poster and headings, not the spelling of proper nouns.

### The two hard GitHub constraints

1. **No external resources.** GitHub renders README SVGs inside `<img>`, which
   blocks every external load, `@font-face` included. All type ships as outlined
   path data. A `font-family` in an asset is a bug.
2. **Assets carry their own ground.** GitHub's theme and the OS
   `prefers-color-scheme` can disagree, so any `<picture>` theme switch is
   unreliable. Lime measures 15.5:1 on ink and 1.2:1 on white, and bone ornament
   measures about 1.1:1 on white, so nothing may sit on a transparent ground.

## 5. Motion System

**Where motion is allowed:** the poster only, once, on load. The divider sits
below the fold, and an `<img>` animation fires on load regardless of viewport,
so its entrance would play unseen. Deliberately static.

**The gesture:** the bio lands top to bottom in the reading direction of
tategaki, the signature and role follow, and the hanko is pressed last, the way
a seal closes a signed scroll. The ground never animates. Nothing loops.

- Curve: `cubic-bezier(.16, 1, .3, 1)`.
- Rises 780ms, press 520ms.
  Delays 120 / 200 / 280 / 420 / 500 / 660ms. Total under 1.2s.
- `prefers-reduced-motion: reduce` sets `animation: none` on every class. The
  reduced state is the finished state.

**Graceful degradation is a hard requirement.** Nothing animates `opacity`, and
nothing animates existence. An animation starting from `opacity: 0` with
`animation-fill-mode: backwards` publishes a blank poster in any renderer that
does not run the animation; this was reproduced in headless Chromium during the
build. Worst case now: type sits 16 units low and the hanko sits 20% large. The
ground is never animated at all. Every element remains present and legible.

### Positioning versus motion

A CSS rule beats an SVG presentation attribute. An animated `transform` applied
to the same `<g>` that carries the positioning `translate()` silently discards
that translate and collapses the element to the origin. Positioning and motion
must live on separate nested groups. Never merge them.

## 6. Accessibility Baselines

- WCAG 2.1 AA for type. Ornament is exempt because it carries no information.
- The poster's `alt` transcribes it in full, including a gloss of 鋼の心, since
  the meaning is unavailable to a non-Japanese reader. The divider is `alt=""`.
- `prefers-reduced-motion` honoured.
- Colour is never the only carrier of meaning: the hanko marks a signature's
  foot, never a state.
- The role line is small at mobile widths by design. Its content is repeated as
  real markdown text below, so no information lives only inside an image.

## 7. Anti-patterns (banned in this project)

- **All-caps.** Only initials are capitalised, anywhere.
- **Repository, language, and stack content.** No project tables, no language
  lists, no tooling inventories, no "built with". The profile is a person, not
  a manifest.
- **Japanese motifs as decoration.** Torii, sakura, rising suns, katana, kanji
  chosen for looks rather than meaning. Ornament must be constructed geometry
  with a name.
- Shields.io badge rows, streak or trophy cards, contribution snakes.
- Any external image host or third-party stats service.
- `@font-face` or `font-family` inside an asset. Outline the type.
- Animating `opacity`, or animating existence, on anything.
- Emoji standing in for icons.
- A second typeface, or lime used on anything but the hanko.
- `<picture>` with `prefers-color-scheme` for theme switching.
- Ornament above the text floor: the ground must stay quieter than the type it
  sits behind.
- Tiling the ground. Marbling that repeats is wallpaper.
- Aphoristic copy: the "serious statement, then punchy short negation" cadence.
- `.github/README.md`. It silently overrides the root README on GitHub.
- Anything depending on GitHub Actions. Actions is disabled account-wide
  (`enabled: false` on every repo checked), so a workflow-generated asset would
  never build.

## 8. Decision Log

**2026-08-14 — Art direction: "Steel and signal".**
Chosen by the maintainer over a pure sumi-e personal direction and a pure
Hagalink brand direction. Still in force.

**2026-08-14 — Type outlined, not embedded.**
`<img>`-rendered SVG cannot load fonts. Outlining renders identically everywhere
and keeps the poster under 70KB. Generated with HarfBuzz shaping, so kerning is
real. Still in force.

**2026-08-14 — Assets carry their own ground; no theme switching.**
GitHub's theme and the OS colour scheme can disagree, which makes `<picture>`
switching unreliable in both directions. Still in force.

**2026-08-14 — Motion never animates opacity or existence.**
Reproduced a blank hero in headless Chromium when the entrance started from
`opacity: 0` with `fill-mode: backwards`. Still in force, and extended: the
enso animates rotation and scale rather than being drawn from nothing, so it
too survives a renderer that skips the animation.

**2026-08-14 — No live stats cards.** Still in force.

**2026-08-14 — Lowercase, no inventory content.** *(partly superseded below)*
The uppercase two-line display signature was replaced by a single lowercase
line, and the colophon asset, the selected-work table, and the project links
were deleted. The inventory ban stands. The all-lowercase signature does not.

**2026-08-14 — Centred kakemono, capitalised initials, constructed ornament.**
Requested by the maintainer after seeing the asymmetric version live. Three
changes:

- The ma / fukinsei composition, with its void upper-left and the bio hung in a
  corner, is **superseded** by a symmetric centred stack. The maintainer read
  the asymmetry as misalignment rather than as intent, which means it failed at
  its job regardless of the principle behind it. Symmetry is also a legitimate
  Japanese register: this is a kakemono, not a sumi-e scroll.
- The all-lowercase signature is **superseded**: initials are capitalised.
  All-caps remains banned.
- Ornament was added rather than implied: seigaiha ground, the enso ring, and a
  closing divider. The keisen hairline and the horizontal brush rule are
  **superseded and removed**, since the enso now carries the brush gesture.

**2026-08-14 — Suminagashi ground, enso removed.**
Requested by the maintainer. The seigaiha tile and the enso ring are
**superseded and deleted**. The ground is now suminagashi, generated rather
than tiled. The bio stands on its own above the signature with no frame, and
the poster lost the ring's height: 620 units becomes 566.

Seigaiha was the safer pattern and it tiled cheaply, but a tile is the one
thing marbling can never be. Generating the ground costs about 48KB of path
data in the poster; that is the price of the ground not repeating, and it is
worth paying.

**2026-08-14 — A version query is the cache-busting mechanism.**
*(supersedes the rename rule recorded the same day)*

GitHub serves README images through a caching proxy, so after a redesign the
old artwork keeps being served to anyone who already loaded the page. That is
indistinguishable from a change that never shipped. Diagnose it by curling
`raw.githubusercontent.com`: if the served bytes are already correct, it is
cache, not code.

The first fix was a rename, `hero.svg` to `poster.svg`. That works but leaves
the filename carrying version history, which it should not.

Verified better: GitHub preserves query strings when it rewrites relative image
paths, so `assets/poster.svg?v=2` reaches the browser intact and is a distinct
cache key. **Bump the `v` on every visual change to a committed asset.** Do not
rename files for this, and do not tell one person to hard-refresh: that fixes
the page for them and for nobody else who has already visited.

**2026-08-14 — Assets are committed, the generator is not.**
The build tool needs Python, fontTools, uharfbuzz, and two font files. Shipping
that toolchain into a profile repository costs more than it returns, and with
Actions disabled nothing could run it anyway. Regenerating means rebuilding the
script from this document; the geometry and tokens above are the specification,
including the suminagashi harmonics, drop positions, and shear. The
generator is untested by design, a one-shot tool whose output is reviewed
visually rather than asserted.

## Type credits

Instrument Serif by Instrument, and Noto Serif JP by Google, both under the SIL
Open Font License 1.1. Only outlines derived from them are distributed here; no
font files are included in this repository.
