# 08 — Open questions and defaults

These facts are unknown. **Do not stop to ask.** Use the default, and list every item still open in `REPORT.md` (in Russian, as a checklist for the owner).

| # | Question | Default used by Claude Code |
|---|---|---|
| 1 | Production domain? (`company.json → domain`) | `https://cybermove.example` in `astro.config.mjs → site`; hreflang/canonical/sitemap use it. Owner changes one line and rebuilds. |
| 2 | Company email for contacts and for the form? (`contacts.json → email`, `formEmail`) | Not shown. Form works via WhatsApp only; `api/send.php` ships disabled (returns 503). |
| 3 | Telegram, Instagram, LinkedIn, office address? | Not shown (null). |
| 4 | Are the 6 team members and their photos real and up to date? Is Cyrillic spelling of names wanted on the RU version? | Use names in Latin in both languages, photos from the old site. |
| 5 | Brand name form: "CYBERMOVE" vs "CYBER MOVE"? | Wordmark `CYBERMOVE`; legal line in footer `CYBER MOVE CONSULTING`. |
| 6 | Real case results (numbers) for projects? | None shown. The project overlay shows description + tags only. Structure supports adding a `results: [{ value, label: {ru,en} }]` array per project later — implement the rendering (hidden when absent). |
| 7 | Project types: are VAN DOG AGENCY / Ukrainian Fashion Week / INTERTOP etc. direct clients or team experience? | As marked in `projects.json` (`type`). |
| 8 | Should politically sensitive projects (UDAR party) be shown? | Shown, as on the old site. Easy to hide: set `"hidden": true` in `projects.json` — implement support for this flag. |
| 9 | Kazakh language version? | Not built. Keep i18n code generic so adding `kk` is adding one JSON file + one locale in config. |
| 10 | Analytics (Yandex Metrica / Google Analytics)? | None. Leave a commented slot in `Base.astro` `<head>`, and a note that a cookie banner is then required. |
| 11 | Privacy policy legal review | Text from `content/i18n/*.json → privacy` with `{date}` = build date. Mark for lawyer review. |
| 12 | Logo | Text wordmark per design system. |
