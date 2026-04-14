# Portfolio-3d — Claude context

Single-page 3D portfolio for Brian (BrianPaulDev). Currently at "v1 completed" — **working, do not modify anything unless explicitly asked.** Before editing any file, re-read this file and the relevant section below.

## Golden rules

1. **Don't touch working code.** v1 is locked. Only edit when the user explicitly requests a change or a new section. No drive-by refactors, no "while I'm here" cleanups.
2. **Ask before refactoring.** Especially `Canvas.astro` — it's load-bearing and fragile.
3. **No footer, no contact section, no i18n.** Intentional omissions. Do not suggest adding them.
4. **English only** across all copy.
5. **Follow the existing patterns.** Hardcoded content in `index.astro`, Tailwind utilities, GSAP ScrollTrigger for reveals, section anchors with `#id`.
6. **No React.** Project is pure Astro + vanilla Three.js. Don't introduce a framework.
7. **pnpm only** (v10.33.0). Don't run npm/yarn.
8. **Always invoke the `threejs-fundamentals` skill** before doing any Three.js or 3D model work (editing `Canvas.astro`, loading `.glb` files, adding 3D scenes, debugging rendering, etc.). It's installed globally from `cloudai-x/threejs-skills`.

## Stack

- **Astro 5.14.6** — static single-page site, no SSR yet
- **Three.js 0.180.0** — 3D rendering, GLTFLoader, post-processing (UnrealBloomPass, BokehPass, EffectComposer)
- **GSAP 3.13.0** + ScrollTrigger — scroll-driven animations
- **Swiper 12.0.3** — projects carousel
- **Tailwind CSS v4.1.14** via `@tailwindcss/vite`
- **TypeScript** strict (`astro/tsconfigs/strict`)
- **pnpm 10.33.0** (pnpm-workspace.yaml present)
- **Deploy target:** Vercel (adapter not yet installed — add `@astrojs/vercel` only when user asks to deploy)

## File map

```
src/
├── pages/index.astro              # main page, all sections hardcoded here
├── layouts/Layout.astro           # wrapper: #0a0a0a bg, noise overlay, canvas z-index
├── components/
│   ├── ProjectsCarousel.astro     # Swiper carousel, projects array inline (→ projects.json later)
│   └── Canvas/
│       ├── Canvas.astro           # ~1175 lines, the 3D engine — FRAGILE
│       └── Canvas-commented.astro # annotated reference version
└── styles/global.css              # @font-face, clamp() typography

public/
├── dev.glb                        # source geometry for particle system
├── dev-test.glb                   # test variant
├── earth-globe.glb                # RESERVED for future globe section — keep it
├── triangles/py-lod1..7.glb       # triangle LODs (engine uses py-lod7.glb)
├── triangles/pos-33.exr           # position reference
├── fonts/                         # CabinetGrotesk, Switzer (.woff2)
└── project*.webp                  # carousel placeholder images

astro.config.mjs                   # devToolbar disabled, Tailwind vite plugin
```

## Sections in `src/pages/index.astro`

All content hardcoded. One long scroll, no separate pages, no blog.

| ID | Name | Content |
|---|---|---|
| `#main` | Hero | Name, "Front end developer", "Schedule a call" CTA |
| `#introduction` | Intro | "Transforming ideas into pixel-perfect reality" tagline + 5yr intro |
| `#about` | About | Mercado Libre, Globant, US startups, 70% perf claim |
| `#about-detailed` | Extended about | Dashboards, 3D experiences, React/TS, business impact |
| `#projects` | Projects | Rendered by `ProjectsCarousel.astro` — 8 placeholders + "Coming Soon" modal |

**Planned future section:** Globe (using `public/earth-globe.glb`).

When adding a new section: give it an `#id` anchor, use Tailwind utilities, and wire up a GSAP ScrollTrigger reveal matching the cadence of the existing ones.

## The 3D engine — `src/components/Canvas/Canvas.astro`

**Treat as fragile. Never edit without explicit user request — ask first.** If a new section needs 3D interaction, prefer extending the existing scene rather than creating a second Three.js context.

**Dual-canvas architecture**
- **Background canvas:** Three.js scene with `UnrealBloomPass` + `BokehPass` (depth-of-field blur) composed via `EffectComposer`.
- **Foreground canvas:** transparent overlay, sharp particles, no post-processing.
- Both fixed-positioned in `Layout.astro` at z-index -1 / -2.

**Floating triangles (900 instances)**
- Template: `public/triangles/py-lod7.glb`
- 6 colors: turquoise, gold, orange, off-white, purple, pink
- 5 animation modes: `CIRCULAR`, `VERTICAL_WAVE`, `DIAGONAL`, `SPIRAL`, `DRIFT`
- Spherical distribution around the camera, radius 10–35
- **Inverted parallax:** triangles offset *opposite* to mouse direction

**Particle system from `dev.glb`**
- ~30 particles generated per face of the model's geometry
- Raycaster hover detection, ~100px radius
- Scale + color animate on cursor proximity

**Mouse / scroll behavior**
- Camera look offset capped at 0.01 radians (intentionally subtle)
- Smooth easing between states (no snapping)
- GSAP ScrollTrigger: hero entrance (Y + opacity + rotationX), section cascades, parallax on gradient overlays, staggered reveals on about paragraphs

## Data & content

- **Projects data:** currently an inline array inside `ProjectsCarousel.astro`. Shape: `{ name, category, image, github, demo }`. **Planned:** move to `src/data/projects.json` when real projects are added — do this as part of that task, not preemptively.
- **Everything else** (hero copy, about paragraphs) is hardcoded in `index.astro`. No CMS, no content collections (yet).

## Styling

- **Tailwind v4** utility-first, primary approach.
- **`src/styles/global.css`:** `@font-face` for CabinetGrotesk (bold/medium) and Switzer (regular/semibold); responsive typography via `clamp()` (h1: 3rem–10rem, h3: 2rem–6rem, h4: 1.5rem–4rem); global reset on `*`.
- **Layout.astro:** dark `bg-[#0a0a0a]`, animated SVG fractal-noise overlay, fixed canvas positioning.
- Component-scoped `<style>` blocks in `Canvas.astro` and `ProjectsCarousel.astro`.

## Astro conventions (how this project uses Astro)

- **`.astro` file anatomy:** frontmatter script at the top (runs at **build time**, can import, fetch, read files), then HTML-like template, then optional `<style>` and `<script>` blocks. Don't confuse frontmatter code with client code — anything that needs to run in the browser goes in a `<script>` tag, not frontmatter.
- **Client interactivity = `<script>` tags.** Astro bundles them with Vite, hoists them, and runs them client-side. This is how `Canvas.astro` boots Three.js without a framework island. For new interactive features, follow the same pattern: put the logic in a `<script>` block inside the component. **Do not propose React/Vue/Svelte islands** — no UI framework integration is installed and the user doesn't want one.
- **Scoped styles:** `<style>` blocks inside `.astro` files are automatically scoped to that component. Use them for component-local styling that Tailwind utilities can't express cleanly. Global styles go in `src/styles/global.css`.
- **Routing:** file-based from `src/pages/`. Currently only `index.astro` — intentional (single-page scroll). Don't add new pages unless the user asks; new sections go inside `index.astro` as anchored blocks.
- **`public/` vs `src/`:**
  - `public/` — served as-is at the site root. Reference with absolute paths like `/dev.glb`, `/fonts/Switzer-Regular.woff2`. All 3D models, fonts, and carousel images live here because they need stable URLs for Three.js loaders and `@font-face`.
  - `src/` — processed by Vite. Import assets from here when you want hashing/bundling.
- **Tailwind v4 wiring:** uses `@tailwindcss/vite` plugin in `astro.config.mjs` (not the legacy PostCSS setup). Config is CSS-first — don't create a `tailwind.config.js`, v4 doesn't need one unless there's something custom to declare.
- **TypeScript in `.astro`:** frontmatter is TS by default (strict mode via `astro/tsconfigs/strict`). `<script>` tags are also type-checked. Types flow through — use them.
- **No SSR yet:** project is fully static. If a future section needs server-side logic, that's when we add the Vercel adapter (`@astrojs/vercel`) and switch `output` mode. Until then, keep it static.
- **Dev toolbar is disabled** in `astro.config.mjs` — intentional, don't re-enable.

## Commands

```bash
pnpm dev       # start dev server
pnpm build     # production build
pnpm preview   # preview production build
```

## Before you edit anything

1. Re-read this file.
2. Re-read the memory files under `~/.claude/projects/-home-paulsalinas-dev-personal-Portfolio-3d/memory/` that match what you're about to touch.
3. Confirm the user actually asked for the change. If in doubt, ask.
