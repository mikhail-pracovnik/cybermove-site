# 05 — Technical spec

## Stack

- **Astro** (latest stable, `output: 'static'`) — used purely as a static-site generator: components, layouts, i18n routing, image optimisation, bundling. Output is plain HTML/CSS/JS in `dist/` that runs on any hosting (Plesk/Apache/nginx) without Node.
- **TypeScript** for scripts (`<script>` tags in `.astro` files, bundled by Astro/Vite). No UI framework integrations.
- **Plain CSS** with custom properties and `@layer reset, tokens, base, components, utilities, transitions;`. No Tailwind, no CSS-in-JS.
- Fonts: `@fontsource-variable/inter-tight`, `@fontsource-variable/inter`, `@fontsource-variable/jetbrains-mono`.
- Images: `astro:assets` `<Picture>` (formats `['avif','webp']`, widths `[480, 800, 1200]`).
- `@astrojs/sitemap` with i18n config.
- Dev-only: `@playwright/test` for the QA loop, `sharp` (already a dependency of Astro) for the OG image script.
- **Forbidden:** GSAP, Lenis, Locomotive, Barba, Swup, three.js, jQuery, React/Vue/Svelte/Solid, Tailwind, any analytics/tracking script, any CDN-hosted asset, Google Fonts links.

## Project structure

```
/
├─ astro.config.mjs
├─ package.json
├─ tsconfig.json
├─ content/                 ← from this kit (source of truth, do not move)
│  ├─ i18n/ru.json, en.json
│  └─ data/*.json
├─ scripts/
│  ├─ fetch-assets.mjs      ← from this kit
│  └─ og-image.mjs          ← you write it
├─ public/
│  ├─ .htaccess
│  ├─ robots.txt
│  ├─ favicon.svg, favicon.ico, apple-touch-icon.png, og.png
│  └─ api/send.php          ← optional, see Forms
├─ src/
│  ├─ assets/team/*.jpg     ← downloaded photos
│  ├─ lib/
│  │  ├─ i18n.ts            ← t(lang, key), route helpers, alternate URL
│  │  └─ content.ts         ← typed loaders for content/data/*.json
│  ├─ layouts/Base.astro    ← <html lang>, meta, hreflang, fonts, header, footer
│  ├─ components/           ← Header, Footer, Block, WindowFrame, Button, Tag, Accordion,
│  │                          ProjectCard, ProjectOverlay, TeamMember, Marquee, SectorMorph,
│  │                          MethodSteps, Estimator, ContactForm, BendText, ProjectPhoto, Cover (generated SVG fallback)
│  ├─ pages/
│  │  ├─ index.astro, services.astro, projects.astro, team.astro, about.astro,
│  │  │  contact.astro, privacy.astro, 404.astro
│  │  └─ en/ (same set, thin wrappers passing lang="en")
│  ├─ scripts/
│  │  ├─ core/raf.ts, core/spring.ts, core/observe.ts, core/motion.ts (reduced-motion flag)
│  │  ├─ nav.ts, transitions.ts, reveal.ts, magnetic.ts, cursor-label.ts,
│  │  │  sector-morph.ts, overlay.ts, bend.ts, estimator.ts, form.ts, marquee.ts, lang.ts
│  └─ styles/ tokens.css, base.css, components.css, transitions.css
└─ tests/ qa.spec.ts
```

Pages are implemented once as components that take `lang` (`src/components/pages/HomePage.astro` etc.); `src/pages/*.astro` and `src/pages/en/*.astro` are one-liners that render them.

## i18n

- Astro `i18n: { defaultLocale: 'ru', locales: ['ru', 'en'], routing: { prefixDefaultLocale: false } }`.
- `t(lang, 'home.hero.title')` reads from `content/i18n/<lang>.json`; data fields `{ ru, en }` resolved by a `pick(lang, field)` helper. A missing key must **fail the build** (throw), not render the key.
- `<html lang="ru">` / `lang="en"` — required for correct hyphenation.
- In `<head>`: `<link rel="alternate" hreflang="ru" href="https://<domain>/…">`, `hreflang="en"`, `hreflang="x-default"` (→ RU), and `<link rel="canonical">`. The domain comes from `astro.config.mjs → site`; use the value in `content/data/company.json → domain` (if null, use `https://cybermove.example` **and list it in REPORT.md as the #1 thing to change** — it's a one-line edit in `astro.config.mjs`).
- Language switch = normal link to the alternate URL (see interactions for scroll restore). No client-side text swapping — each language is its own static HTML page (correct for SEO and no layout jump).
- Month/number formatting via `Intl` with `ru-RU` / `en-GB`.

## SEO

- Per-page `<title>` and `<meta name="description">` from `meta.<page>` in the i18n file.
- Open Graph + Twitter tags; `og:locale` `ru_RU` / `en_US` + `og:locale:alternate`.
- JSON-LD `Organization` on every page (name, url, logo, `contactPoint` with the WhatsApp phone, `areaServed` from geography countries, `sameAs` from non-null social links). `Person` items for team on `/team/`.
- `robots.txt` allowing all, with sitemap URL.
- Semantic HTML: one `h1` per page, landmarks (`header`, `nav`, `main`, `footer`), lists as lists.

## Forms

`content/data/contacts.json` controls behaviour:
- `whatsapp` (always present): the form, after validation, opens `https://wa.me/<number>?text=<encoded message>` in a new tab. Message template: `contact.form.waTemplate` (lines: name, company, contact, needs, message). Show a success note `contact.form.success` after opening.
- `formEmail` (currently `null`): if set, the form additionally `POST`s JSON to `/api/send.php`. Write `public/api/send.php` anyway so it's ready:
  - Accept only POST + `application/json`, max 10 KB body.
  - Honeypot field `website` must be empty; reject submissions faster than 3 s after page load (timestamp field).
  - Validate and sanitise every field (`strip_tags`, length limits, email/phone format for contact if it looks like one), no header injection (no user input in headers except a validated `Reply-To`).
  - Send with PHP `mail()` to `FORM_EMAIL` defined at the top of the file (read from the same value — put a clear `// TODO: set recipient` comment and have it return 503 while empty).
  - Return JSON `{ ok: true }` / `{ ok: false, error }` with proper status codes.
  - On the client: if the POST fails, fall back to the WhatsApp flow — the lead must never be lost.
- Without JS the form's `action` is the WhatsApp link (`https://wa.me/<number>`), so something always happens.

## Accessibility

- WCAG 2.2 AA contrast for all text (check `--fg-muted` on both themes).
- Visible focus on everything; skip link `nav.skip`.
- Nav menu: focus trap, Esc closes, `aria-expanded`, `aria-controls`.
- Overlays: `role="dialog"`, `aria-modal`, labelled by the title, focus restore.
- Sliders in the estimator: native `<input type="range">` (styled), with `aria-valuetext` including units.
- Everything animated has a reduced-motion path (see interactions).

## Performance budgets (mobile, Lighthouse, 4× CPU throttle)

- LCP < 2.0 s, CLS < 0.02, TBT < 150 ms, INP < 200 ms.
- JS ≤ 35 KB gz per page, CSS ≤ 30 KB gz, fonts ≤ 180 KB total (subsetted variable woff2, only the scripts needed).
- No layout shift from fonts: set `size-adjust`/metric overrides or use `font-display: swap` with a matched fallback (`Arial` with `ascent-override` etc.) — Fontsource provides fallback metrics; use them.
- Images lazy except the first visible portrait; `fetchpriority="high"` for the hero-critical font only.
