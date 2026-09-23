# 04 — Pages

All copy lives in `content/i18n/{ru,en}.json` (UI + page text) and `content/data/*.json` (structured, bilingual `{ "ru": …, "en": … }` fields). Keys referenced below are paths in those files, e.g. `home.hero.title` → `content/i18n/ru.json → home → hero → title`.

## URL map

| Page | RU | EN | Starts |
|---|---|---|---|
| Home | `/` | `/en/` | dark |
| Services | `/services/` | `/en/services/` | dark |
| Projects | `/projects/` | `/en/projects/` | dark |
| Team | `/team/` | `/en/team/` | dark |
| About | `/about/` | `/en/about/` | dark |
| Contact | `/contact/` | `/en/contact/` | light |
| Privacy | `/privacy/` | `/en/privacy/` | light |
| 404 | `/404.html` (bilingual: RU text + EN link) | | dark |

Trailing slashes always. Old-site URLs `/approach`, `/partners`, `/services/*` must 301 to the new ones (see `docs/07-deploy-plesk.md`).

Every page = Header (capsule nav) → blocks → Footer (with bending text). Each block: `data-theme`, `data-label` (for the nav capsule), a block header row (mono index + label).

Keep it lean: **no block may have more than one heading, one lead sentence and one visual/list.** If content doesn't fit that rule, it belongs on an inner page.

---

## Home — `/`

1. **Hero** (dark, min-height 100svh)
   - Mono label `home.hero.label`
   - H1 `home.hero.title` in `--fs-h1` (reveal by lines)
   - Lead `home.hero.lead` (max 2 lines on desktop)
   - Buttons: primary `home.hero.ctaPrimary` → `/contact/?need=audit`; secondary `home.hero.ctaSecondary` → `/projects/`
   - **Window grid** (interaction #1) with 5 windows; titles from `home.windows.*`. Desktop layout (12 cols): Projects 7 cols × 2 rows, Services 5 cols, Team 5 cols, About 4 cols, Contact 3 cols… choose a balanced asymmetric bento; mobile: stacked.
2. **Clients marquee** (dark, continuous with hero — no theme switch, separated by a hairline)
   - Mono label `home.clients.label`; two marquee rows of all `projects.json` names where `showInMarquee: true`; link `home.clients.link` → `/projects/`.
3. **Pains → answers** (light)
   - H2 `home.pains.title`. List of 6 quotes (`home.pains.items[].quote`) as large typographic rows. Clicking/tapping a row expands it to show `answer` + a tag linking to the service (`/services/#<service>`). One open at a time. This replaces the old 12-quote wall.
4. **Services index** (dark)
   - H2 `home.services.title`. 7 rows from `services.json`: mono index `00`–`06`, title in `--fs-h3`, lead in `--fg-muted` (hidden on mobile until tapped). Desktop hover: row background `--bg-sunken`, arrow slides in, a tiny window frame preview of the includes list appears on the right, attached to the row (not to the cursor). Each row → `/services/#<id>`.
5. **Method** (light)
   - H2 `home.method.title`. 4 steps (`home.method.steps`). Desktop: sticky left column with the step counter `01/04` and a thin progress line; right column scrolls through the 4 steps (each ~70 vh), the active step is full contrast, others muted. Mobile: simple vertical list with hairlines.
6. **Sectors** (dark) — interaction #3
   - H2 `home.sectors.title`, lead `home.sectors.lead`; sectors from `sectors.json`; names from `projects.json`.
7. **Team teaser** (light)
   - H2 `home.team.title`, lead `home.team.lead`; 6 portraits in a single row (desktop) / 2×3 grid (mobile), name + role under each; link `home.team.link` → `/team/`.
8. **Final CTA** (dark)
   - H2 `home.cta.title` in `--fs-h1`, lead, primary → `/contact/?need=audit`, secondary WhatsApp link (`contacts.json`).
9. Footer.

## Services — `/services/`

1. **Hero** (dark): label, H1 `services.hero.title`, lead.
2. **Audit — the entry point** (light): H2 `services.audit.title`, lead; left: "What we look at" = `services.json[id=audit].includes` as tags; right: 4 deliverables `services.audit.deliverables` as numbered hairline rows; primary CTA `services.audit.cta` → `/contact/?need=audit`.
3. **Directions** (dark): H2 `services.directions.title`. Accordion of the 6 other services (`services.json` minus audit). Each item: index, title, lead; expanded body: "What's included" (`includes` as 2-column list), "What you get" (`outcome`), and a "Discuss" link → `/contact/?need=<id>`. `id` anchors: `#marketing #brand #ai #bizdev #tender #legal`. The `#ai` item additionally contains the **automation flow**: 6 steps from `services.aiFlow` drawn as a horizontal chain of small window frames connected by hairline arrows (mobile: vertical).
4. **Automation estimator** (light) — see "ROI estimator" below.
5. **Formats** (dark): H2 `services.formats.title`; 5 cards `services.formats.items` in a horizontal scroll-snap row (mobile) / 5-column grid (desktop).
6. **FAQ** (light): accordion `services.faq.items`.
7. **CTA** (dark) → contact.

### ROI estimator (`src/components/Estimator.astro` + `src/scripts/estimator.ts`)
- Copy: `services.estimator.*`.
- Inputs (range sliders, custom-styled, with number readouts and ± buttons for touch):
  - People doing routine work: 1–200, default 10
  - Hours of routine per person per week: 1–30, default 8
  - Loaded hourly cost: currency toggle ₸ / € / $ (default ₸ on RU, € on EN); ranges ₸ 1 000–20 000 (default 3 000), € 8–80 (default 20), $ 8–90 (default 22)
- There is no “share” slider: the automatable share is a fixed 25 % (the team’s practice), stated in the formula line (`services.estimator.formula`).
- Output (count-up animation, `tabular-nums`):
  - Hours freed per month = people × hours/week × 4.33 × 0.25
  - Value per month = hours × hourly cost
  - Equivalent FTE = hours / 160
- Show the formula in a small mono line under the results and the disclaimer `services.estimator.disclaimer`. CTA `services.estimator.cta` → `/contact/?need=ai`, and pass the computed numbers in the query (`&est=<hours>h`) so the contact form pre-fills the message.
- Number formatting via `Intl.NumberFormat` for the current locale. No external APIs.

## Projects — `/projects/`

1. **Hero** (dark): label, H1 `projects.hero.title`, lead `projects.hero.lead`, small note `projects.hero.note` (explains that the list includes clients, projects the team worked on inside agencies, and own ventures — honesty matters).
2. **Filter + grid** (dark, same block): sector chips from `sectors.json` + "All" (`projects.filterAll`), with counts. `?sector=<id>` preselects a chip on load; changing the chip updates the URL via `history.replaceState` (no parameter for "All"). Grid: 3 cols desktop / 2 tablet / 1 mobile. Filtering uses interaction #3 (FLIP).
   - **Card:** project photo with the team treatment (grayscale → colour on hover/focus) and a corner chip with monogram + sector (see design system; generated cover only as fallback when a photo is missing), name (h3), 1-line description, tags. A small mono badge when `type` is `experience` (`projects.typeLabels.experience`) or `own` (`projects.typeLabels.own`).
   - Click → **project window overlay**: a window frame (interaction #1 styling) grows from the card (shared-element transition) to a centered 720 px window (mobile: full-screen sheet). Content: name, sector, geography (if any), full description `details`, tags, external link `projects.overlay.visit` if `url` is not null (opens in new tab, `rel="noopener"`). Close: ×, Esc, backdrop click; focus returns to the card. URL updates to `?p=<id>` (so a project is shareable) — opening the page with `?p=<id>` opens the overlay.
   - **Never** show "view case" links to pages that don't exist.
3. **CTA** (light).

## Team — `/team/` (keep the old page's structure, restyle)

1. **Hero** (dark): label, H1 `team.hero.title`, lead.
2. **Members** (one dark block): for each member of `team.json`, a row on a 12-col grid: portrait (5 cols, 4:5), content (6 cols, offset 1): focus line (mono, `focus`), name (h2), role, bio (lead size), 4 tags. Rows alternate portrait left/right on desktop. Mobile: portrait full width, content under it. Portrait uses the design-system treatment (grayscale → colour on hover/focus-within). Reveal on scroll.
3. **CTA** (light): H2 `team.cta.title`, lead, WhatsApp button (`contacts.json`), secondary → `/contact/`.

## About — `/about/`

1. **Hero / manifesto** (dark): label, H1 `about.hero.title`, two short paragraphs `about.hero.paragraphs`.
2. **Principles** (light): 3 columns `about.principles.items` (index, title, text).
3. **The Core** (dark): H2 `about.core.title`, lead; 8 elements `about.core.items` as a 4×2 grid of hairline cells (mobile 2×4); each cell: index, title, one-line text. On desktop, hovering a cell highlights it and draws an accent line from the cell to a central "CORE" label (simple SVG line, computed once on resize).
4. **AI stance** (light): H2 `about.ai.title`, lead, the chain `about.ai.chain` (CORE → PROCESS → DATA → AUTOMATION → AI) as a mono line with arrows that light up sequentially on reveal.
5. **What we measure** (dark): H2 `about.metrics.title`, 10 metric names `about.metrics.items` as large tags in a wrapping cloud, note `about.metrics.note`.
6. **Where we work** (light): H2 `about.geo.title`, lead; the 13 cities of `geography.json` as a 3-column typographic list grouped by region (EU / CIS & Central Asia / APAC — region labels in `geography.json`), each city with its country. Optional minimal dot-map (hand-placed SVG dots on an equirectangular grid of dots — no map library, no map tiles). Hover/tap a city highlights its dot.
7. **CTA** (dark).

## Contact — `/contact/`

1. **Hero + form** (light): H1 `contact.hero.title`, lead. Left column: channels — WhatsApp (big, primary), and any other channel present in `contacts.json` (email, telegram, instagram, address) — **render only non-null ones**. Right column: form.
   - Fields: name, company, contact (email / Telegram / WhatsApp), "What do you need?" chips (multi-select, `contact.form.needs`), message.
   - `?need=<id>` preselects a chip; `?est=` prefills the message with `contact.form.estimatePrefill`.
   - Submit behaviour — see `docs/05-tech.md → Forms`. Default: opens WhatsApp with a prefilled, nicely formatted message (as the old site did), after client-side validation.
   - Consent line with link to Privacy.
2. **Geography strip** (dark): one line of the 13 cities in mono, scrolling marquee.
3. Footer.

## Privacy — `/privacy/`
Plain, readable legal page from `privacy.*`. Light theme, 60ch column. Mark in REPORT.md that it needs a lawyer's review.

## 404
Dark, H1 huge "404", lead from `notFound.*` (RU with EN underneath), link home. Bending text footer included.

## Footer (all pages, dark)
- Top: bending text (interaction #4), links to `/contact/`.
- Grid: wordmark + tagline `footer.tagline`; Services list (7 links to `/services/#id`); Company (Projects, Team, About, Contact); Contact (WhatsApp + any non-null channels); language switch.
- Bottom row (mono, `--fg-muted`): `© <current year> CYBER MOVE CONSULTING` · `footer.privacy` link · "Back to top ↑".
