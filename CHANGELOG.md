# Changelog — Pour

All notable changes to this project are recorded here.

This changelog is **version-scoped**: a new entry is added at *tag time*, not
per commit. Each entry corresponds to a git tag and summarizes that version in
human-readable terms. For commit-level detail, see `git log`.

Format loosely follows [Keep a Changelog](https://keepachangelog.com/).
Tags follow `v{major}.{minor}-{scope}` during screen-by-screen prototyping.

---

## [Unreleased]

_Work in progress since the last tag. Promote these notes into a versioned
entry when the next tag is cut._

- _(nothing yet)_

---

## [v0.2-standards] — 2026-06-12

Captures the architecture and tooling design worked out during prototyping.
Documentation only — no changes to the prototype screens or any code.

### Added
- `docs/architecture-spec.md` — two-plane architecture (operational vs.
  management/config console), the SKU-hub configuration model, the three-channel
  depletion model (theoretical / logged non-sale / unexplained), staff
  consumption logging, the invoice-OCR vintage gate, the wine identity model,
  the daily depletion-vs-audit function, the integration layer, and open decisions.
- `docs/stack-decisions.md` — locked app skeleton (Next.js + TS, Tailwind +
  shadcn, Recharts, pnpm, Biome, Vercel) and the recommended backend direction
  (Supabase + a job runner later), pending confirmation.

### Notes
- Design records only; not yet reflected in code.
- Open decisions tracked in the spec: backend confirmation, the POS-item mapping
  seam, comms-hub one-way vs. two-way, and the RBAC matrix.

---

## [v0.1-baseline] — 2026-06-12

Initial import of the Pour prototyping work. Establishes a clean floor to diff
against; no fixes applied as part of the baseline.

### Added
- Analytics & Variance dashboard (hero screen)
- Inventory Count workflow (hero screen)
- Product Lifecycle Map & Tracker (hero screen)
- Open Purchase Orders view (hero screen)
- `calc.html` — cocktail batch calculator

### Known debt
- `calc.html` uses an inconsistent design system (slate/cyan/violet palette,
  `system-ui` fonts) vs. the unified dark theme (Syne / DM Mono) used by the
  hero screens. Flagged for a later unification commit.
- No build step, auth, backend, or tenant isolation — intentional for the
  vanilla HTML/JS prototyping phase.
