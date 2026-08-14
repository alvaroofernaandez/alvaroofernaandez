# DESIGN.md — alvaroofernaandez (GitHub profile)

Source of truth for this repository's design system. The rendered surface is a
single GitHub profile README, so the constraints of GitHub's Markdown sanitizer
are part of the design system, not an afterthought.

## 1. Intent & North Star

**Direction: "Steel and signal".** An ink poster with one acid signal in it.

The visitor arrives from a Hagalink proposal or from a starred repo, at night,
on GitHub's dark theme, and decides in about four seconds whether this person is
serious. The page answers with a printed object, not a dashboard.

The identity blends two sources deliberately: the personal register comes from
the ink typography and the 鋼の心 seal (the account's own bio, "heart of steel"),
and the single accent colour is Hagalink's lime, so the personality is his and
the colour is his company's.

**Anti-references (explicitly rejected):**

- Badge soup: rows of shields.io pills, streak counters, trophy walls,
  "contribution snake", profile-3d. This is the saturated template.
- The fake terminal (`$ whoami`, blinking cursor, typing SVG). This is the
  anti-template, equally saturated.
- Greeting headings ("Hi, I'm X 👋") and emoji used as icons.
- Third-party stat cards. They add an external dependency, go down, and
  duplicate the contribution graph GitHub already renders directly below.

## 2. Tokens

| Role | Value | Use |
| --- | --- | --- |
| `ink` | `#0B0B0C` | Ground of every generated asset |
| `bone` | `#EDE7DC` | All type |
| `lime` | `#E6FF2B` | The single accent, from the Hagalink mark |
| `wash` | `#161A1B` | Radial depth wash, top-left, very low contrast |

**The one-accent rule.** Lime appears exactly once per asset: the brush rule in
the hero, the short tick on the colophon's top rule. It is never used for type.

**Opacity ramp on ink** (measured, not guessed):

| Opacity | Contrast on `ink` | Allowed for |
| --- | --- | --- |
| `1.00` | 16.4:1 | Display type |
| `0.92` | 12.6:1 | Body items |
| `0.72` | 8.4:1 | Secondary type |
| `0.52` | 4.8:1 | Column headers, floor for text |
| `0.42` | 3.5:1 | Large decorative type only (the seal) |
| `0.14` | — | Hairlines only, never type |

Do not drop text below `0.52`. `0.30` was tried on the seal and measured
2.4:1, which fails even the 3:1 large-text minimum.

### Typography

One family, two styles. Hierarchy comes from size, case, and tracking, never
from a second competing typeface.

- **Instrument Serif Regular** (SIL OFL 1.1) — display and body.
- **Instrument Serif Italic** — colophon column headers. Italic lowercase is
  used instead of tracked uppercase to avoid the "eyebrow" trope.
- **Noto Serif JP, wght 500** (SIL OFL 1.1) — the three seal glyphs only.

Scale in viewBox units (assets are 1200 wide and scale to roughly 830px, so
multiply by ~0.69 for rendered pixels):

| Step | Size | Tracking |
| --- | --- | --- |
| Display | 108 | 0 |
| Section head | 28 italic | 0 |
| Body | 27 | 0.055em hero, 0.01em colophon |
| Seal | 84 | — |

Display line leading is 102 on a 108 body, a tight 0.94em.

## 3. Layout Primitives

- Canvas width 1200 units for every asset, so all assets share one scale.
- Page margin 92 units, left and right.
- The hero splits into a name field (92 to 960) and a seal field (960 to 1200),
  divided by a 1px hairline at 14% opacity. The seal is optically centred inside
  its own field, not on the canvas.
- Vertical margins are optically balanced: cap-top of the first line and the
  descender of the last line sit on roughly equal margins (80 and 73 units).
- The colophon is a three-column index at x 92 / 470 / 830, with a 44-unit
  baseline step. A ragged last column is correct, not a defect.

## 4. Component Patterns

### Hero poster (`assets/hero.svg`)

Name over two ragged lines, brush rule, two role lines, vertical seal.
Accessibility: `role="img"`, `<title>`, and `aria-label` inside the SVG, plus
`alt` on the `<img>` in the README. Both are required because the SVG's own
accessibility tree is not exposed through `<img>`.

### Colophon (`assets/colophon.svg`)

The stack rendered as a printed index: italic headers, plain items, no logos and
no badges. Contents are grounded in the account's real GitHub language data, not
aspiration.

### The two hard GitHub constraints

1. **No external resources.** GitHub renders README SVGs inside `<img>`, which
   blocks every external load, `@font-face` included. All type therefore ships
   as outlined path data. There is no way around this; a `font-family` in one of
   these files is a bug.
2. **Assets carry their own ground.** GitHub's theme and the OS
   `prefers-color-scheme` can disagree, so any `<picture>` theme switch is
   unreliable. Every asset paints its own ink ground instead, which renders
   identically under both GitHub themes. Lime measures 15.5:1 on ink and 1.2:1
   on white, so lime on a transparent ground would be invisible in light mode.

## 5. Motion System

**Where motion is allowed:** the hero only. The colophon sits below the fold,
and an `<img>` animation fires on load regardless of viewport, so its entrance
would play unseen. Deliberately static.

**The gesture:** one orchestrated entrance, played once. Name lines settle
upward, the brush rule wipes left to right, the role lines follow, the seal
lands last. Nothing loops. An infinite loop on a profile page is the tell.

- Curve: `cubic-bezier(.16, 1, .3, 1)`.
- Rise 760ms, wipe 900ms. Delays 60 / 150 / 320 / 460 / 530 / 640 / 700 / 760ms.
  Total under 1.7s.
- `prefers-reduced-motion: reduce` sets `animation: none` on every class. The
  reduced state is the finished state.

**Graceful degradation is a hard requirement.** Entrances animate `transform`
only, never `opacity`. An animation that starts from `opacity: 0` with
`animation-fill-mode: backwards` publishes a blank poster in any renderer that
does not run the animation. This was reproduced in headless Chromium during the
build. The worst case now is type sitting 16px low.

The one element that does animate from nothing is the brush rule's `scaleX(0)`.
It is backed by an unanimated 11-unit lime seed tick flush with the stroke's
flat edge, so the accent survives even when the wipe never runs.

### Positioning versus motion

A CSS rule beats an SVG presentation attribute. An animated `transform` applied
to the same `<g>` that carries the positioning `translate()` silently discards
that translate and collapses the element to the origin. Positioning and motion
must live on separate nested groups. Never merge them.

## 6. Accessibility Baselines

- WCAG 2.1 AA. Text at least 4.5:1, large or decorative type at least 3:1,
  against the asset's own ink ground.
- Every `<img>` in the README carries `alt` that transcribes the asset's full
  content, including every item in the colophon.
- `prefers-reduced-motion` honoured in every animated asset.
- Colour is never the only carrier of meaning: lime marks position, never state.

## 7. Anti-patterns (banned in this project)

- Shields.io badge rows, streak or trophy cards, contribution snakes.
- Any external image host or third-party stats service.
- `@font-face` or `font-family` inside an asset. Outline the type.
- Animating `opacity` from 0 on anything that carries information.
- Emoji standing in for icons.
- A second typeface, or lime used on type.
- `<picture>` with `prefers-color-scheme` for theme switching.
- `.github/README.md`. It silently overrides the root README on GitHub.
- Anything that depends on GitHub Actions. Actions is disabled account-wide
  (`enabled: false` on every repo checked), so a workflow-generated asset would
  never build.

## 8. Decision Log

**2026-08-14 — Art direction: "Steel and signal".**
Chosen by the maintainer over a pure sumi-e personal direction and a pure
Hagalink brand direction. Ink structure carries the personal identity, the lime
accent carries the company. Supersedes nothing.

**2026-08-14 — Type outlined, not embedded.**
`<img>`-rendered SVG cannot load fonts. Outlining was chosen over a base64
WOFF2 data URI because it renders identically everywhere and keeps each asset
under 90KB. Generated with HarfBuzz shaping, so kerning is real.

**2026-08-14 — Assets carry their own ground; no theme switching.**
GitHub's theme and the OS colour scheme can disagree, which makes `<picture>`
switching unreliable in both directions. A self-grounded poster removes the
whole failure class.

**2026-08-14 — Motion animates transform only.**
Reproduced a blank hero in headless Chromium when the entrance started from
`opacity: 0` with `fill-mode: backwards`. Rewrote every entrance as
transform-only and added the lime seed tick behind the wipe.

**2026-08-14 — No live stats cards.**
They duplicate the contribution graph GitHub renders directly below the README,
and they add a third-party dependency that fails visibly when it rate-limits.

**2026-08-14 — Assets are committed, the generator is not.**
The build tool needs Python, fontTools, uharfbuzz, and two font files. Shipping
that toolchain into a profile repository costs more than it returns, and with
Actions disabled nothing could run it anyway. Regenerating means rebuilding the
script from this document; the geometry and tokens above are the specification.
The generator is therefore untested by design, as a one-shot tool whose output
is reviewed visually rather than asserted.

## Type credits

Instrument Serif by Instrument, and Noto Serif JP by Google, both under the SIL
Open Font License 1.1. Only outlines derived from them are distributed here; no
font files are included in this repository.
