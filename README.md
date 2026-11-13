# RT Schilderwerken — website

One-page site for René van der Tuijn, schilder in Katwijk aan Zee.
Nuxt 4 · Tailwind v4 · GSAP (ScrollTrigger, Flip, SplitText, DrawSVG) · Lenis.
Design source: `docs/brand-guidelines.md` (v2.0) and `docs/superpowers/specs/2026-11-10-apple-style-restyle-design.md`.
Palette and logo artwork from the client: `../colors/palette.scss`, `../colors/RT-schilderwerken.svg`.

```bash
pnpm install
pnpm dev        # http://localhost:3000
pnpm build      # prerendered page + /api/offerte server route
```

Use pnpm. (npm currently fails on this machine because of a root-owned `~/.npm` cache.)

## Before launch — placeholders

| What | Where |
| --- | --- |
| Phone, e-mail, address, KvK, BTW, Instagram handle, project count | `app/content/site.ts` |
| Biography details (where René trained, start year) | `app/components/sections/SiteOver.vue` |
| Portrait photo of René (real photo, 4:5, ≥1200×1500) | `SiteOver.vue` placeholder block |
| Testimonials (marked as placeholder on the page) | `app/content/testimonials.ts` |
| Privacy statement / voorwaarden `[BRACKETS]` | `app/content/legal.ts` |
| Werk tiles (Higgsfield placeholders) → René's own detail photos | `app/content/werk.ts` + `public/img/` |
| Mail provider key (Resend-compatible) | env `NUXT_MAIL_API_KEY`, `NUXT_MAIL_FROM`, `NUXT_MAIL_TO` |
| E-mail templates (notification to René + confirmation to customer) | `server/utils/offerte-emails.ts` — logo PNG served from `public/email/logo.png` (regenerate with `node scripts/make-email-logo.mjs`) |
| Photo upload presigner (S3/R2) | `server/api/upload-url.post.ts` — answers 501 until wired; form still submits |

## Assets (Higgsfield)

Raw generations live in `assets-raw/` (git-ignored). Stills: GPT Image 2.5; films: MiniMax H3 (2K, start + end keyframe).
The hero film and its keyframes are shot in the current palette. Everything else comes from the first (blue-graded) set and gets the neutral grade from `scripts/grade.mjs`; the first hero set is kept in `assets-raw/v1/`.

```bash
node scripts/grade.mjs                               # self-check of the grade
node scripts/process-images.mjs --only=werk          # assets-raw/*.png → public/img/*.jpg, graded (also: dienst, tape)
node scripts/process-images.mjs --only=hero --grade=none
# film → scroll-scrub frames: <video> <outDir> <count> <w> <h> [tail frames] [webp quality] [--tail-color=#hex] [--grade=neutral|none]
node scripts/extract-frames.mjs assets-raw/hero-16x9.mp4 public/frames/hero/desktop 150 1920 1080 18 62 --tail-color=#f2f2f3 --grade=none
node scripts/extract-frames.mjs assets-raw/hero-9x16.mp4 public/frames/hero/mobile 90 990 1760 12 42 --tail-color=#f2f2f3 --grade=none
node scripts/extract-frames.mjs assets-raw/tape-16x9.mp4 public/frames/tape/desktop 60 1440 810 0 38
node scripts/extract-frames.mjs assets-raw/tape-9x16.mp4 public/frames/tape/mobile 60 810 1440 0 40
node scripts/build-logo.mjs         # client SVG → app/components/ui/logo-paths.ts
node scripts/make-icons.mjs         # favicon, apple-touch-icon, og-image from the logo paths
```

The hero's tail frames blend to the page colour (`#f2f2f3`), so the film ends exactly on the page background. If the mobile frame size changes, update `frames.mobile` in `SiteHero.vue`.

Frame budgets: hero desktop 6.2 MB / mobile 3.2 MB; tape desktop 5.8 MB / mobile 2.3 MB (tape loads lazily).
