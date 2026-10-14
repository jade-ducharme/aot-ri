# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is the public website for Astronomy on Tap Rhode Island (a public astronomy talks/trivia meetup series). It is a single-page Create React App (CRA) site — one long scrolling home page assembled from section components, with no client-side router.

## Commands

- `npm start` — run the dev server at http://localhost:3000 (CRA hot reload)
- `npm test` — run the Jest/RTL test suite in watch mode (`react-scripts test`); pass `-- --watchAll=false` for a single non-watching run, or `-- -t "test name"` to filter
- `npm run build` — production build to `build/`
- There is no separate lint script; ESLint runs via CRA's build/dev pipeline using the `react-app` / `react-app/jest` configs declared in `package.json`

Deployment is via Firebase App Hosting (`apphosting.yaml`) with Firebase Storage rules (`storage.rules`, currently deny-all) and an empty Remote Config template (`remoteconfig.template.json`). Vercel Analytics (`@vercel/analytics`) is also wired into `App.js`.

## Architecture

**Content is data-driven via static JSON, not hardcoded in components.** The three content-heavy sections fetch JSON from `public/data/` at runtime with `fetch("/data/*.json")` in a `useEffect` (not imported as modules):

- `public/data/events.json` — past/upcoming talk events. Each event has a `date` (ISO with timezone) and one or two speakers, with per-speaker fields keyed by suffix (`speaker1`/`speaker2`, `title1`/`title2`, `ins1`/`ins2`, `dep1`/`dep2`, `photo1`/`photo2`) rather than an array — `Events.js` normalizes these into a `speakers` array at render time.
- `public/data/gallery.json` — gallery photos, each with `img`, `caption`, and a `display` boolean flag used to filter what's shown; sorted reverse-alphabetically by filename.
- `public/data/trivia.json` — past trivia quizzes, keyed by event with a `date`/`title` plus a nested Q&A object that's unpacked into an array in `Trivia.js`.

Images referenced by these JSON files live under `public/images/{speakers,gallery,organizers,promo}/`. Organizer bios in `About.js`, by contrast, are hardcoded inline (not JSON-driven).

**Component composition** (`src/App.js` → `Home.js`): `Welcome` (splash) then `Home`, which renders `NavBar`, `Events`, `Trivia`, `Gallery`, `Faq`, `About`, `Footer` in sequence. `NavBar` links are same-page anchors (`#events`, `#trivia`, `#gallery`, `#faq`, `#about`, `#contact`) into these sections — there's no routing, so new sections must be added to this single scroll flow and given a matching nav anchor.

**Responsive layout pattern**: components generally render both a mobile and a desktop layout in the same return (Bootstrap `d-flex d-lg-none` / `d-none d-lg-flex` pairs), rather than using CSS alone to reflow one markup tree — see `Events.js`'s `MobileLayout`/`DesktopLayout` and `Home.js`'s `Intro`. Several components also track `window.innerWidth >= 992` in state (`showControls`) to switch between a desktop Swiper (with navigation arrows) and a touch-friendly mobile Swiper (swipe-only, no arrows) for the same carousel.

**Styling**: one CSS file per component under `src/assets/`, imported directly into that component (e.g. `Events.js` imports `../assets/Events.css`), plus Bootstrap 5 for grid/utility classes. There is no CSS-in-JS or Tailwind.

**Carousels**: Swiper (`swiper/react`) is the carousel library used for past events, trivia quizzes, and the gallery — not `react-slideshow-image` (present as a dependency but not currently used in `src/components`).

**Date handling**: `date-fns` for plain formatting, `date-fns-tz`'s `formatInTimeZone` specifically for the upcoming event's date (hardcoded `America/New_York`), since event timestamps in the JSON already carry an explicit UTC offset.
