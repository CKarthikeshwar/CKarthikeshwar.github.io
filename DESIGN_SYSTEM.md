# Design system & architecture

This site runs **two coexisting visual systems**, split cleanly by route. There is no shared
design system across the whole site — that's deliberate (see "Why two systems" below).

| | Homepage (`/`) | Everything else (`/dump`, `/dump/*`, `/projects/*`) |
|---|---|---|
| Layout | `src/layouts/HomeLayout.astro` | `src/layouts/BaseLayout.astro` |
| Chrome | Top bar (prompt + nav pills + socials) | Fixed left sidebar (`src/components/Sidebar.astro`) |
| Palette | "Matrix Mint" (dark green/mint) | Violet (`src/styles/global.css` `@theme`) |
| Body font | IBM Plex Sans | Inter |
| Mono font | JetBrains Mono (both) | JetBrains Mono (both) |
| Identity | "Glass Terminal" — CS/terminal flavored | Plain minimal portfolio |

## Why two systems

The homepage went through a full design exploration (`design/README.md`,
`design/homepage/*.dc.html`) that settled on "Direction E — Glass Terminal". That redesign was
scoped to the homepage only (see `SPEC.md` V2) — `/dump` and `/projects/coinflow` were never
part of it and keep the original sidebar-based look. `/projects/coinflow` already had its own
bespoke visual treatment (Geist/Manrope fonts, custom styling) before this redesign, so a
per-route visual identity isn't a new pattern here.

**If asked to extend the terminal look to the rest of the site**, that's a deliberate scope
expansion, not a bug fix — confirm with the user before touching `BaseLayout.astro` or
`Sidebar.astro`, since every other route depends on them.

---

## Homepage design tokens ("Glass Terminal")

Defined as CSS custom properties on `.terminal` in `src/pages/index.astro`, scoped so they can
never leak into the sidebar-based pages:

```css
.terminal {
  --t-bg: #090d0b;
  --t-text: #e8f5ee;        /* headings, primary text */
  --t-text2: #9fb8ac;       /* body copy */
  --t-muted: #62796e;       /* labels, numerals, prompts */
  --t-accent: #6ee7b7;      /* mint accent */
  --t-accent-soft: #a7f3d0; /* unused today — reserved for a lighter accent step */
  --t-border: rgba(110, 231, 183, 0.16);
}
```

Fonts: **JetBrains Mono** (terminal chrome — prompts, nav pills, numerals, hero name) +
**IBM Plex Sans** (body copy — bio line, descriptions). Both loaded via Google Fonts in
`HomeLayout.astro`. Inter was deliberately dropped for the homepage — it's flagged as an
overused/generic face by design review (`impeccable`'s detector); IBM Plex Sans was chosen
because it's thematically aligned (designed for technical products) and not on that overused
list. This does **not** affect `/dump` or `/projects/*`, which still use Inter via the
site-wide `--font-sans` token in `global.css`.

Spacing/radius/shadow are not tokenized — they're inline Tailwind arbitrary values or literal
px in `terminal.css`, since the homepage is one page with a small, fixed set of components. If
this system grows (more pages adopt it), promote the repeated values to tokens then.

---

## Homepage file map

```
src/layouts/HomeLayout.astro        <html> shell: fonts, meta, imports global.css + terminal.css
src/styles/terminal.css             all homepage-only CSS (see below)
src/pages/index.astro               assembles the page + the three <script> behaviors
src/components/home/
  HomeNav.astro                     prompt + nav pills + social icons
  HomeHero.astro                    terminal boot-sequence + photo
  HomeExperience.astro              collapsible "$ ls experience/"
  HomeProjects.astro                collapsible "$ ls projects/ --sort=date"
  HomeFooter.astro                  "$ open ./dump" → /dump
```

Content (bio line, experience entry, project entries) lives inline in each component as a
small array — there's no shared data file. With one experience entry and three projects, a
central data module would be premature; revisit if either list grows past ~5–6 entries or a
second page needs the same data.

### Why `terminal.css` is a real stylesheet, not scoped `<style>` blocks

Astro's per-component `<style>` blocks are scoped to that file only (via a generated
`data-astro-cid` attribute) — a class defined in `index.astro` would **not** apply to markup
rendered by `HomeNav.astro` etc. Since colors, animations, `.t-glass`, `.t-toggle` and so on
need to be shared across every home component, they live in one plain global stylesheet
(`terminal.css`, imported once in `HomeLayout.astro`) instead. Component-specific one-off
values (padding, font-size at a given breakpoint) stay as inline styles/Tailwind classes in
each component.

---

## Key patterns

### 1. Boot-sequence hero (`HomeHero.astro` + `terminal.css` keyframes)

Pure CSS, no JS — a chain of `animation-delay`s:

```
$ whoami --full   (0s → 1s, width-reveal typing effect)
name reveals      (1.1s → 2.8s, clip-path wipe)
$ cat role.txt    (2.8s → 3.7s, width-reveal typing effect)
role line fades   (3.7s → 4.3s)
```

If you change any duration, the next step's delay must shift to match — they're not
independently timed. (This bit us once: an earlier version changed the name-reveal duration
without retiming `.t-type2`/`.t-role-line`, leaving a visible dead gap.)

**Hero name sizing uses CSS container queries, not viewport units.** `font-size: clamp(1.75rem,
10.5cqw, 4.5rem)` on the `<h1>`, with `container-type: inline-size` on its `.t-glass` parent.
This was a real bug fix, not a stylistic choice: the hero panel's actual width doesn't scale
linearly with viewport width — it jumps at the `lg` breakpoint when the layout switches from
stacked to a two-column grid (photo takes a fixed 380px + gap). A `vw`-based clamp doesn't know
about that jump, so at some widths the computed font-size was too large for the narrower
post-breakpoint column, and `overflow: hidden` clipped the last letter while the cursor wrapped
to its own line. `cqw` scales off the actual rendered container width instead, so it can't
drift out of sync with the layout, however the layout changes in the future.

### 2. Collapsible sections (`.t-toggle` / `.t-toggle-panel`, wired in `index.astro`)

Server-rendered fully expanded (`max-height: none`) so it works with no JS. On load, a script
measures each panel's `scrollHeight` and switches it to an explicit px `max-height` so the CSS
transition has a real value to animate to/from — `max-height` can't transition to/from `none`
or `auto`. Toggling flips `aria-expanded` on the `<button>` and animates the panel between `0px`
and its measured height; a resize listener re-measures open panels since text reflow changes
their height.

**Panels need internal padding, not just `overflow: hidden`.** The clip boundary that makes the
collapse animation work also clips anything that visually escapes the panel — which the card
hover effect (`translateY(-4px)` + shadow) does. Each panel's content sits inside an extra
`pt-2 pb-6 px-1` wrapper purely to give the hover lift and shadow headroom before hitting that
boundary. Don't remove this padding without re-checking hover on the first/last card in a panel.

### 3. Numbered items (`.t-item-row` / `.t-item-num`)

Each experience/project entry is a two-column grid: a muted `01`/`02`/`03` numeral in its own
column (`.t-item-num`), the `.t-glass` card in the other. The numeral sits **outside** the card
on purpose — this replaced an earlier version where a date line sat inside the card, then a
numeral inside the card; both were rejected in favor of matching `design/homepage/DirectionD.dc.html`'s
margin-note treatment.

### 4. Card hover (`.t-glass:has(.t-hover-title):hover`)

Lift + soft shadow, deliberately **without** the 3D tilt (`rotateX`/`rotateY`) that
`DirectionD.dc.html`'s original hover used — that tilt was tried and explicitly rejected.
`:has()` is used to scope the effect to cards that contain a `.t-hover-title` (experience/
project cards), leaving the hero/photo panels with only the plain border-brighten hover.

### 5. Particle background (`index.astro` `<script>`)

Canvas-based, vanilla JS, no dependency. Two important, non-obvious choices:

- **Sized off the wrapper's `getBoundingClientRect()`, not `window.innerWidth/innerHeight`.**
  Using the viewport caused the canvas's internal coordinate space to mismatch its actual
  rendered size inside the page, squashing every particle into one corner and leaving the rest
  of the page blank. Always measure the actual DOM element the canvas is meant to fill.
- **`position: absolute` inside `.terminal`, not `position: fixed`.** An earlier version used
  `fixed`, which pins the canvas to the *viewport* — so the same frozen pattern just followed
  you as you scrolled a page taller than one screen, instead of covering the whole page.
  `absolute` (with `.terminal` as `position: relative`) makes it scroll with the content and
  span the full page height.
- Particle count scales with rendered page area (`Math.max(60, Math.min(160, area / 9000))`)
  so density feels consistent from a phone to a tall desktop page, and the distribution mixes
  a uniform base layer with a smaller `Math.random() ** 2.6`-biased layer concentrated toward
  the top-left, so there's a deliberate hotspot without leaving other corners empty.
- Respects `prefers-reduced-motion` — the whole thing is skipped for users who've asked for it.

### 6. Nav (scroll-spy + always-scrollable pills)

`.t-nav-pills` is `overflow-x: auto` unconditionally, at every screen size — rather than
maintaining a separate "mobile nav" markup variant, the same row just never needs to scroll
when it already fits. An `IntersectionObserver` in `index.astro` watches `#about`,
`#experience`, `#projects`, `#dump` and toggles `aria-current="true"` on the matching pill
(`[data-navpill]`) as the user scrolls.

---

## Responsive strategy

Not a fixed desktop/mobile pair — the layout is meant to hold up at any width in between:

- **Fluid, not stepped, where it matters most.** Hero name uses `clamp()` (container-query
  based, see above) instead of jumping between fixed sizes at a breakpoint.
- **One breakpoint for the hero's structural layout** (`lg` / 1024px): stacked → two-column
  grid via `order-1`/`order-2` (same markup, CSS-only reorder), not duplicated markup.
- **Nav pills never need a distinct mobile variant** — see above.
- **Particle density scales continuously with page area**, not a mobile/desktop toggle.
- Tailwind's default breakpoints (`sm`/`lg`) are used for padding/spacing/type-size steps
  throughout; there's no custom breakpoint config.

---

## Routes

| Route | Layout | Notes |
|---|---|---|
| `/` | `HomeLayout` | Glass Terminal homepage |
| `/dump` | `BaseLayout` | Blog index (renamed from `/writing` — old route 404s) |
| `/dump/[slug]` | `BaseLayout` | Individual post, from the `blog` content collection (`src/content/blog/`) |
| `/projects/coinflow` | `BaseLayout` (`bleed`) | Bespoke case-study page, its own fonts (Geist/Manrope) and styling — predates this redesign |

---

## Extending this

- **New homepage section** ("$ ls something/"): copy the `HomeExperience.astro` pattern — a
  `<button class="mono t-toggle" aria-expanded="true" aria-controls="X-panel">`, a
  `<div id="X-panel" class="t-toggle-panel" style="max-height: none;">` with a padded inner
  wrapper, and add its id to the `sections` array in `index.astro`'s scroll-spy script if it
  should get a nav pill.
- **New experience/project entry**: add an object to the array in the relevant component — the
  `.map()` already handles numbering (`i + 1`) and layout.
- **New homepage color**: add it to `.terminal` in `terminal.css`, not `global.css` — keep it
  scoped.
- **Changing an animation timing**: re-check the whole boot-sequence chain in
  `HomeHero.astro`/`terminal.css` (see pattern 1) — nothing in that chain is independently
  timed.
