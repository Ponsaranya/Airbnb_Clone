# Project guide for Claude Code

Original take-home implementation of an Airbnb-style listing page (listing page,
photo tour, lightbox). Desktop only (~1440px). Next.js App Router + React +
TypeScript + CSS Modules, no backend, no runtime UI libraries.

## Non-negotiables
- **Originality:** never copy code or assets from the reference site or from
  existing clones. The reference is only a visual/behavioural specification.
- **Honesty about the reference:** the live reference was not inspectable when
  this was built; only three screenshots from the assignment exist. Anything not
  visible in them must stay marked `ASSUMED` / `UNVERIFIED` / `TODO(reference)`.
  Never claim pixel parity that was not measured.
- **Data layer owns content:** photos, categories, listing facts live in
  `src/data/*`. Components must not hard-code photo or listing information.
- **One overlay state system:** `GalleryProvider` (`src/components/gallery`)
  owns tour/lightbox state. Do not add a second one.
- **Keep it small:** no new dependencies without a strong reason.

## Conventions
- Comments in CSS/TSX mark values as `MEASURED` (from a screenshot), `DERIVED`,
  `ASSUMED` or `UNVERIFIED`. Preserve and update these when changing values.
- Motion durations/easings only via `--motion-*` tokens in `src/styles/tokens.css`;
  everything must respect `prefers-reduced-motion`.
- Every icon-only button needs an accessible name; overlays need
  `role="dialog"`, `aria-modal="true"`, focus trap, focus return, scroll lock.

## Commands
`npm install` · `npm run dev` · `npm run lint` · `npm run typecheck` · `npm run build`
Optional browser checks: see `docs/qa-harness/README.md`.

## AI workflow
Project agents and skills live in `.claude/` (see `.claude/README.md`).
The prompt history is in `PROMPTS.md`; QA status is in `docs/QA.md`.
