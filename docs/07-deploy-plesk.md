# 07 — Deploy to Plesk

The owner hosts on **Plesk** (Apache + nginx proxy is the Plesk default). The site is fully static.

## What you (Claude Code) prepare

1. `npm run build` → `dist/`.
2. `public/.htaccess` (copied into `dist/` by the build) containing:
   - Force HTTPS and a single host (non-www → keep as is unless `company.json → domain` says otherwise; write the rule generically with a comment on how to switch).
   - `ErrorDocument 404 /404.html`
   - `DirectorySlash On`, `DirectoryIndex index.html`
   - 301 redirects from old-site URLs:
     - `/approach` → `/about/`
     - `/partners` → `/projects/`
     - `/services/<anything>` (old sub-pages like `/services/audit`) → `/services/`
     - `/marketing`, `/brand-development` → `/services/`
     - add trailing slash to known page paths without one
   - Caching: `Cache-Control: public, max-age=31536000, immutable` for `/_astro/*` (hashed) and fonts; `max-age=0, must-revalidate` for `.html`.
   - Compression: `mod_deflate` for text types (Plesk's nginx usually gzips too — harmless).
   - Security headers: `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`, a CSP that allows only self (+ `https://wa.me` for navigation is not needed in CSP). `frame-ancestors 'self'`.
   - Correct MIME for `.avif`, `.webp`, `.woff2`.
3. `npm run zip` script (Node, cross-platform, e.g. using the `archiver` or `adm-zip` dev dependency) that produces `cybermove-dist.zip` with the **contents** of `dist/` at the zip root.

## What the owner does (write these steps, in Russian, into REPORT.md)

1. Plesk → Websites & Domains → the domain → **File Manager** → open `httpdocs`.
2. Delete the default files (index.html, etc.) there.
3. Upload `cybermove-dist.zip` → right click → **Extract files** → delete the zip.
4. Plesk → **SSL/TLS Certificates** → issue a free Let's Encrypt certificate (if not yet), enable "Redirect from HTTP to HTTPS".
5. If the domain differs from `astro.config.mjs → site`, change it there, rebuild, re-upload.
6. If Plesk uses **nginx-only** hosting (no Apache): `.htaccess` is ignored — then paste the nginx equivalent (which you also generate as `deploy/nginx.conf.txt`) into Plesk → Apache & nginx Settings → Additional nginx directives.
7. Form email (optional): once a recipient email is known, set it in `content/data/contacts.json → formEmail` and in `api/send.php`, rebuild/re-upload. Check that PHP is enabled for the domain (Plesk → PHP Settings) and mail sending works.
