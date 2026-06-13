# Pour

Bar inventory management for mid-market restaurant & bar groups (6–20 locations).
The product's center of gravity is a deep **Analytics & Variance** engine; every
other module exists to feed it clean inputs or to act on what it surfaces.

Competitive wedge: UX/design quality plus variance depth, positioned against
Craftable and BinWise.

## Status

Prototyping phase — vanilla HTML/JS hero screens validating UX before
consolidating into a Next.js / TypeScript app. No build step, backend, or auth
yet; that's intentional for this phase (see `docs/stack-decisions.md`).
Architecture and tooling are decided on paper; the codebase migration has not
started.

## Repo layout

```
README.md          — this file
CHANGELOG.md       — version-scoped, updated at tag time
CONTRIBUTING.md    — commit format + release/tag ritual
docs/
  stack-decisions.md     — locked tooling + backend recommendation
  architecture-spec.md   — two-plane architecture, config console, depletion model, open decisions
prototypes/
  *.html                 — hero screens + calc.html
```

## Conventions

Commits and version tags follow the ritual in `CONTRIBUTING.md`: Conventional
Commits, `v{major}.{minor}-{scope}` tags during prototyping, and the changelog
updated at tag time (not per commit). Reference any past version with
`git checkout <tag>`.

## Current build

Four hero screens — Analytics & Variance, Inventory Count, Product Lifecycle Map
& Tracker, Open Purchase Orders — plus `calc.html` (cocktail batch calculator,
carrying a known design-system debt). See `CHANGELOG.md` → `v0.1-baseline`.
