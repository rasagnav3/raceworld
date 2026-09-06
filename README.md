# RACEWORLD — Phase 1 Build

A single-file, fully working preview of the RACEWORLD homepage: cinematic hero,
scroll-driven racing car, Liquid-Glass-style UI, shader-style hero background,
and every Phase 1 section from the brief (Live On Track, Next On The Grid,
Global Racing Map, Categories, Calendar, Racing Media, Drivers, Teams,
Circuits, News, My Racing, Finish Line).

## 1. What starts the application
Open **`raceworld.html`** directly in a browser. There's no build step —
it's plain HTML/CSS/JS plus two CDN scripts (GSAP + ScrollTrigger), so it
runs immediately, including on a phone, with no `npm install`.

## 2. Why one file, not a Next.js project yet
My sandbox has no network access, so I can't `npm install` React, GSAP,
`liquid-glass` from GitHub, or `@shadergradient/react`, and I can't run a dev
server to catch hydration/build errors — I can only write files, not execute
a live network build. Rather than hand you an unverified Next.js scaffold I
couldn't actually run, I built the real experience as one static file you can
open right now and judge for real. Section 6 below covers converting this into
the modular Next.js project the brief calls for once you're in an environment
with npm access.

## 3. Where things live in this file
The file is organized into the same conceptual modules the brief asks for,
marked with `/* MODULE: ... */` or `<!-- SECTION -->` comments:

- **Loader** — `#rw-loader`, hidden after `window.load`
- **Nav** — `#rw-nav`, gains a Liquid-Glass surface past 40px of scroll
- **ScrollRace track** — `#rw-track-layer`: a fixed SVG path along the right
  edge; a car icon travels along it as you scroll (see §4)
- **ShaderBackground** — `#rw-hero-canvas`: a small hand-written WebGL
  fragment shader producing the "liquid metal" hero look (see §6)
- **LiquidGlass** — the `.rw-glass` CSS class (`backdrop-filter: blur() saturate()`
  + a refraction highlight layer): applied to the nav, live cards, the map
  panel, and the My Racing panel
- **Sections** — Live On Track, Next On The Grid, Global Racing Map,
  Categories, Calendar, Racing Media, Drivers, Teams, Circuits, News,
  My Racing, Finish Line, Footer — each its own `<section>`
- **Demo data + renderers** — `EVENTS`, `DRIVERS`, `TEAMS`, `CIRCUITS`, `NEWS`,
  `CATEGORIES` arrays near the top of the `<script>` block, each with a
  `render...()` function. This mirrors the `data/` + component split the
  brief asks for — in the Next.js version each array becomes its own file
  under `data/`, and each `render...()` becomes its own component.

## 4. How the scroll-driven racing car works
`ScrollRace` (in the script) computes `window.scrollY / (document height − viewport
height)` as a 0–1 progress value on every scroll event, then:
1. Uses `path.getPointAtLength(progress × totalLength)` on the fixed SVG path
   to get the car's (x, y) position at that point in the journey.
2. Samples a point 1px further along the path to compute a tangent angle, and
   rotates the car icon to match — so it visibly "steers" through the curves.
3. Animates the path's `stroke-dashoffset` so the electric-blue "traveled"
   portion of the track grows as you scroll.

It's hidden on narrow screens (<900px), per the brief's mobile-simplification
requirement, and short-circuits to a static frame under
`prefers-reduced-motion`.

GSAP ScrollTrigger is layered on top for two extra scroll-scrubbed touches
(hero car drift + speed-line reveal) and a one-time reveal animation on the
Finish Line — both skipped under reduced motion.

## 5. Liquid Glass — what's real vs. simulated here
The brief asks for the real `liquid-glass.js` implementation from
github.com/shuding/liquid-glass. I couldn't fetch that repo in this sandbox
(no network), so `.rw-glass` here is a CSS-only approximation: backdrop blur +
saturation + a diagonal highlight overlay. It looks and reads as glass, but it
doesn't do true refractive distortion of what's behind it.

**To use the real implementation**, once you have npm access:
```bash
npm install liquid-glass-react   # or follow shuding/liquid-glass's own install steps
```
Then wrap `.rw-glass` elements in the real component instead, inside
`components/LiquidGlass.tsx`, so the rest of the app doesn't need to change.

## 6. ShaderGradient — what's real vs. simulated here
Same constraint: `@shadergradient/react` needs npm + React Three Fiber, which
I can't install offline. `#rw-hero-canvas` instead runs a small hand-written
WebGL fragment shader (layered sine waves blended between pearl/silver/blue/red)
that produces a comparable "liquid metal" look with zero dependencies.

**To use the real implementation**, once you have npm access:
```bash
npm install @shadergradient/react @react-three/fiber three
```
Then swap the shader block in `components/ShaderBackground.tsx` for
ShaderGradient's `<ShaderGradientCanvas>` per the package's docs, keeping the
same pearl/silver/electric-blue palette.

## 7. Where demo data comes from / where real APIs plug in
Every number and result on this page is clearly demo data — see the
`DEMO DATA` tags on event, live-session, and calendar cards. Live sessions
that would come from a real timing feed show `LIVE DATA UNAVAILABLE` rather
than inventing scores; the third video slot shows `NO VIDEO AVAILABLE` rather
than a broken player. When you're ready for Phase 3–6 of the brief, each demo
array (`EVENTS`, `DRIVERS`, etc.) becomes a call into a `services/*Service.ts`
file that fetches from a real API and returns the same shape.

## 8. Converting this into the full modular Next.js project
This file intentionally mirrors the brief's target file structure so the
split is mechanical:

```
components/Navbar.tsx        <- #rw-nav + its script listener
components/Hero.tsx          <- #rw-hero markup
components/ShaderBackground.tsx <- the WebGL module (or real ShaderGradient)
components/ScrollRace.tsx    <- the ScrollRace module
components/RacingTrack.tsx   <- the SVG path/track markup
components/LiquidGlass.tsx   <- the .rw-glass treatment (or real liquid-glass)
components/LiveEvents.tsx    <- #rw-live section + card
components/UpcomingEvents.tsx / EventCard.tsx <- #rw-events + card
components/RacingMap.tsx     <- #rw-map section
components/CategoryCard.tsx  <- #rw-categories cards
components/VideoCard.tsx     <- #rw-media cards
components/DriverCard.tsx / TeamCard.tsx / CircuitCard.tsx <- people grids
components/Calendar.tsx      <- #rw-calendar
components/NewsCard.tsx      <- #rw-news
components/Footer.tsx        <- #rw-footer
data/*.ts                    <- EVENTS/DRIVERS/TEAMS/CIRCUITS/NEWS/CATEGORIES arrays
services/*Service.ts         <- new files, wrap future API calls
```
Once split up, wrap any WebGL/ScrollTrigger component with `"use client"` and
initialize GSAP/canvas work inside `useEffect` to avoid Next.js SSR/hydration
errors, per the brief.

## 9. Running locally
No install needed for this file — just open `raceworld.html` in any modern
browser (Chrome, Safari, Edge, Firefox). An internet connection is needed
only for the Google Fonts and GSAP CDN links; without it, the page still
works and falls back to system fonts with the CSS-driven interactions intact.

## 10. Exporting / connecting to GitHub
This is already a single portable file — commit it as-is, or use it as the
first commit in a new repo (`git init`, `git add raceworld.html README.md`,
`git commit -m "Phase 1: RACEWORLD homepage"`) before splitting it into the
Next.js structure in §8.

## 11. Deployment
As a static file, it can be deployed instantly to Netlify, Vercel, GitHub
Pages, or Cloudflare Pages with zero configuration — drag-and-drop the file
(renamed to `index.html`) onto any static host.

## 12. What's next (Phases 2–8 from the brief)
- **Phase 2**: turn the nav links and event/driver/team/circuit cards into
  real routed pages (`/events/[slug]`, `/drivers/[slug]`, etc.)
- **Phase 3**: move demo arrays into a typed `data/` layer with shared
  interfaces (`Event`, `Driver`, `Team`, `Circuit`, `Result`)
- **Phase 4**: add `services/*Service.ts` calling real racing/live-timing APIs
- **Phase 5**: wire the video grid to a real video API/official embeds
- **Phase 6**: replace the `LIVE DATA UNAVAILABLE` placeholder with a live
  timing feed
- **Phase 7**: add auth (sign up/login/profile) and persist My Racing
  favorites server-side
- **Phase 8**: performance pass (image optimization, code splitting) and
  deploy
