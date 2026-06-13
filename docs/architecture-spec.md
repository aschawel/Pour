# Pour — architecture & design spec

**Status:** design record, captured during prototyping. Not yet reflected in code.
**Last updated:** June 12, 2026
**Scope:** the product architecture and operational designs worked out to date. Tooling/backend decisions live in `stack-decisions.md`; build status lives in `CHANGELOG.md`.

---

## 1. Product framing

Pour is a bar inventory platform for mid-market restaurant and bar groups (6–20 locations). The competitive wedge is UX/design quality plus a deep **Analytics & Variance** engine, positioned against Craftable and BinWise.

The variance engine is the **north star**. Every other module exists to feed it clean inputs or to act on what it surfaces. This framing is the tie-breaker for design and scope decisions.

---

## 2. Two-plane architecture

The system splits into two planes, not one flat app:

- **Operational plane (store level)** — the six modules (Dashboard, Inventory, Purchasing, Recipes, Analytics, Menu Editor), used by bar managers, head bartenders, and store staff. Executes against the rules.
- **Management / configuration plane (the console)** — bev management + IT only, **hidden from store staff**. Sets the rules the operational plane runs on, and triages the exceptions it produces.

**The loop:** configuration flows *down*; oddities flow *up*. Integrations and the console are *planes/layers*, not peer modules.

**The visibility boundary is itself a control.** Putting depletion thresholds and SKU config behind a management wall means the people being measured can't edit the measurement — a defense against the exact shrinkage the variance engine exists to catch.

---

## 3. Configuration console

The **SKU master is the hub**; the other areas attach to it. The console's whole job is to feed the variance engine clean inputs and to triage its outputs.

- **SKU master** — vintage-agnostic `Product` → `variant` (pack + vintage, each with its own cost) → `identifiers`. Carries the unit-conversion chain (purchase unit → inventory unit → pour unit), which is the basis of all depletion math. Hosts the POS-item ↔ recipe ↔ SKU mapping. Surface for approving/merging OCR-proposed SKUs and aliases. *Shared ownership:* bev owns the catalog, IT owns mapping + hygiene.
- **Depletion levels** — variance *tolerances* per product / category / location / day-part; the sensitivity dial on the engine (too tight → alert fatigue; too loose → shrinkage hides). Bev-management owned. Distinct from par/reorder levels (purchasing-side).
- **Vendors** — accounts, order method (EDI/email/portal/rep), schedules + cutoffs, minimums, terms, lead times; price lists mapped to SKUs; substitution rules (same brand/different size, no sub, different vintage, custom per line). Bev-owned; IT sets up EDI.
- **Pricing / cost analysis** — landed cost per SKU (from OCR'd invoices), cost history, normalized cost-per-oz; theoretical pour cost per recipe, margin / pour-cost % per item, vendor comparison, what-if modeling. Sets the pour-cost % targets Analytics measures against; the NetSuite COGS reconciliation point.
- **Comms hub** — exception triage for variance flags, lifecycle anomalies, purchase-history flags, failed syncs, and staff-consumption patterns. **Open question:** one-way triage vs two-way (route a flag to a store with a note, bartender replies in-thread). Two-way is more powerful but softens the store-invisibility wall.

**Internal split:** IT (integrations, credentials, system) vs bev management (SKUs, depletion, vendors, pricing, triage) → the console needs its own sub-roles.

---

## 4. Depletion model (the core)

Depletion has **three channels**:

1. **Theoretical (POS)** — `units sold × recipe pour × unit conversion`. The expected usage.
2. **Logged non-sale** — staff consumption, comps, tastings, R&D, spillage, guest samples. Real depletion: it decrements on-hand, but it **never enters theoretical** (not a sale) and carries its **own cost line**.
3. **Unexplained variance** — the residual you actually investigate.

**Equation:**

```
unexplained variance = actual depletion − theoretical (POS) − logged non-sale
```

**Actual depletion reconciliation:**

```
actual = opening on-hand + receipts + transfers in − transfers out − closing/audit count
```

Logging the non-sale channel converts what would otherwise read as shrinkage into *accounted* depletion, shrinking the residual to real loss.

### Staff consumption logging

- **Surface:** store-level, **low-friction** (scan-and-confirm, smart defaults). If it takes more than a couple of taps, staff won't log and the variance silently returns.
- **Captures:** product (vintage resolved by bin), amount (partial pour or full unit), reason code (staff/shift drink, tasting/education, R&D, comp, spillage/breakage, sample), employee who consumed + who logged, location/spot, auto-derived cost.
- **Per-SKU policy** (lives in the console, beside depletion levels): **Open** / **Approval-required** / **Prohibited** (allocated/rare — a disappearance *should* trip variance) / **Capped** (N per shift/week).
- **Controls:** named attribution, caps, manager auth on flagged SKUs, and management oversight of patterns (one employee, one product, spikes) → comms hub. The log is also a theft alibi, so it needs its own variance-watching. Store staff log (operational plane); management sets policy + watches (management plane).

---

## 5. Operational designs

### Invoice OCR + forced-vintage gate

Missing vintage is the most common intake error, and OCR can't extract what isn't printed. The correct behavior is to **refuse to resolve** the line rather than guess or null it.

Flow: upload → OCR parses lines → auto-match against the open PO (substitution rules apply) → any line with an unresolved attribute drops into an **exception gate**. Missing vintage forces one of: confirm off the bottle in hand, mark NV where legitimate, bind to a vintage already open, or escalate to the buyer. The gate also catches price changes, pack/unit changes, and quantity mismatches. **Nothing posts to inventory until every exception clears.** Receiving is the right place to force it — the bottle is physically in hand.

### Wine identity model

Manufacturer UPC generally doesn't vary by vintage, so a front-label scan can't distinguish years.

- `Product` (vintage-agnostic) → `variant` (product + pack + vintage, own cost) → `identifiers[]`.
- Manufacturer UPC and any back-label code map to the **Product** (they fan out across vintages); a Pour-generated **internal barcode** encodes vintage and maps to the **variant**.
- Scanning a UPC with multiple vintages on hand → disambiguation prompt, or resolve by bin location.
- **Wedge:** pin vintage once at receiving and bind it to a bin, so a count scan resolves vintage by *where it is* — avoiding labor-heavy per-bottle vintage labels (the incumbent approach).

### Daily theoretical-depletion-vs-audit

The live form of the (still-unbuilt) variance-detail screen, lighter than a full count: management names a target set, store counts just those, system compares.

**Resources required:**
- *Data:* item-level POS sales daily (Toast, gated on the POS→recipe mapping); recipe specs + SKU unit conversions; last-known on-hand for audited SKUs; receipts + inter-location transfers; depletion tolerances; the audit counts.
- *Compute:* a daily scheduled job — the **first genuine need for the deferred background-job layer** (Inngest/Trigger.dev or Supabase cron) — idempotent and tolerant of late-arriving sales; rolls theoretical up per SKU into a Postgres snapshot table so the comparison is a cheap read.
- *Surfaces:* audit initiation (console) + count capture (store-level Daily Audit Queue) + variance breakdown screen + exception path to comms hub.
- *Watch-outs:* comps/voids/spills/staff drinks not in POS; checks open across midnight; free-pour drift (tolerances absorb); partial bottles by tenths/weight; unmapped POS items (surface mapping-coverage % as a health metric); net out transfers or each reads as loss + phantom gain.

---

## 6. Integration layer

A **connector layer** in the foundation — plumbing that feeds the six modules, not a seventh module.

- **Toast (POS) — high priority, sequencing-critical.** Unblocks live actual-vs-theoretical variance. Registers as a Toast *Partner API* account (multi-group). Production access is gated by a partner application + ~1-hour certification call + piloted beta, so **start the application early** — it has external lead time. Token auth, webhooks + polling.
- **NetSuite (ERP) — high priority, upmarket positioning.** Signals targeting the multi-entity end of the range. SuiteTalk REST + token/OAuth2 is fine; the real cost is per-tenant schema divergence → ongoing per-customer configuration, not a one-time build.
- **Backlog:** Square, Lightspeed (POS); QuickBooks (accounting); distributor EDI.

Per connection: auth/credentials, sync health, field mapping (POS items → recipes/SKUs; GL accounts → categories), schedule/webhooks, per-location instances. Failed syncs and unmapped POS items route up to the comms hub.

---

## 7. Open decisions

- **Backend** — recommended Supabase (Postgres + auth + storage + RLS) plus a job runner (Inngest/Trigger.dev) added when integration syncs get real. *Recommended, not yet confirmed* — see `stack-decisions.md`.
- **POS-item mapping seam** — recommend a single shared mapping table that both SKUs and Integrations open into (jointly owned by bev + IT), rather than duplicating the mapping in either. It's the join that makes sales-to-depletion work, so the seam is worth getting right early.
- **Comms hub** — one-way triage vs two-way conversation (see §3).
- **RBAC matrix** — store / bev-management / IT tiers plus console sub-roles; nudges toward Clerk-style organizations or a serious RLS + role-claims design.
