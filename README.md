# Airbnb listing clone (take-home)

An original implementation of an Airbnb-style **listing page**, **photo tour**
and **lightbox**, built with Next.js (App Router), React, TypeScript and CSS
Modules. Desktop only (designed at a 1440px-wide viewport). There is no backend:
all content comes from typed data in `src/data`.

> **Please read the status notes before judging fidelity.** The live reference
> (`https://airbnb-clone-umber-two.vercel.app`) could not be inspected while this
> was built: only the three 1440x860 screenshots embedded in the assignment were
> available. Everything visible in them was measured and matched; everything else
> is clearly marked as an assumption or placeholder. The photos are generated
> placeholders. See [Known limitations](#known-limitations--assumptions) and
> [`docs/QA.md`](docs/QA.md).

## Status at a glance

| Check | Status |
| --- | --- |
| `npm install`, `npm run lint`, `npm run build`, `npm run dev` | **Not run by the author of this documentation** (no network in the build sandbox). Please run them locally, see [Commands](#install-and-run). |
| Type-check of all source against stand-in React types | Passed (see `docs/QA.md`; not a substitute for `npm run typecheck`) |
| 58 headless-Chromium checks (interaction, focus, keyboard, scroll lock, reduced motion, hydration, measured geometry) | Passed, run against an esbuild bundle of the same source, not against `next build` |
| Pixel-perfect parity with the live reference | **Not claimed.** Not verifiable; see QA notes |

## Assignment requirements

| Requirement | Status |
| --- | --- |
| Listing page: layout, spacing, typography, colours | Implemented for everything visible in the reference screenshot: header + search pill, title + Share/Save, 5-image hero grid + "Show all photos", summary, promo card, Guest-favourite card, sticky booking card. Sections below the visible area are **placeholders**. |
| Photo tour opened from "Show all photos" or any hero image | Implemented (full-screen white view, 9 categories, thumbnail navigation, per-category sections and photo mosaic) |
| Lightbox opened from any gallery photo, prev/next arrows, keyboard ←/→ | Implemented (counter "N of 43", disabled Previous/Next at the ends, Escape, mouse and keyboard) |
| Interaction and accessibility: keyboard, focus management | Implemented: `role="dialog"`, `aria-modal`, focus trap, focus return, `inert` background, scroll lock, live announcements, `prefers-reduced-motion` |
| Animations, hover and scroll motion matching the reference | **Approximated only.** Motion is unverified: subtle fades using central tokens (`UNVERIFIED` comments) |
| Desktop only | Yes |
| Architecture diagram (image/PDF) | [`docs/architecture.png`](docs/architecture.png) and [`docs/architecture.pdf`](docs/architecture.pdf) |
| AI workflow: agents, skills, prompts | [`.claude/`](.claude/README.md), [`CLAUDE.md`](CLAUDE.md), [`PROMPTS.md`](PROMPTS.md) |
| Original code (no lift-and-shift from the reference) | Yes: written from screenshots and measurements only; no reference source or third-party clone code was used |
| Deployment | Not deployed (optional in the assignment) |

## Tech stack

- Next.js 15 (App Router), React 19, TypeScript (strict)
- Plain CSS Modules with design tokens (`src/styles/tokens.css`), no UI library
- Runtime dependencies: only `next`, `react`, `react-dom`
- Dev tooling: ESLint (`eslint-config-next`), TypeScript
- Node.js 18.18 or newer

## Project structure

```text
.
├── CLAUDE.md                     # project rules for Claude Code
├── PROMPTS.md                    # the actual prompts used, in order
├── README.md
├── .claude/
│   ├── README.md                 # what each agent/skill is for
│   ├── agents/                   # ui-pixel-reviewer, accessibility-reviewer, code-quality-reviewer
│   └── skills/                   # visual-diff (+measure.cjs, reference values), a11y-check, replace-placeholders
├── docs/
│   ├── architecture.png|pdf|svg  # production-scale architecture diagram
│   ├── QA.md                     # verified vs not-verified
│   └── qa-harness/               # optional headless-browser test scripts (not part of the build)
├── public/                       # empty: no static assets are bundled
└── src/
    ├── app/                      # layout.tsx, page.tsx, globals.css
    ├── components/
    │   ├── header/               # Header, SearchPill
    │   ├── listing/              # TitleBar, HeroGallery, ListingBody, ListingSummary,
    │   │                         # PromoCard, RatingBanner, BookingCard, SectionPlaceholder
    │   ├── gallery/              # GalleryProvider (state), GalleryShell, PhotoTour, Lightbox
    │   └── ui/                   # Container, Icons, Logo, Photo
    ├── data/                     # photos.ts (43 photos, 9 categories, hero ids), listing.ts
    ├── hooks/                    # useFocusTrap, useScrollLock, useReducedMotion
    ├── lib/                      # types.ts, format.ts, placeholder.ts (temporary image generator)
    └── styles/                   # tokens.css
```

Design rules worth knowing:

- **The data layer owns content.** Photos, categories and listing facts live in `src/data`. Components never hard-code photo data, so swapping the placeholders for real photos is a data-only change (see `.claude/skills/replace-placeholders/SKILL.md`).
- **One overlay state system.** `GalleryProvider` holds `listing | tour | lightbox` state plus the elements that opened each layer (for focus return).
- **Honest markers in code.** Comments mark values as `MEASURED`, `DERIVED`, `ASSUMED` or `UNVERIFIED`, and open questions as `TODO(reference)` / `TODO(asset)`. List them with `grep -rnE "ASSUMED|UNVERIFIED|TODO\(" src`.

## Install and run

```bash
npm install          # install dependencies
npm run dev          # http://localhost:3000
npm run lint         # next lint (ESLint, eslint-config-next)
npm run typecheck    # tsc --noEmit
npm run build        # production build (also type-checks and lints)
npm run start        # serve the production build
```

Notes: `next lint` prints a deprecation notice on newer Next 15.x releases but
still works. The version ranges in `package.json` were chosen without network
access, so if `npm install` reports a conflict, adjust them to the current
Next 15 / React 19 releases.

## Gallery interactions

- **Hero grid**: clicking any of the five images, or "Show all photos", opens the photo tour. Clicking a specific image scrolls the tour to that photo's category (assumed behaviour).
- **Photo tour**: white full-screen view with a header (back, title, Share, Save), 9 category thumbnails, and a section per category with heading, detail line and photo mosaic. Clicking a thumbnail scrolls to its section (smooth, instant under reduced motion). The back button or Escape closes it and returns focus to whatever opened it.
- **Lightbox**: clicking any tour photo opens it at that photo. Shows the category name, "N of 43", Previous/Next (disabled and faded at the ends, no wrap-around), a grid button that returns to the photo tour, and a close button. Closing returns to the tour with focus on the photo that was showing.
- While an overlay is open the page underneath cannot be scrolled, focused or clicked, and scroll position is restored on close.

## Keyboard accessibility

| Key | Where | Action |
| --- | --- | --- |
| Tab / Shift+Tab | Photo tour, lightbox | Cycle through controls; focus never leaves the open dialog |
| Enter / Space | Any button | Activate (hero images, thumbnails, photos, controls) |
| ← / → | Lightbox | Previous / next photo (ignored with Alt/Ctrl/Cmd/Shift held) |
| Escape | Lightbox | Close lightbox, back to the photo tour |
| Escape | Photo tour | Close the tour, back to the listing |

Other accessibility work: real `<button>` elements everywhere, labels on all
icon-only buttons, `role="dialog"` + `aria-modal="true"` + accessible names,
`inert` on the background, `aria-disabled` (not `disabled`) on the end-stop
arrows so focus is not lost, a polite live region announcing "Living room 1,
photo 3 of 43", `:focus-visible` outlines, semantic landmarks and headings,
and `prefers-reduced-motion` support. Real screen-reader testing has **not**
been done.

## Responsive / scope note

This implementation targets the requested **desktop** viewport (measured at
1440x860, content column 1120px). Mobile layouts are out of scope per the
assignment and are not implemented; behaviour at very narrow widths is not
tested.

## Known limitations / assumptions

- **Placeholders:** the 43 photos are generated SVG stand-ins (`src/lib/placeholder.ts`). How the 43 split across the 9 categories is a stand-in; detail lines exist only for "Living room 1". The logo, search-pill house, laurel and tag artwork are temporary drawings.
- **Sections below the fold** (host, description, amenities, reviews, location) are labelled placeholders; the booking-card date/guest controls and "Reserve" button are assumed structure.
- **Font:** Airbnb Cereal is proprietary and not bundled, so a system font stack is used. This is the biggest visible difference (text widths, one label wrapping).
- **Motion and hover states** are approximations (marked `UNVERIFIED`).
- **Unknown behaviours implemented by assumption:** hero-image click scrolls the tour to that photo's section; lightbox X returns to the tour (like Escape); tour header assumed sticky; Share, Save, Claim and Terms apply do nothing.
- **Contrast:** the assumed white "Reserve" label on `#ff385c` is ~3.6:1.
- **Not built:** backend, real availability/pricing logic, payments, header menus, date/guest pickers.

## Visual QA limitations

The live reference could not be opened, so visual QA was limited to comparing
against three screenshots, and only for what they show (first viewport of the
listing, the top of the photo tour, the first lightbox photo). Hover states,
animations, scrolling behaviour, all below-the-fold content and the real photos
were never seen. Details, including what was measured and how, are in
[`docs/QA.md`](docs/QA.md) and `.claude/skills/visual-diff/reference-measurements.md`.

## Architecture diagram

![Production-scale architecture](docs/architecture.png)

- Files: [`docs/architecture.png`](docs/architecture.png), [`docs/architecture.pdf`](docs/architecture.pdf), editable source [`docs/architecture.svg`](docs/architecture.svg).
- It shows a production-scale vacation-rental marketplace: web/mobile clients, CDN/edge (WAF), load balancer, Next.js web tier, API gateway, auth, ten services (listing, search, availability/calendar, booking, payment, user/profile, reviews, photo/media, recommendation, notification), an event bus, PostgreSQL, Redis, OpenSearch, feature store, data lake, object storage + image CDN, external services (payment gateway, identity providers, maps, email/SMS/push), and cross-cutting observability, security, CI/CD and multi-region deployment.
- Solid blue arrows are synchronous calls, dashed orange arrows are asynchronous events, green arrows are data/storage paths, dotted purple arrows are external integrations. A side panel spells out the **scaling strategy** for frontend, backend, storage, search and deployment.
- The green dashed outline marks the **only part implemented in this repository** (the Next.js UI with static data); everything else is design only.

## AI-assisted workflow

- [`PROMPTS.md`](PROMPTS.md): the real prompts, in order, verbatim.
- [`.claude/`](.claude/README.md): three review agents (UI/pixel, accessibility, code quality) and three skills (visual-diff, a11y-check, replace-placeholders).
- [`CLAUDE.md`](CLAUDE.md): standing project rules (originality, honesty about the reference, data-layer rule).
- The agents and skills are provided so the workflow is repeatable in Claude Code; in the chat session that built this project their checklists were applied by hand.

## Testing

See [`docs/QA.md`](docs/QA.md). An optional headless-browser harness lives in
`docs/qa-harness/` (needs `esbuild` and `playwright`, installed with
`--no-save`; see its README).

## Asset attribution

No third-party photographs, fonts, icon packs or images are bundled. Placeholder
photos are generated SVGs; icons and the temporary logo/house/laurel/tag
artwork are simple original drawings. "Airbnb" is a trademark of its owner and
is used here only because the assignment asks for a clone of an Airbnb listing.
Replace the placeholders only with assets you have the right to use.

## Submission note

The assignment asks not to publish this code in a public GitHub repository;
submit it as a ZIP as instructed.
