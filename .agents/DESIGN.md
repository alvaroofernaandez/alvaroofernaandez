# DESIGN.md — alvaroofernaandez (GitHub profile)

Source of truth for this repository's design system. The rendered surface is a
single GitHub profile README, so the constraints of GitHub's Markdown sanitizer
are part of the design system, not an afterthought.

## 1. Intent & North Star

**Direction: "Steel and signal", composed on Japanese layout principles.**
An ink poster with one acid signal in it, and a great deal of nothing.

The visitor arrives from a proposal or a shared link, at night, on GitHub's dark
theme, and decides in about four seconds whether this person is serious. The
page answers with a printed object, not a résumé.

The identity blends two sources deliberately: the personal register comes from
the ink typography and the 鋼の心 column (the account's own bio, "heart of
steel"), and the single accent colour is Hagalink's lime, so the personality is
his and the colour is his company's.

**The Japanese register is structural, never decorative.** Three principles do
the work, and no motif is allowed to stand in for them:

- **Ma (間)** — the void is an element, not leftover space. The upper-left of the
  poster is empty on purpose and must stay empty.
- **Fukinsei (不均整)** — balance without symmetry. The column hangs top-right,
  the signature sits bottom-left, and the tension between them is the layout.
- **Tategaki (縦書き)** — the bio is set vertically, read top to bottom, with a
  keisen (罫線) hairline ruling its column as in a printed page.

The lime square is a **hanko**, placed after the signature because that is where
a seal goes. It is not a bullet and never repeats.

**Anti-references (explicitly rejected):**

- Badge soup: shields.io pills, streak counters, trophy walls, contribution
  snake, profile-3d. The saturated template.
- The fake terminal (`$ whoami`, blinking cursor, typing SVG). The equally
  saturated anti-template.
- Torii gates, cherry blossoms, rising-sun discs, and "brush-style" display
  faces. Japanese-as-costume. The principles above replace all of it.
- Greeting headings ("Hi, I'm X 👋") and emoji used as icons.
- Third-party stat cards. An external dependency that duplicates the
  contribution graph GitHub already renders directly below.

## 2. Tokens

| Role | Value | Use |
| --- | --- | --- |
| `ink` | `#0B0B0C` | Ground of the poster |
| `bone` | `#EDE7DC` | All type |
| `lime` | `#E6FF2B` | The hanko, and nothing else |
| `wash` | `#161A1B` | Radial sumi wash, upper left, very low contrast |

**The one-accent rule.** Lime appears exactly once, as the hanko. It is never
used for type, rules, or emphasis.

**Opacity ramp on ink** (measured, not guessed):

| Opacity | Contrast on `ink` | Allowed for |
| --- | --- | --- |
| `1.00` | 16.4:1 | The signature |
| `0.72` | 8.4:1 | The role line |
| `0.55` | 5.3:1 | The vertical column |
| `0.14` | — | Keisen hairline only, never type |

Do not drop type below `0.52` (4.8:1). The column was tried at `0.30`, measured
2.4:1, and failed even the 3:1 large-text minimum.

### Typography

**One family. No uppercase anywhere, ever.** Case is not a hierarchy tool in
this system; size, tracking, and opacity are.

- **Instrument Serif Regular** (SIL OFL 1.1) — signature and role line.
- **Noto Serif JP, wght 500** (SIL OFL 1.1) — the three column glyphs.

Scale in viewBox units (the asset is 1200 wide and scales to roughly 830px, so
multiply by ~0.69 for rendered pixels):

| Step | Size | Tracking |
| --- | --- | --- |
| Signature | 84 | 0.01em |
| Column glyph | 88 | — |
| Role line | 26 | 0.05em |

The signature and the role line are set to nearly equal measures (700 and 688
units), so the left block closes on a clean right edge. Preserve that pairing
when the role text changes.

## 3. Layout Primitives

Canvas 1200 × 516. Page margin 92 left, 64 top and bottom.

- **Left field** (92 to 992): empty above y 348. The signature sits on baseline
  404, the role line on 444. This lower-left cluster is the whole of the Latin
  content.
- **Right column** (keisen at x 992): three glyphs on baselines 152, 252, 352,
  centred on x 1062. It ends above the signature, the way a hanging scroll ends
  above where it is signed.
- **The void** spans roughly x 92–992, y 60–348. It is the largest single
  element on the poster. Do not fill it.

## 4. Component Patterns

### Poster (`assets/hero.svg`)

The only asset. Signature, hanko, role line, vertical column, keisen.
Accessibility: `role="img"`, `<title>`, and `aria-label` inside the SVG, plus
`alt` on the `<img>` in the README. Both are required, because the SVG's own
accessibility tree is not exposed through `<img>`.

### README body

Prose and two sections, each headed by a single kanji, a middot, and a lowercase
English word. Verified rendering in Hiragino: 今 and 心 resolve correctly through
GitHub's font stack.

Sentence case is used in prose. "No uppercase" governs the poster and headings,
not the spelling of proper nouns.

### The two hard GitHub constraints

1. **No external resources.** GitHub renders README SVGs inside `<img>`, which
   blocks every external load, `@font-face` included. All type therefore ships
   as outlined path data. A `font-family` in the asset is a bug.
2. **The asset carries its own ground.** GitHub's theme and the OS
   `prefers-color-scheme` can disagree, so any `<picture>` theme switch is
   unreliable. The poster paints its own ink ground and renders identically
   under both GitHub themes. Lime measures 15.5:1 on ink and 1.2:1 on white, so
   lime on a transparent ground would vanish in light mode.

## 5. Motion System

**Where motion is allowed:** the poster only, once, on load.

**The gesture:** the signature and role settle upward, the column lands top to
bottom in the reading direction of tategaki, and the hanko is pressed last, the
way a seal closes a signed page. Nothing loops. An infinite loop on a profile
page is the tell.

- Curve: `cubic-bezier(.16, 1, .3, 1)`.
- Rise 780ms, press 520ms. Delays 60 / 150 / 320 / 400 / 480 / 660ms.
  Total under 1.2s.
- `prefers-reduced-motion: reduce` sets `animation: none` on every class. The
  reduced state is the finished state.

**Graceful degradation is a hard requirement.** Entrances animate `transform`
only, never `opacity`. An animation starting from `opacity: 0` with
`animation-fill-mode: backwards` publishes a blank poster in any renderer that
does not run the animation. This was reproduced in headless Chromium during the
build. Worst case now: type sits 16 units low and the hanko sits 18% large. A
useful consequence is that if CSS is blocked entirely, everything lands in its
finished state.

### Positioning versus motion

A CSS rule beats an SVG presentation attribute. An animated `transform` applied
to the same `<g>` that carries the positioning `translate()` silently discards
that translate and collapses the element to the origin. Positioning and motion
must live on separate nested groups. Never merge them.

## 6. Accessibility Baselines

- WCAG 2.1 AA. Text at least 4.5:1 against the poster's own ink ground.
- The `<img>` carries `alt` transcribing the poster in full, including a gloss
  of 鋼の心, since the meaning is not available to a non-Japanese reader.
- `prefers-reduced-motion` honoured.
- Colour is never the only carrier of meaning: the hanko marks a signature's
  end, never a state.
- The role line is small at mobile widths by design. Its content is repeated as
  real markdown text below, so no information lives only inside the image.

## 7. Anti-patterns (banned in this project)

- **Uppercase.** No all-caps signature, headings, labels, or tracked eyebrows.
- **Repository, language, and stack content.** No project tables, no language
  lists, no tooling inventories, no "built with". The profile is a person, not
  a manifest.
- Shields.io badge rows, streak or trophy cards, contribution snakes.
- Any external image host or third-party stats service.
- `@font-face` or `font-family` inside the asset. Outline the type.
- Animating `opacity` from 0 on anything that carries information.
- Emoji standing in for icons.
- A second typeface, or lime used on type.
- `<picture>` with `prefers-color-scheme` for theme switching.
- Filling the void.
- Aphoristic copy: the "serious statement, then punchy short negation" cadence.
- `.github/README.md`. It silently overrides the root README on GitHub.
- Anything depending on GitHub Actions. Actions is disabled account-wide
  (`enabled: false` on every repo checked), so a workflow-generated asset would
  never build.

## 8. Decision Log

**2026-08-14 — Art direction: "Steel and signal".**
Chosen by the maintainer over a pure sumi-e personal direction and a pure
Hagalink brand direction. Ink structure carries the personal identity, the lime
accent carries the company. Still in force.

**2026-08-14 — Type outlined, not embedded.**
`<img>`-rendered SVG cannot load fonts. Outlining was chosen over a base64
WOFF2 data URI because it renders identically everywhere and keeps the asset
under 65KB. Generated with HarfBuzz shaping, so kerning is real. Still in force.

**2026-08-14 — The asset carries its own ground; no theme switching.**
GitHub's theme and the OS colour scheme can disagree, which makes `<picture>`
switching unreliable in both directions. A self-grounded poster removes the
whole failure class. Still in force.

**2026-08-14 — Motion animates transform only.**
Reproduced a blank hero in headless Chromium when the entrance started from
`opacity: 0` with `fill-mode: backwards`. Every entrance is transform-only.
Still in force.

**2026-08-14 — No live stats cards.**
They duplicate the contribution graph GitHub renders directly below the README,
and add a third-party dependency that fails visibly when it rate-limits. Still
in force.

**2026-08-14 — Japanese composition, lowercase, no inventory content.**
Requested by the maintainer. Three changes, all superseding earlier decisions:

- The uppercase two-line display signature is **superseded** by a single
  lowercase line. Uppercase is now banned outright (§7).
- The colophon asset listing languages, interface tools, and practice is
  **superseded and deleted**, along with the selected-work table and the
  project links in the README body. Repository and stack content is now banned
  outright (§7).
- The horizontal brush rule and its lime seed tick are **superseded** by the
  hanko square, which needs no degradation backstop because it animates scale
  rather than existence.

The layout was recomposed around ma, fukinsei, and tategaki rather than
restyled, because lowercasing the previous poster would have left a Western
editorial layout wearing Japanese glyphs.

**2026-08-14 — The asset is committed, the generator is not.**
The build tool needs Python, fontTools, uharfbuzz, and two font files. Shipping
that toolchain into a profile repository costs more than it returns, and with
Actions disabled nothing could run it anyway. Regenerating means rebuilding the
script from this document; the geometry and tokens above are the specification.
The generator is therefore untested by design, a one-shot tool whose output is
reviewed visually rather than asserted.

## Type credits

Instrument Serif by Instrument, and Noto Serif JP by Google, both under the SIL
Open Font License 1.1. Only outlines derived from them are distributed here; no
font files are included in this repository.
