# Pour — stack & tooling decisions

**Status:** decisions on paper only — codebase not yet scaffolded. Keep building hero screens in vanilla HTML until ready to consolidate.
**Last updated:** June 12, 2026

---

## Locked: app skeleton

The tools the React/Next.js codebase will be built on when the prototypes graduate into it.

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js (App Router) | Consolidation target for the standalone HTML prototypes |
| Language | TypeScript | Non-negotiable once variance models + POS/ERP payloads are in play |
| Styling | Tailwind CSS | Existing color tokens + dark theme encode into the Tailwind config |
| Components | shadcn/ui | Unstyled, ownable primitives; dress in Syne + DM Mono |
| Charts | Recharts | Already used in the Analytics prototypes; carry over |
| Package manager | pnpm | Faster, stricter than npm |
| Lint / format | Biome | One tool in place of ESLint + Prettier |
| Hosting | Vercel | Branch preview URLs; near-zero Next.js config |
| Repo | GitHub "Pour" | Existing |

**Design-system note:** the Tailwind config is where Syne / DM Mono and the shared color tokens get encoded once, so every shadcn component inherits the look. This is what keeps the UX wedge intact through the migration.

---

## Recommended: backend *(awaiting confirmation)*

**Direction:** start with Supabase; add a dedicated job runner when the integration syncs get real.

**Why it fits Pour:**
- Collapses Postgres + auth + file storage + tenant isolation (row-level security) into one foundation — fastest path to a working multi-tenant app for a small team.
- Plain Postgres underneath, so the variance/analytics SQL is unconstrained and the database stays portable (low lock-in).
- Storage bucket covers invoice-scan images for the Purchasing OCR flow.

**Where it bends — plan for it, don't solve it yet:**
- Background / long-running jobs (Toast pulls, NetSuite read/write syncs) belong on a durable job runner — **Inngest** or **Trigger.dev** — added alongside Supabase. Additive, not a re-architecture.
- Multi-location RBAC (e.g. the ops-director role) is built on top of Supabase auth. If org/role modeling gets complex, reach for **Clerk** — its org primitives map cleanly to group → location → role, and can pair with a Supabase or Neon database.

**Alternative considered — assemble it** (Next.js + Neon Postgres + Drizzle ORM + Clerk/Auth.js): more control, best-in-class per layer, Clerk orgs map neatly to the tenancy model — but more moving parts to maintain solo and slower to a first working screen. Revisit if Supabase's auth or jobs limits bite early.

---

## Deferred — decide when wiring real data

- Exact DB schema for inventory / variance / product lifecycle
- Multi-tenancy model details (row-level-security policy design)
- Auth role & permission matrix
- Integration sync architecture (queue, retry, scheduling) — pairs with the job-runner choice above
- Billing / subscription
- Testing strategy

**Revisit trigger:** when the first hero screen needs live data instead of mock data.
