# 02 — Design system

Direction: **Swiss cyber-minimalism.** Monochrome, large neo-grotesk type, hairline rules, strict grid, one electric accent. Pages are made of full-width *blocks* that alternate between a dark and a light theme — the switch is the main rhythm of the site.

Put all tokens in `src/styles/tokens.css` as CSS custom properties. Nothing in components may use a raw hex value or a raw px spacing value.

## Color

### Dark theme (`[data-theme="dark"]`)
| Token | Value | Use |
|---|---|---|
| `--bg` | `#08090B` | block background (space black) |
| `--bg-raised` | `#111316` | cards, window frames (graphite) |
| `--bg-sunken` | `#1A1D21` | inputs, hover fills (wet asphalt) |
| `--fg` | `#F2F2EF` | primary text |
| `--fg-muted` | `#8C9199` | secondary text, labels |
| `--line` | `rgba(242,242,239,0.12)` | hairlines |
| `--line-strong` | `rgba(242,242,239,0.28)` | active hairlines, focus outlines base |

### Light theme (`[data-theme="light"]`)
| Token | Value | Use |
|---|---|---|
| `--bg` | `#F3F3F0` | block background (paper) |
| `--bg-raised` | `#FBFBF9` | cards |
| `--bg-sunken` | `#E6E7E4` | inputs, hover fills (matte silver) |
| `--fg` | `#0B0C0E` | primary text |
| `--fg-muted` | `#5C6169` | secondary text |
| `--line` | `rgba(11,12,14,0.12)` | hairlines |
| `--line-strong` | `rgba(11,12,14,0.30)` | |

### Accent (same in both themes)
| Token | Value | Rule |
|---|---|---|
| `--accent` | `#C8FF2E` | "volt" — primary buttons, active states, key numbers **on dark**, the progress line in the nav, focus rings |
| `--accent-ink` | `#0B0C0E` | text on accent fills |

Rules: the accent covers **< 3 %** of any screen. Never use accent as text colour on the light theme (fails contrast) — on light blocks the accent appears only as a fill (button, dot, underline bar) with `--accent-ink` text. No gradients, no glows except a single `box-shadow: 0 0 0 1px var(--accent), 0 0 24px -6px var(--accent)` on the primary button hover in dark blocks.

A very subtle paper grain is allowed on light blocks: an inline SVG `feTurbulence` noise at 3–4 % opacity as a fixed `background-image` (data URI, < 1 KB). No grain on dark blocks.

## Typography

Self-host via npm (`@fontsource-variable/*`), subsets `latin`, `latin-ext`, `cyrillic`. `font-display: swap`; preload only the display weight used in the hero.

| Role | Family | Settings |
|---|---|---|
| Display | **Inter Tight Variable** | weights 500–600, `letter-spacing: -0.035em` at ≥ 48 px, `-0.02em` below; `line-height: 0.95–1.05` |
| Text | **Inter Variable** | 400/500, `line-height: 1.5`, `font-feature-settings: "ss01", "cv11"` |
| Labels / index numbers / tech meta | **JetBrains Mono Variable** | 400, UPPERCASE, `letter-spacing: 0.06em`, size 11–13 px |

Fluid scale (`clamp`, 375 → 1440 px):

| Token | Min → Max |
|---|---|
| `--fs-mega` | 56 → 184 px (hero word, footer bending text) |
| `--fs-h1` | 44 → 112 px |
| `--fs-h2` | 34 → 72 px |
| `--fs-h3` | 22 → 32 px |
| `--fs-lead` | 18 → 24 px |
| `--fs-body` | 16 → 17 px |
| `--fs-small` | 14 → 15 px |
| `--fs-label` | 11 → 13 px (mono) |

Rules: max line length 60ch for body, 18ch for H1/H2. Russian words are long — use `hyphens: auto` with `lang` attribute set correctly, `text-wrap: balance` on headings, `text-wrap: pretty` on paragraphs. Test every heading in both languages at 320 px — no overflow, no single-word orphans in H1.

## Grid & spacing

- Container: max 1440 px, side padding `--gutter-x`: 16 px (≤ 599), 24 px (600–1023), 40 px (≥ 1024).
- Columns: 4 (mobile) / 8 (tablet) / 12 (desktop), column gap 16 / 20 / 24 px.
- Spacing scale (`--s-1`…`--s-10`): 4, 8, 12, 16, 24, 32, 48, 72, 112, 160 px.
- Block vertical padding: `clamp(72px, 12vw, 160px)`.
- **Hairlines everywhere instead of boxes:** section tops, list rows, card borders are 1 px `--line`. Corners: 0 on blocks and lists; 14 px on window frames and cards; full-round on buttons and tags.
- Every block starts with a **block header row**: mono label left (`01 — УСЛУГИ`), optional link right, hairline under it.

## Blocks & theme switching

- Each `<section class="block" data-theme="dark|light">` sets its own tokens; children just use `var(--bg)`, `var(--fg)`.
- Adjacent blocks always alternate theme. The switch is hard (no gradient between blocks); what makes it "smooth" is the header re-theming and the reveal described in `03-interactions.md`.
- Page rhythm: Home starts dark; Services, Projects, Team, About start dark; Contact starts light. The footer is always dark.

## Imagery

- **No stock photos, no illustrations of robots/brains/code.**
- **Team photos:** one treatment for all — 4:5 crop, face at the same height (eyes at ~38 % from top), grayscale with contrast 1.1, placed on `--bg-sunken`. On hover/focus (desktop) the photo gains colour over 400 ms. Export `.avif` + `.webp` at 480/800/1200 px via `sharp`.
- **If a team photo is missing:** render a monogram tile (initials in Inter Tight 600, `--fs-h2`, on `--bg-sunken`, hairline border). Never a generic avatar icon.
- **Project photos:** every project has a real photo from the old site (`src/assets/projects/<id>.<ext>`, downloaded by `scripts/fetch-project-photos.mjs`, file name in `projects.json → photo`). Same treatment as the team: grayscale with contrast 1.1, colour on hover/focus over 400 ms, `object-fit: cover` in a 4:3 card (optional `photoFocus` = CSS `object-position` per project), exported as `.avif` + `.webp` at widths up to the original (never upscaled). A small window-bar-style chip sits in the top-left corner of the photo: the project monogram (Inter Tight 600) + the sector (mono), on a dark translucent background — it keeps the grid scannable. In the open project window (overlay) the photo is always in full colour — it is its own screen, not a preview; grayscale applies only to grid cards and the home “Projects” window.
- **If a project photo is missing** (`photo: null` or the file failed to download): fall back to the generated cover — the monogram set huge in Inter Tight (outlined), cropped by the card edge, plus a deterministic line pattern (seeded by the project id: parallel lines, concentric arcs or a dot grid — `hash(id) % 3`) in inline SVG with `--line-strong`. The home “Projects” window miniature shows 12 project photos from different sectors (smallest 480 px variant, grayscale → colour on window hover over `--dur-3`, small monogram chip), with this cover as the fallback.
- Client "logos" = typographic wordmarks (names in Inter Tight 500, uppercase, tracking 0.02em). Do not hotlink or fetch real logos.

## Components (visual spec)

- **Button primary:** height 52 px (mobile 56), padding 0 28 px, full-round, `--accent` fill, `--accent-ink` text, 15 px/500, trailing arrow `→` that slides 4 px right on hover. Magnetic (see interactions).
- **Button secondary:** same geometry, transparent, 1 px `--line-strong` border, `--fg` text; hover fills `--bg-sunken`.
- **Text link:** `--fg`, underline via `background-size` animation from left (1 px, `currentColor`).
- **Tag:** height 28 px, mono label size, 1 px `--line` border, full-round, padding 0 12 px.
- **Window frame** (signature element): radius 14 px, 1 px `--line` border, `--bg-raised` fill, 36 px title bar with three 8 px circles in `--line-strong` (left), mono path label centred (`cybermove / услуги`), optional close `×` right. Used for the home grid, project detail overlay, and the page-transition frame.
- **Accordion row:** full-width, top hairline, index (mono) + title (h3) + plus icon that rotates to ×; body expands with `grid-template-rows: 0fr → 1fr` transition (no height measuring in JS).
- **Inputs:** 56 px high, no box — only a bottom 1 px `--line-strong`, label floats above on focus/fill; focus line turns `--accent` (dark) / `--fg` (light). Chips for "what do you need" are multi-select tags.
- **Focus ring:** `outline: 2px solid var(--accent); outline-offset: 3px` on dark, `outline: 2px solid var(--fg)` on light. Never remove outlines.

## Motion tokens

```
--ease-out: cubic-bezier(0.16, 1, 0.3, 1);   /* expo-out, default */
--ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);
--dur-1: 160ms;  /* hovers */
--dur-2: 320ms;  /* small UI */
--dur-3: 560ms;  /* reveals */
--dur-4: 800ms;  /* page / window transitions */
```

## Logo

No logo file exists. Build the wordmark as text: `CYBERMOVE` in Inter Tight 600, tracking -0.02em, with the accent as a small 8×8 px square placed after the E (a "move forward" pixel) that nudges 3 px right on hover. Export it also as `public/logo.svg` (text converted to paths is not required — use the SVG `<text>` with the font as fallback-safe system stack in the SVG file) and generate the favicon from the accent square + "C".
