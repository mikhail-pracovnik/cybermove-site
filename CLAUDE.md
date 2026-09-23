# CLAUDE.md — CYBERMOVE website build

You are building a brand-new marketing website for **CYBERMOVE** (legal/footer name: CYBER MOVE CONSULTING), a multidisciplinary business-growth consultancy: marketing, brand, AI automation, business development, tender and legal support.

The repository you are in currently contains only this kit (instructions + content). Your job is to turn it into a finished, working, production-ready static website — **end to end, without asking for confirmation between phases**. The owner will type one word ("делай") and expects a result he can open in a browser and upload to hosting.

## Read before writing any code (in this order)

1. `docs/01-brief.md` — who the company is, audience, tone, what went wrong with the old site
2. `docs/02-design-system.md` — tokens, typography, grid, colors, dark/light blocks
3. `docs/03-interactions.md` — the six signature interactions (precise specs + performance rules)
4. `docs/04-pages.md` — every page, section by section, and which content keys feed it
5. `docs/05-tech.md` — stack, project structure, i18n, SEO, forms
6. `docs/06-qa.md` — Definition of Done + the verification loop you must run
7. `docs/07-deploy-plesk.md` — how the build is deployed (you prepare the files, the owner uploads)
8. `docs/08-open-questions.md` — unknown facts and the defaults to use
9. `content/` — ALL site copy (RU + EN) and data. **Use it verbatim.** Do not invent copy.
10. `reference/` — background only (old-site audit, competitor notes, a brief from another AI). Read it for context; the docs above win on any conflict.

## Non-negotiable rules

- **Copy comes from `content/` only.** Never write marketing text yourself, never invent numbers, results, clients, testimonials, emails, addresses, prices or dates. If a field is `null`/empty, the UI element that needs it is not rendered (no placeholders like "lorem", "email@example.com", "+42% revenue").
- **No stock imagery, no AI-generated "robots/brains/glowing circuits" pictures.** Visual language = typography, thin lines, grids, generated SVG/CSS patterns, and the team's real photos treated uniformly (see design system).
- **The native cursor is never hidden or replaced.** The old site's lagging custom cursor was a top complaint. Only the small contextual label described in `03-interactions.md` is allowed.
- **Performance is a feature:** no GSAP, no Lenis/Locomotive, no three.js, no React/Vue/Svelte islands, no jQuery. Vanilla TypeScript + CSS. Animate only `transform`, `opacity`, `clip-path` (and SVG path `d` for the one footer effect). Total JS per page ≤ 35 KB gzipped.
- **`prefers-reduced-motion: reduce` must switch every animation to an instant or opacity-only state.** Everything must work with JS disabled (content visible, links work, form falls back to WhatsApp link).
- **Two languages, equal quality:** Russian (default, at `/`) and English (at `/en/`). The Russian version is fully Russian — no English headlines mixed in (a key flaw of the old site). Tags/terms like CRM, AI, SEO, CAC are fine.
- **Mobile-first.** Design at 375 px first, then scale up. No horizontal scroll at any width from 320 px. Tap targets ≥ 44 px.
- **Windows-friendly tooling:** the owner works on Windows. All scripts must be npm scripts or `.mjs` Node scripts — no bash-only commands, no `rm -rf` in package.json (use `node` scripts or cross-platform packages).

## Working method

1. Create a todo list from the phases below and keep it updated.
2. Work phase by phase. After each phase run `npm run build` and fix every error/warning before moving on.
3. Commit after every phase if git is available (`git init` if the folder is not a repo yet). Commit messages in English, short.
4. Do not stop to ask questions. When something is genuinely unknown (missing email, missing photo), follow `docs/08-open-questions.md`: take the documented default and list it in the final report.
5. At the end run the full verification loop from `docs/06-qa.md` (build, preview, Playwright screenshots on mobile + desktop in both languages, console-error check, link check). Look at the screenshots yourself and fix what looks wrong. Repeat until the Definition of Done is met.

## Phases

| # | Phase | Output |
|---|-------|--------|
| 0 | Environment check | `node -v` ≥ 20 (if missing, stop and tell the owner to install Node.js LTS from nodejs.org — this is the only allowed stop). |
| 1 | Scaffold | Astro project per `docs/05-tech.md`, fonts, tokens, base layout, header/footer, i18n plumbing, `content/` wired in via typed loaders. |
| 2 | Assets | Run `node scripts/fetch-assets.mjs` (downloads the 6 team photos from the old site into `src/assets/team/`). Render them with `astro:assets` `<Picture>` (avif/webp, 3 widths) and the design-system treatment (uniform 4:5 crop via `object-position` per member from `team.json → photoFocus`, grayscale via CSS). Look at each photo (open the file) to set `photoFocus` so eyes line up across portraits. If the download fails, use the monogram fallback described in the design system and note it in the report. |
| 3 | Components | Buttons (magnetic), section shell with `data-theme`, window frame, tag, accordion, project card, team card, stat, marquee, form controls. |
| 4 | Pages | Home, Services, Projects, Team, About, Contact, Privacy, 404 — RU and EN. |
| 5 | Signature interactions | All six from `docs/03-interactions.md`. |
| 6 | SEO & meta | Titles/descriptions from content, hreflang, canonical, sitemap, robots, OG image (generate one static 1200×630 PNG with the wordmark on the dark palette via a Node script using `sharp` + SVG), favicon set, JSON-LD Organization. |
| 7 | Hosting files | `public/.htaccess`, optional `public/api/send.php`, deploy notes. |
| 8 | QA loop | `docs/06-qa.md` in full. |
| 9 | Report | Write `REPORT.md` in Russian for the owner: what was built, how to preview (`npm run preview`), how to upload, list of open questions/defaults used, and screenshots paths. |

## Quality bar

The reference level is work.co, area17.com, rga.com, frog.co, ourama.fr: expensive restraint, large confident typography, generous whitespace, precise thin lines, motion that feels physical and never slows the user down. If a section feels crowded, remove things before shrinking type. When in doubt, choose the calmer option.
