# 03 — Signature interactions

Six interactions define the site. They are **inspired by** work.co, area17.com and ourama.fr — do not copy their markup, visuals or exact choreography. Each one below has: idea → behaviour → implementation → mobile → reduced motion → budget.

Global performance rules (apply to everything here):
- Animate only `transform`, `opacity`, `clip-path`. The single exception is the SVG path in #4.
- All scroll-driven work goes through **one** shared `requestAnimationFrame` loop (`src/scripts/core/raf.ts`) that runs only while something needs it and sleeps otherwise. Scroll listeners are `passive` and only set a dirty flag.
- Prefer native: CSS scroll-driven animations (`animation-timeline: view()`) where supported, with an IntersectionObserver fallback; View Transitions API for page and state transitions.
- Pointer effects only on `(hover: hover) and (pointer: fine)`.
- `will-change` is added right before an animation and removed after it.
- No layout reads inside the rAF write phase (read all rects first, then write).

---

## 1. Window navigation (inspired by work.co/grid)

**Idea:** the site behaves like a set of windows. Sections of the site are "windows" you open; pages open on top of each other like a stack, and you can always see where you came from.

**Behaviour**
1. **Home "window grid" (hero, below the headline):** 5 window frames in an asymmetric bento grid — *Услуги, Проекты, Команда, О компании, Контакты*. Each window has a title bar (`cybermove / проекты`) and a live-looking miniature of that page: for Projects — 12 scrolling project photos from different sectors (grayscale, colour on hover, monogram chip); for Team — 6 grayscale portraits in a 3×2 grid; for Services — the 7 service names in mono with an index; for About — the 4-step method line; for Contacts — one row per non-empty channel (WhatsApp, Telegram, email: mono label left, value right) with a blinking caret on the last row; the whole window is one link, no nested links. Miniatures are real HTML/CSS (no screenshots).
2. Hovering a window (desktop): it lifts (`translateY(-4px)`), its border goes `--line-strong`, the other windows dim to 0.6 opacity.
3. Clicking a window navigates to that page with a **shared-element transition**: the window frame expands to fill the viewport and becomes the new page's frame; the title bar travels to the top and stays as the page's "window bar" for 800 ms, then fades.
4. **Every page-to-page navigation** uses the window transition: the old page scales to 0.94, darkens (overlay to 40 %), and moves back; the new page slides up from 12 vh with a 14 px top radius and a title bar showing its path, then the radius flattens to 0 as it settles.
5. **Tab strip:** the header keeps a compact strip of the last 3 visited pages as small "tabs" (mono labels). Clicking a tab navigates back to that page with the reverse animation (new page comes in from behind, old page slides down). Stored in `sessionStorage`; hidden on mobile.

**Implementation**
- Cross-document **View Transitions API**: in global CSS `@view-transition { navigation: auto; }`.
- `view-transition-name: page` on `<main>`; `view-transition-name: win-<slug>` on each home window and on the target page's `<main>` (set via `pageswap`/`pagereveal` events: on `pageswap` read the clicked link's `data-win` and set `view-transition-name` on it; on `pagereveal` set the same name on `main`). Direction (forward/back) via `document.documentElement.dataset.navDir` computed from the tab-strip history, and CSS `:active-view-transition-type(forward|back)`.
- Keyframes live in `src/styles/transitions.css`, using `::view-transition-old(page)` / `::view-transition-new(page)`.
- Do **not** use Astro's `<ClientRouter />` — native cross-document transitions keep every page a normal HTML document (better for Plesk, SEO, no script re-init bugs).

**Fallback:** browsers without cross-document view transitions (check `CSS.supports('view-transition-name: a')` + `'onpagereveal' in window`) get a 240 ms opacity fade-in of `main` on load only. Navigation always works as plain links.

**Mobile:** grid becomes a vertical stack of 5 windows with 16:10 miniatures; the transition is the same slide-up but 560 ms and no scale-back of the old page (keeps it cheap).

**Reduced motion:** transitions become a 150 ms crossfade. Hover lift off.

**Budget:** transition script ≤ 3 KB gz.

---

## 2. Morphing nav capsule (inspired by ourama.fr)

**Idea:** the navigation is a single floating capsule at the top that grows and changes as you scroll — it always tells you where you are and gives one tap to the next action.

**States**
- **A — top of page:** full-width transparent header: wordmark left; nav links centre (Услуги, Проекты, Команда, О компании, Контакты); right: RU/EN switch + primary CTA "Начать с аудита".
- **B — scrolled > 80 px, scrolling down:** header collapses into a centred **capsule** (max-width 560 px, height 52 px, full-round, `--bg-raised` at 82 % with `backdrop-filter: blur(16px) saturate(140%)`, 1 px `--line`). Content: wordmark (compact "C▪" mark) · **current section label** (mono, e.g. `02 / МЕТОД`) · a 2 px accent progress line along the bottom edge showing page progress · menu button.
- **C — scrolling up, or pointer hovers the capsule:** capsule widens (to max 880 px) and reveals the nav links inline with a staggered fade (30 ms step); the section label slides out. Returns to B after 1.5 s without interaction when scrolling down again.
- **D — menu open:** the capsule expands downward into a panel (clip-path from capsule shape to a 14 px-radius sheet of ~70 vh on mobile, 420 px tall on desktop): big nav links (`--fs-h2`), language switch, WhatsApp link, CTA. Esc / outside click / link click closes it. Focus is trapped inside while open; `aria-expanded` on the button.

**Theme awareness:** the capsule re-themes to whatever block is under it. Every block has `data-theme`; an IntersectionObserver with `rootMargin: "-26px 0px -95% 0px"` (a thin line at capsule height) sets `header.dataset.theme`. The swap animates colours over `--dur-2`.

**Section label:** each block declares `data-label="02 / МЕТОД"` (localised). The same observer updates the label with a vertical slot-machine swap (old label `translateY(-100%)`, new from `100%`).

**Implementation:** one component `Header.astro` + `src/scripts/nav.ts`. States are classes on `<header>`; geometry changes via `transform: scaleX` on a background layer + `clip-path` so no layout is thrashed; the links live in the DOM always (accessible), hidden visually in state B with `opacity`/`visibility`.

**Mobile:** states A → B (capsule is full width minus 16 px margins, 56 px high); state C is skipped; tapping the capsule opens D as a bottom-anchored sheet that fills 85 vh (thumb-friendly). Links ≥ 56 px tall.

**Reduced motion:** state changes are instant; no slot-machine label.

---

## 3. Sectors → projects morph (inspired by area17.com)

**Idea:** one block shows the industries the team has worked in; switching sector smoothly rearranges the names of the brands underneath — the page feels alive and shows breadth without a wall of text.

**Behaviour (Home block "Отрасли", Projects page filter)**
- Left (desktop) / top (mobile): list of 7 sectors from `content/data/sectors.json` in `--fs-h3`, with count in mono (`08`). Active sector is `--fg`, others `--fg-muted`; a 2 px accent bar slides vertically to the active one.
- Right / below: the project names of the active sector as large typographic wordmarks, wrapping, separated by ` · ` in `--fg-muted` (not ` / ` — it clashes with names like “USYK / Александр Усик”). Each name links to `/projects/?p=<id>`; under them a link `home.sectors.allLink` → `/projects/?sector=<id>`.
- Sector names are links to `/projects/?sector=<id>`: on desktop hover previews and click navigates; on touch a tap previews (the “all projects” link navigates).
- Switching: on desktop on hover/focus of a sector (with 120 ms intent delay), on mobile by tap (horizontal scroll-snap chips). Names that stay keep their position and **morph** to the new layout; leaving names drop out with `opacity 0 + translateY(8px)`; entering names rise in with a 20 ms stagger. The sector's short description (one line) crossfades.
- Autoplay: on Home, if the user hasn't interacted, the block cycles sectors every 3.5 s while in view (paused off-screen, on hover, on focus, and permanently after first interaction).
- On the **Projects page** the same mechanism filters the card grid: cards reposition via FLIP.

**Implementation:** same-document View Transitions — `document.startViewTransition(() => render(sector))`, each wordmark/card with `view-transition-name: p-<id>`. Fallback: manual FLIP (read rects → update DOM → invert with transform → play) in ≤ 60 lines. Limit simultaneously animated items to 40.

**Reduced motion:** instant swap with a 150 ms opacity fade.

---

## 4. Bending text finale (inspired by area17.com)

**Idea:** the last thing on every page is a playful, elastic line of huge type that reacts to you — a memorable sign-off.

**Behaviour**
- In the footer (dark), full-bleed: the localised phrase from `footer.bend` (RU «Двигаем бизнес вперёд →», EN "We move business forward →") set in `--fs-mega`, Inter Tight 600, as an SVG `<text>` on a `<textPath>`.
- The path is a horizontal line with one quadratic control point. **Scroll velocity** bends it: fast scroll down → the line sags into a smile (control point y + velocity × k, clamped to ±18 % of the font size); it springs back to straight with a damped spring (stiffness 170, damping 18) when scrolling stops.
- **Pointer (desktop):** when the pointer is over the footer, the control point's x follows the pointer and y pulls toward it by up to 12 % — the text "stretches" toward the cursor like a rubber band. Leaving the footer springs it back.
- **Tap (mobile):** a tap makes a single "pluck" — the line wobbles and settles (spring from ±14 %).
- The phrase is a link to Contact. Under it, a plain accessible `<a>` with the same text is present for screen readers (`aria-hidden="true"` on the SVG).
- Additionally the text slowly scrolls horizontally along the path (`startOffset` loops, 40 s per cycle) so long RU text is fully readable on mobile.

**Implementation:** `src/scripts/bend.ts`. Only runs while the footer is intersecting (IntersectionObserver starts/stops it in the shared rAF). Per frame: integrate spring → write one `d` attribute string + one `startOffset`. Nothing else.

**Reduced motion:** static straight text, no loop, no spring.

**Budget:** ≤ 2 KB gz; must hold 60 fps on a mid-range Android (test with 4× CPU throttle in Playwright/DevTools — no long tasks > 50 ms).

---

## 5. Block switching & text reveal

**Idea:** the dark/light alternation is the site's heartbeat. Each switch should feel like a deliberate cut, not a scroll of random boxes.

**Behaviour**
- As a new block enters, its top edge is revealed with `clip-path: inset(8% 0 0 0 round 14px 14px 0 0)` → `inset(0 0 0 0 round 0)` tied to scroll progress (the block "slides over" the previous one like a sheet). Use CSS scroll-driven animations: `animation-timeline: view(); animation-range: entry 0% entry 40%;`. Fallback (no support): no clip animation at all — just the hard cut.
- **Headings:** split into lines (not letters) at build time where possible, or at runtime by wrapping words in spans and grouping by `offsetTop` once after fonts load (`document.fonts.ready`) and on resize (debounced 150 ms). Each line rises from `translateY(105%)` inside an `overflow: clip` wrapper, stagger 70 ms, `--dur-3`, `--ease-out`. Triggered once by IntersectionObserver (threshold 0.2).
- **Mono labels and small text:** fade + 8 px rise.
- **Numbers** (counts in sectors, the ROI estimator results): count up with `requestAnimationFrame` over 900 ms, `tabular-nums` so width doesn't jump.
- Without JS: everything visible (reveal classes are only added by JS: `html.js .reveal { … }`).

**Reduced motion:** no transforms, just opacity 0 → 1 over 200 ms.

---

## 6. Magnetic buttons & contextual cursor label (no lag)

**Magnetic buttons** (primary + secondary buttons, window frames on home, social/WhatsApp icons):
- Within a radius of 1.4 × the button's half-width, the button translates toward the pointer by up to 30 % of the distance (max 10 px), its label by an extra 40 % of that (parallax inside the button).
- Easing: critically damped spring **with a fast response** (lerp factor 0.35 per frame at 60 fps, frame-rate corrected: `1 - Math.pow(1 - 0.35, dt / 16.67)`), so the button reacts within ~3 frames. Returns to 0 on leave.
- Read the button rect once on `pointerenter` (and on resize/scroll invalidation), never on every `pointermove`.

**Contextual cursor label** (the only cursor-related effect allowed):
- The **native cursor stays visible and unchanged.** Over project cards and home windows, a small pill (`Открыть →` / `Open →`, mono 12 px, accent fill) appears **attached to the pointer** at offset (+16 px, +16 px).
- It follows the pointer with **zero interpolation**: position is set directly from the latest `pointermove` coordinates in the next rAF via `transform: translate3d()`. No lerp, no delay — this fixes the old site's lag.
- Appears/disappears with scale 0.8 → 1 + opacity over `--dur-1`. Hidden on touch devices entirely.

**Reduced motion:** magnetic effect off; label still appears (it's informational) but without scale.

---

## Also required (small interactions)

- **Marquee of client names** (Home, below hero): two rows moving in opposite directions via CSS `@keyframes` on a duplicated track, 60 s per loop; pauses on hover and when not in view (`animation-play-state` toggled by IO). Reduced motion: static, wrapped list.
- **Accordion** (Services, FAQ): CSS grid-rows trick; only one open at a time on mobile; deep links `#ai` open the matching item and scroll to it.
- **Language switch:** a link to the same page in the other language (`/services/` ↔ `/en/services/`), with the current scroll position preserved via `sessionStorage` (`scrollY` ratio restored on load). Layout must not shift — reserve widths so both languages fit.
- **Form micro-interactions:** floating labels, chips toggle with a 120 ms scale 0.96 press, submit button shows a progress state.
