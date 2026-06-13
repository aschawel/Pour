# Pour — Site Map (exhaustive)

**Status:** design record, prototyping phase. **Last updated:** 2026-06-13.
**Companion to:** `docs/architecture-spec.md`, the roadmap doc, and the version tree.

This is the full page inventory across both planes, one block per leaf. It is the
human-readable spine for routing and the data model when the codebase migrates to
Next.js. Modules are headings; their sub-pages are the leaves.

---

## How to read a leaf

Each leaf carries:

- **Type** — route · single page or module sub-page
- **Category** — plane + finer operational category
- **Status** — ✅ built · ⬜ to build · ◐ decision pending
- **Permissions** — role tiers (see roster below)
- **Scope** — `location` · `client` · `cross-location`
- **Data objects** — core entities read/written
- **Integrations** — potential connectors
- **Cross-leaf** — which other leaves it reads from / writes to / hands off to
- **Analytics** — direction of the relationship to the variance core
- **Guardrails** — hard invariants that must survive migration
- **Function** — detailed outline of what the page does

### Role roster

| Role | Plane | Scope of authority |
|---|---|---|
| `staff` | operational | counting, logging, submitting |
| `lead` | operational | bar lead / head bartender / sommelier — storage, propose recipe changes, run batches, finalize counts |
| `manager` | operational | GM / ops — operational approvals, location oversight |
| `bev_director` | admin/console | SKU master, recipes, analytics, tolerances |
| `purchasing` | admin/console | vendors, non-inventory price authority |
| `finance` | admin/console | accounting, period locks, exports, adjustment approvals |
| `it_admin` | admin/console | integrations, roles/permissions, system |

### Analytics-relationship vocabulary

`feeds theoretical` · `feeds actual` · `reads variance` · `owns a reconciliation term` ·
`feeds cost basis` · `balanced (no variance)` · `none/indirect`

Core reconciliation the map serves:
`unexplained variance = actual depletion − theoretical (POS) − logged non-sale`

---

# Operational plane — what teams use daily

## Dashboard *(single page)*

**Type:** `/dashboard` · single page
**Category:** operational · landing / synthesis
**Status:** ◐ decision (whether distinct from the Analytics & Variance screen)
**Permissions:** any authenticated operational role; content filtered by role
**Scope:** `location` (with `cross-location` rollup mode for `manager`/`bev_director`)
**Data objects:** KPI aggregates, Alert, PO summary — read-only, no writes
**Integrations:** none directly (consumes downstream surfaces)
**Cross-leaf:** links out to Analytics, Inventory, Purchasing, Lifecycle; surfaces variance-engine alerts
**Analytics:** reads variance (consumes KPIs, watchlist, alert feed)
**Guardrails:** read-only — the page writes nothing
**Function:**
- At-a-glance KPIs: pour cost %, inventory value, variance $, COGS, items at par
- Pour-cost trend chart; category variance; variance watchlist with sparklines
- Alert feed (severity-coded); pending POs panel
- Location switcher + multi-location rollup mode
- Pure synthesis; every tile links out for depth

## Inventory *(module — counting)*

The act of counting. Distinguishes daily theoretical depletion from management-initiated audit — different feeds, different compute.

### Count session
**Type:** `/inventory/count` · sub-page · **Status:** ✅ built
**Category:** operational · counting
**Permissions:** `staff`+ to count; `lead` to finalize/close
**Scope:** `location` · **Data objects:** Count, CountLine, StorageLocation, SKU (read), Prep (countable)
**Integrations:** none; device: mobile/tablet, offline-tolerant
**Cross-leaf:** reads topology from Cellar Manager; reads SKU/prep identity from Console/Recipes; writes on-hand → Analytics
**Analytics:** feeds actual (on-hand → actual depletion)
**Guardrails:** same SKU may live in multiple spots (sum, never overwrite); chain-of-custody; daily ≠ audit
**Function:**
- Location-first, spot-by-spot count across a flexible-depth hierarchy
- Multi-spot SKUs; count-by-area; partial / rolling counts
- Close → snapshot that feeds the variance engine

### Daily Audit Queue
**Type:** `/inventory/audit-queue` · sub-page · **Status:** ✅ built (within Inventory)
**Category:** operational · targeted counting
**Permissions:** `staff`+ count; `lead` manage queue · **Scope:** `location`
**Data objects:** AuditQueueItem, Count (partial), SKU, VarianceFlag (read)
**Integrations:** none
**Cross-leaf:** auto-seeded by the variance engine (recurring-variance SKUs) + manual adds; writes spot counts → actual
**Analytics:** feeds actual + reads variance (queue seeded by flags)
**Guardrails:** targeted, not comprehensive; distinct from full audit
**Function:**
- Short daily count of recurring-variance / high-risk SKUs
- Auto-suggested by the engine plus manual additions
- Tight reconcile loop that keeps the watchlist honest day to day

### Full audit
**Type:** `/inventory/audit` · sub-page · **Status:** ⬜ to build
**Category:** operational · management-initiated audit
**Permissions:** `manager`/`bev_director` initiate; `staff` execute · **Scope:** `location` (schedulable `cross-location`)
**Data objects:** Audit, CountLine (full), SKU, Prep
**Integrations:** none
**Cross-leaf:** reads topology (Cellar); writes full snapshot → Analytics
**Analytics:** feeds actual (full physical truth-up)
**Guardrails:** separate feed & compute from daily depletion; freezes counts during audit
**Function:**
- Scheduled or initiated full physical count
- Freeze → count-all → reconcile → sign-off
- Sets the actual baseline the daily loop drifts from

### Count history
**Type:** `/inventory/history` · sub-page · **Status:** ⬜ to build
**Category:** operational · review
**Permissions:** `lead`+, `manager` · **Scope:** `location`
**Data objects:** Count (historical), variance results
**Integrations:** none
**Cross-leaf:** reads closed counts; links to Analytics variance detail
**Analytics:** reads variance (per-count outcomes)
**Guardrails:** immutable historical snapshots
**Function:** browse past counts/audits, their variance outcomes, per-spot trends; export.

## Cellar Manager *(module — storage)*

Managing storage, not counting it. Bar and wine teams share the engine, different mental models. Straddles operational/admin (topology editing is gated higher).

### Storage map / topology
**Type:** `/cellar/topology` · sub-page · **Status:** ⬜ to build
**Category:** operational ↔ admin · storage structure
**Permissions:** `lead` (place product) / `bev_director`/`it_admin` (edit structure)
**Scope:** `location` (structure may be `client`-templated)
**Data objects:** StorageLocation (hierarchy), Zone, Bin, Tap
**Integrations:** none
**Cross-leaf:** defines the hierarchy consumed by Inventory (count structure), Receiving (bin assignment), Lifecycle (storage stage)
**Analytics:** indirect (underpins FIFO/misfile anomalies)
**Guardrails:** structure edits gated higher (plane straddle); never orphan counted product
**Function:**
- Define & view cellars → rooms → shelves/tiers → bins → kegs/taps, flexible depth
- Templated per location; the spatial backbone everything else references

### Bar storage
**Type:** `/cellar/bar` · sub-page · **Status:** ⬜ to build
**Category:** operational · storage (bar)
**Permissions:** `lead`; `staff` view · **Scope:** `location`
**Data objects:** StorageLocation, SKU placement, Par
**Integrations:** none
**Cross-leaf:** reads topology; informs count; pars feed reorder (Purchasing)
**Analytics:** feeds par / availability signals
**Guardrails:** par by spot; service-spot definitions
**Function:** manage bar service spots, pars, rotation, restock-from-bulk; well / back-bar / keg / walk-in views.

### Wine cellar
**Type:** `/cellar/wine` · sub-page · **Status:** ⬜ to build
**Category:** operational · storage (wine)
**Permissions:** `lead` / sommelier · **Scope:** `location`
**Data objects:** StorageLocation (bins), WineSKU (vintage), bottle-level (optional), Consignment flag (read)
**Integrations:** none (wine DBs a future option)
**Cross-leaf:** vintage from Receiving (gate); consignment flag from Accounting; bottle-level → Lifecycle
**Analytics:** feeds actual (high-value tracking), reads variance
**Guardrails:** vintage required (hard gate); drink-window; consignment ownership visible
**Function:** bin-by-bin wine map, vintage-aware, optional bottle-level, drink windows, values, consignment-held visibility.

### Internal movements / rotation
**Type:** `/cellar/movements` · sub-page · **Status:** ⬜ to build
**Category:** operational · intra-location movement
**Permissions:** `lead`+ · **Scope:** `location` (inter-location lives in Accounting → Transfers)
**Data objects:** Movement, StorageLocation, SKU
**Integrations:** none
**Cross-leaf:** distinct from inter-location Transfers (Accounting); feeds Lifecycle (storage → service)
**Analytics:** balanced (no variance)
**Guardrails:** balanced movement — never creates variance; FIFO-aware
**Function:** move product between spots within a location, restock service from bulk, rotation; logs chain-of-custody.

## Purchasing *(module)*

### Orders / Open POs
**Type:** `/purchasing/orders` · sub-page · **Status:** ✅ built
**Category:** operational · purchasing
**Permissions:** `lead` create; `manager`/`bev_director` approve over $ thresholds
**Scope:** `location` (cross-location ordering for groups)
**Data objects:** PurchaseOrder, POLine, Vendor (read), SKU
**Integrations:** distributor EDI (future), vendor catalogs
**Cross-leaf:** pars/reorder from Cellar & Reorder; → Receiving
**Analytics:** feeds cost basis (committed spend); reads (purchase-history flags)
**Guardrails:** substitution rules per line (same brand/size · none · vintage · custom); purchase-history flags (price/unit/frequency anomalies)
**Function:** build/approve POs, per-line substitution, history flagging, ops-director multi-location toggle.

### Receiving
**Type:** `/purchasing/receiving` · sub-page · **Status:** ⬜ to build
**Category:** operational · receiving
**Permissions:** `lead` / receiver · **Scope:** `location`
**Data objects:** Receipt, Invoice (OCR), POLine (read), SKU, StorageLocation (bin), Vintage
**Integrations:** OCR invoice, distributor EDI / ASN
**Cross-leaf:** receives vs PO; assigns bin (Cellar topology); cost → Console/Accounting; → Lifecycle (receiving stage)
**Analytics:** feeds cost basis (actual landed cost) + inventory in
**Guardrails:** **vintage hard gate** (flag missing vintage at intake, never silent); bin assignment at receiving (UPC-limitation wedge vs BinWise)
**Function:** receive vs PO, OCR invoice match, discrepancy flags, bin assignment, vintage resolution, landed cost.

### Reorder / history
**Type:** `/purchasing/reorder` · sub-page · **Status:** ⬜ to build
**Category:** operational · purchasing
**Permissions:** `lead`+, `bev_director` · **Scope:** `location` / `cross-location`
**Data objects:** ReorderSuggestion, Par (read), usage (read), PurchaseHistory
**Integrations:** none
**Cross-leaf:** reads pars (Cellar) + usage (Analytics) → suggestions → Orders
**Analytics:** reads (usage-driven reorder)
**Guardrails:** —
**Function:** par-driven reorder suggestions, order-to-par quantities, purchase-history trends, price-change watch.
*(Vendor master lives in the Configuration Console.)*

## Recipes *(module — the depletion-spec engine)*

The layer that translates a POS sale into theoretical depletion. Everything the variance moat claims rests on it being correct.

### Recipe library
**Type:** `/recipes` · sub-page · **Status:** ⬜ to build
**Category:** operational · recipe authoring
**Permissions:** `lead` view; `bev_director` edit · **Scope:** `client` (location overrides)
**Data objects:** Recipe, RecipeVersion
**Integrations:** none
**Cross-leaf:** feeds depletion via POS mapping; costing reads SKU cost
**Analytics:** feeds theoretical
**Guardrails:** —
**Function:** browse/search recipes by type (cocktail · prepared · batch · retail), status, pour cost, version state.

### Recipe editor
**Type:** `/recipes/:id/edit` · sub-page · **Status:** ⬜ to build
**Category:** operational · recipe authoring
**Permissions:** `bev_director` edit; `lead` propose-only (→ approval per Accounting policy) · **Scope:** `client`
**Data objects:** Recipe, RecipeComponent (sourcing class), SKU/Prep/External ref, RecipeVersion
**Integrations:** none
**Cross-leaf:** components → SKU master / Prep / External (Accounting non-inv pricing); changes gated by Accounting period policy; feeds Lifecycle & Analytics
**Analytics:** feeds theoretical (the core depletion spec)
**Guardrails:** component **sourcing class** (inventoried / prep / external); **effective-dated version** (model primitive); change governance + audit
**Function:**
- Type-aware spec editor; ingredients → SKU or sub-recipe; quantity + UoM
- Garnish / non-pour items; yield + loss; pour-cost rollup
- Sourcing class per component drives cost/count/variance treatment

### Batch & prep builder
**Type:** `/recipes/prep` · sub-page · **Status:** ⬜ to build *(absorbs calc.html)*
**Category:** operational · prep / production
**Permissions:** `lead` run batch; `bev_director` define · **Scope:** `location` (production happens on-site)
**Data objects:** PrepRecipe (BOM), ProductionEvent, Intermediate SKU, yield
**Integrations:** none
**Cross-leaf:** production = balanced pair (deplete components, produce intermediate); intermediate becomes countable (Inventory/Cellar); needs "made" SKU type in Console
**Analytics:** feeds theoretical (prep depletion) + yield variance (prep-side)
**Guardrails:** balanced production event (no double-count); intermediate is first-class inventory; yield + loss
**Function:**
- Define prep BOM, batch yield, scale-to-vessels (the calc.html math, rehomed)
- Log production (consumes components now); produce / break-down intermediates

### Size & variant matrix
**Type:** `/recipes/:id/variants` · sub-page · **Status:** ⬜ to build
**Category:** operational · recipe authoring
**Permissions:** `bev_director` · **Scope:** `client`
**Data objects:** RecipeVariant (size), per-size quantities
**Integrations:** none
**Cross-leaf:** size deltas consumed in depletion resolution (POS size)
**Analytics:** feeds theoretical (size-correct depletion)
**Guardrails:** —
**Function:** grid of size variants (12/16/20 oz), per-size component quantities; one recipe, many sizes.

### POS mapping & modifiers
**Type:** `/recipes/pos-mapping` · sub-page · **Status:** ⬜ to build
**Category:** operational ↔ integration · mapping
**Permissions:** `bev_director`, `it_admin` · **Scope:** `client` / `location` (POS items per location)
**Data objects:** POSItemMap, ModifierRule, Recipe (ref)
**Integrations:** POS (Toast / Square / Lightspeed) — **normalized via adapter**
**Cross-leaf:** binds normalized POS items + modifiers → recipe adjustments; consumes Integrations adapter output; feeds depletion
**Analytics:** feeds theoretical (turns sales into depletion)
**Guardrails:** consumes **normalized** modifier events only (integration isolation) — never vendor payloads
**Function:** map POS PLUs → recipes; modifier → depletion-delta rules; substitutions; size routing; unmapped-item alerts.

### Versions & scheduled changes
**Type:** `/recipes/versions` · sub-page · **Status:** ⬜ to build
**Category:** operational · temporal control
**Permissions:** `lead` propose; `bev_director`/`finance` approve (per Accounting policy) · **Scope:** `client`
**Data objects:** RecipeVersion (effective-dated), ScheduledChange, CorrectionProposal
**Integrations:** none
**Cross-leaf:** retroactive corrections gated by Accounting period locks/lookback; triggers recompute job; writes audit (Team Manager / Accounting)
**Analytics:** feeds theoretical (version-correct at sale date); recompute restates historical variance
**Guardrails:** effective-dating model primitive; retroactive = submit-approve; period-lock gate; recompute job
**Function:**
- Version history; scheduled forward changes; single-day overrides
- Retroactive corrections with impact preview (affected sales + variance delta) → approval routing

### Costing & yield
**Type:** `/recipes/costing` · sub-page · **Status:** ⬜ to build
**Category:** operational · costing / analysis
**Permissions:** `lead` view, `bev_director` · **Scope:** `client` / `location`
**Data objects:** Recipe cost rollup, YieldActual vs expected
**Integrations:** none
**Cross-leaf:** reads SKU cost (Console) + external standard cost (Accounting); expected-vs-actual yield from production events
**Analytics:** feeds cost basis + prep-yield variance signal
**Guardrails:** —
**Function:** pour cost % across recipes, cost sensitivity to price changes, expected vs actual yield (prep variance), margin view.

## Menu Editor *(module)*

### Composer
**Type:** `/menu/compose` · sub-page · **Status:** ⬜ to build (specced)
**Category:** operational · menu authoring
**Permissions:** `bev_director`, `manager` · **Scope:** `client` / `location`
**Data objects:** Menu, MenuItem, Section, StyleLock, Recipe (ref), SKU availability (read)
**Integrations:** POS (menu/PLU sync, optional), print/PDF
**Cross-leaf:** items → recipes (depletion); live stock overlay reads Inventory/Cellar; 86 reads availability
**Analytics:** feeds theoretical (menu-driven depletion) + reads availability
**Guardrails:** style-locking (brand consistency)
**Function:** compose menus, sections, pricing, drag items, brand style-lock, live stock overlay, 86 board.

### Preview / export / versions
**Type:** `/menu/export` · sub-page · **Status:** ⬜ to build
**Category:** operational · menu publishing
**Permissions:** `bev_director`, `manager` · **Scope:** `client` / `location`
**Data objects:** Menu version, ExportArtifact (HTML/PDF)
**Integrations:** print, web embed, PDF
**Cross-leaf:** publishes from Composer; version control
**Analytics:** none
**Guardrails:** versioned publish
**Function:** preview, HTML/PDF export, version history, publish/rollback.

## Analytics *(module — the moat)*

### Variance dashboard
**Type:** `/analytics` · sub-page · **Status:** ✅ built
**Category:** operational · analytics
**Permissions:** `lead` view, `bev_director`, `manager` · **Scope:** `location` + `cross-location` rollup
**Data objects:** VarianceResult, KPI, Watchlist
**Integrations:** consumes POS (theoretical via Recipes) + counts (actual)
**Cross-leaf:** reads theoretical (Recipes/POS), actual (Inventory), logged non-sale (Accounting → Manual Depletions); seeds Audit Queue
**Analytics:** **is** the analytics surface
**Guardrails:** reconciles only over invoiced ∩ counted SKUs; excludes non-inventoried externals
**Function:** pour cost %, theoretical vs actual, variance watchlist, category/SKU/location, alerts. Serves `unexplained = actual − theoretical(POS) − logged non-sale`.

### Variance detail / reports
**Type:** `/analytics/detail` · sub-page · **Status:** ⬜ to build
**Category:** operational · analytics
**Permissions:** `bev_director`, `finance` (read) · **Scope:** `location` / `cross-location`
**Data objects:** VarianceResult (drill), Report
**Integrations:** export to finance (via Accounting)
**Cross-leaf:** drill from dashboard; per-SKU drill into Lifecycle Map; report → Accounting export
**Analytics:** is the analytics detail surface
**Guardrails:** —
**Function:** per-SKU/category/location drill, time-series, scheduled reports, export.

### Consumption review
**Type:** `/analytics/consumption` · sub-page · **Status:** ⬜ to build
**Category:** operational ↔ admin · analytics (staff consumption)
**Permissions:** `manager`, `bev_director`, `finance` · **Scope:** `location` / `cross-location`
**Data objects:** ConsumptionEvent, Policy (per-SKU), Attribution
**Integrations:** none
**Cross-leaf:** reads Manual Depletions (Accounting) + per-SKU policy (Console)
**Analytics:** reads the logged-non-sale term; flags policy violations
**Guardrails:** per-SKU policy (Open / Approval-required / Prohibited / Capped); **named attribution** (anti-alibi)
**Function:** review staff consumption by person / SKU, policy compliance, trends; the human-readable face of the non-sale term.

## Product Lifecycle Map ★ *(single page — primary moat)*

**Type:** `/lifecycle` · single page · **Status:** ✅ built
**Category:** operational · cross-cutting lens
**Permissions:** `lead` view, `bev_director` · **Scope:** `location` (per-SKU / per-bottle)
**Data objects:** LifecycleTrace (reads receiving, storage, service, depletion), Anomaly
**Integrations:** consumes POS depletion, receiving, storage feeds
**Cross-leaf:** reads Receiving (Purchasing), storage (Cellar), depletion (Analytics/POS); surfaces anomalies → alerts
**Analytics:** reads + feeds (anomaly detection complements variance)
**Guardrails:** anomaly detection — short pours, FIFO violations, misfiled product
**Function:** trace every bottle distributor → receiving → storage → service → depletion; anomaly feed; per-SKU/bottle timeline. Read-only.

---

# Admin & support plane — permission-gated

## Configuration Console *(module — PROPOSED, pending confirmation)*

The locked P2-M1 hub. The SKU master is the spine the variance engine reads from. Flagged as an addition for your sign-off.

### SKU master
**Type:** `/console/skus` · sub-page · **Status:** ⬜ to build (P2-M1)
**Category:** admin/console · configuration (hub)
**Permissions:** `bev_director`, `it_admin` · **Scope:** `client`
**Data objects:** SKU (master), UoM, category, sourcing-class default, "made" type for preps
**Integrations:** distributor catalogs, POS item linkage
**Cross-leaf:** the hub every surface reads SKU identity from; defines prep ("made") + external classes
**Analytics:** feeds (defines what variance reconciles)
**Guardrails:** single source of SKU truth
**Function:** create/manage SKUs, categories, UoM, pack config, sourcing defaults.

### Depletion tolerances
**Type:** `/console/tolerances` · sub-page · **Status:** ⬜ to build
**Category:** admin/console · variance config
**Permissions:** `bev_director` · **Scope:** `client` (location override)
**Data objects:** TolerancePolicy per SKU/category
**Integrations:** none
**Cross-leaf:** feeds Analytics flag thresholds
**Analytics:** feeds (variance flag thresholds)
**Guardrails:** tolerance ≠ target; documented bands
**Function:** set acceptable variance bands per SKU/category; drives watchlist flagging.

### Vendor master
**Type:** `/console/vendors` · sub-page · **Status:** ⬜ to build
**Category:** admin/console · configuration
**Permissions:** `purchasing`, `bev_director` · **Scope:** `client`
**Data objects:** Vendor, pricing, terms, catalog
**Integrations:** distributor EDI, vendor portals
**Cross-leaf:** vendors used by Purchasing (POs); pricing feeds cost
**Analytics:** feeds cost basis
**Guardrails:** —
**Function:** manage vendors, catalogs, pricing, terms, substitution defaults.

### Cost & pricing analysis
**Type:** `/console/costing` · sub-page · **Status:** ⬜ to build
**Category:** admin/console · analysis
**Permissions:** `bev_director`, `finance` · **Scope:** `client`
**Data objects:** cost trend, price change, margin
**Integrations:** none
**Cross-leaf:** reads SKU/vendor cost; feeds Recipes costing
**Analytics:** feeds cost basis
**Guardrails:** —
**Function:** SKU-level cost/price trends, vendor compare, margin modeling.

## Toolbox *(single page — open access)*

**Type:** `/toolbox` · single page · **Status:** ✅ built (skeleton)
**Category:** support · utilities (ungated)
**Permissions:** all roles (read-only) · **Scope:** stateless
**Data objects:** none persisted
**Integrations:** none (optional SKU-aware mode reads SKU cost later)
**Cross-leaf:** standalone
**Analytics:** none (read-only, no writes)
**Guardrails:** read-only — never writes inventory or variance
**Function:** registry-driven calculators (pour cost, margin/markup, case→oz, volume convert, dilution); extensible skeleton.

## Integrations *(module — gated)*

### Connections directory
**Type:** `/integrations` · sub-page · **Status:** ⬜ to build
**Category:** admin/console · integrations
**Permissions:** `it_admin` · **Scope:** `client`
**Data objects:** Connector, ConnectionStatus
**Integrations:** Toast, NetSuite, QuickBooks, Square, Lightspeed, distributor EDI, kitchen inventory (future)
**Cross-leaf:** provides normalized feeds to POS mapping (Recipes), Receiving (Purchasing), Finance Export (Accounting)
**Analytics:** indirect (data source)
**Guardrails:** **adapter isolation** — vendor logic never reaches UI
**Function:** directory of available/connected integrations, status, connect/disconnect.

### Per-connector config
**Type:** `/integrations/:id` · sub-page · **Status:** ⬜ to build
**Category:** admin/console · integrations
**Permissions:** `it_admin` · **Scope:** `client`
**Data objects:** Connector config, FieldMapping, credentials (secure)
**Integrations:** per-connector
**Cross-leaf:** defines the normalization mapping consumed downstream
**Analytics:** indirect
**Guardrails:** credentials secured; mapping normalizes to canonical shapes
**Function:** auth, field mapping, sync settings per connector.

### Sync activity
**Type:** `/integrations/sync` · sub-page · **Status:** ⬜ to build
**Category:** admin/console · integrations
**Permissions:** `it_admin` · **Scope:** `client`
**Data objects:** SyncJob, JobLog
**Integrations:** job runner (Inngest / Trigger.dev)
**Cross-leaf:** surfaces sync health for all feeds
**Analytics:** indirect
**Guardrails:** —
**Function:** sync job logs, schedules, retries, error surfacing.

## Team Manager *(module — gated)*

### People
**Type:** `/team/people` · sub-page · **Status:** ⬜ to build
**Category:** admin · access management
**Permissions:** `it_admin`; `manager` (location) · **Scope:** `client` / `location`
**Data objects:** User, LocationAssignment, RoleAssignment
**Integrations:** SSO / identity (future)
**Cross-leaf:** identities + role tiers consumed by every gated action and approval
**Analytics:** none
**Guardrails:** —
**Function:** invite/manage users, assign locations & roles.

### Roles & permissions
**Type:** `/team/roles` · sub-page · **Status:** ⬜ to build
**Category:** admin · access management (defines gating)
**Permissions:** `it_admin` · **Scope:** `client`
**Data objects:** Role, Permission, plane-split
**Integrations:** none
**Cross-leaf:** defines the gating every surface enforces; encodes the two-plane split
**Analytics:** none
**Guardrails:** enforces console vs operational plane split; proposer ≠ approver rules
**Function:** define roles, per-surface permissions, approval tiers.

### Audit log
**Type:** `/team/audit` · sub-page · **Status:** ⬜ to build
**Category:** admin · audit
**Permissions:** `it_admin`; `finance` (finance slice) · **Scope:** `client` / `location`
**Data objects:** AuditEvent (substrate)
**Integrations:** none
**Cross-leaf:** the audit substrate; Accounting → Change Log is a finance-scoped view of this
**Analytics:** none
**Guardrails:** immutable; covers access, config, adjustments
**Function:** who-did-what across access, config, and adjustments; filterable; immutable.

## Accounting *(module — gated)*

Control plane for every value-affecting event that isn't a plain POS sale or a plain invoice. Owns the policy that gates retroactive corrections, manual depletions, and transfers across the product.

### Period controls
**Type:** `/accounting/periods` · sub-page · **Status:** ⬜ to build
**Category:** admin · financial control (policy hub)
**Permissions:** `finance` · **Scope:** `client` / `location`
**Data objects:** AccountingPeriod, Lock, LookbackPolicy
**Integrations:** none
**Cross-leaf:** gates Recipes retroactive corrections, Manual Depletions, Transfers
**Analytics:** governs when variance restatement (recompute) is allowed
**Guardrails:** locked periods need elevated permission; lookback window defined here
**Function:** define periods, lock/close, set retroactive lookback + approval tiers.

### Non-inventory pricing
**Type:** `/accounting/standard-costs` · sub-page · **Status:** ⬜ to build
**Category:** admin · financial control (cost authority)
**Permissions:** `finance` set; `purchasing` approve · **Scope:** `client`
**Data objects:** StandardCost (external / non-inventoried components)
**Integrations:** procurement software, kitchen inventory (future)
**Cross-leaf:** supplies cost for external-class recipe components (Recipes costing)
**Analytics:** feeds cost basis **only** — excluded from variance
**Guardrails:** price changes need purchasing authority; external inputs fenced out of variance
**Function:** maintain standard/transfer costs for non-inventoried inputs; approval routing; future procurement connector.

### Transfers
**Type:** `/accounting/transfers` · sub-page · **Status:** ⬜ to build
**Category:** admin · value movement
**Permissions:** `lead` initiate; `manager`/`finance` approve · **Scope:** `cross-location` + inter-department
**Data objects:** Transfer (out/in pair), valuation
**Integrations:** none
**Cross-leaf:** inter-location (distinct from Cellar intra-location movements); inter-department (bar ↔ kitchen consumption)
**Analytics:** **balanced — must not create variance**
**Guardrails:** balanced paired movement; valuation moves with product
**Function:** inter-location & inter-department transfers, approval, valuation, transfer log.

### Manual depletions
**Type:** `/accounting/manual-depletions` · sub-page · **Status:** ⬜ to build
**Category:** admin · adjustment control
**Permissions:** `staff`/`lead` submit; `manager`/`finance` approve · **Scope:** `location`
**Data objects:** ManualDepletion (waste / breakage / comp / consumption), Attribution
**Integrations:** none
**Cross-leaf:** feeds the logged-non-sale term to Analytics; consumption detail → Consumption Review
**Analytics:** **owns a reconciliation term** (logged non-sale)
**Guardrails:** approval + named attribution (anti-alibi); the only legitimate non-sale channel
**Function:** submit/approve back-end depletions (waste, breakage, comps, staff consumption); the governed source of the logged-non-sale term.

### Change log
**Type:** `/accounting/changes` · sub-page · **Status:** ⬜ to build
**Category:** admin · finance audit
**Permissions:** `finance` · **Scope:** `client` / `location`
**Data objects:** finance-scoped AuditEvent view
**Integrations:** none
**Cross-leaf:** filtered view of the Team Manager audit substrate; tracks corrections / prices / transfers / depletions / locks
**Analytics:** tracks variance-restatement events
**Guardrails:** immutable
**Function:** finance-scoped log of all book-affecting changes with $ impact.

### Finance export
**Type:** `/accounting/export` · sub-page · **Status:** ⬜ to build
**Category:** admin · finance handoff
**Permissions:** `finance` · **Scope:** `client` / `location`
**Data objects:** ExportPackage, GLMapping, period summaries
**Integrations:** NetSuite, QuickBooks (via Integrations connectors)
**Cross-leaf:** composes the export (mapping) consuming an Integrations connection; reads valuation / COGS / variance summaries
**Analytics:** reads (COGS, valuation, variance summaries)
**Guardrails:** Integrations owns the connection, Accounting owns the mapping/composition (isolation)
**Function:** build/define exports (journal entries, COGS, valuation, period close), push to ERP, export history.

### Consignment wine ★ *(sub-module)*

Wine physically on-site but owned by the consignor until sold. Distinct ownership and accounting; an upmarket differentiator.

#### Agreements & intake
**Type:** `/accounting/consignment/agreements` · leaf · **Status:** ⬜ to build
**Category:** admin · consignment (wine)
**Permissions:** `finance`, `bev_director`/sommelier · **Scope:** `client` / `location`
**Data objects:** ConsignmentAgreement, Consignor terms, ConsignmentIntake
**Integrations:** distributor EDI (consignment)
**Cross-leaf:** intake = zero-cost receiving variant (Purchasing); flags stock in Wine Cellar (Cellar)
**Analytics:** cost treatment differs (not an owned asset)
**Guardrails:** not owned inventory (off balance sheet until sold)
**Function:** per-consignor agreements/terms; consignment intake (no invoice / zero-cost); bottle flagging.

#### On-hand (held)
**Type:** `/accounting/consignment/on-hand` · leaf · **Status:** ⬜ to build
**Category:** admin · consignment
**Permissions:** `finance`, sommelier · **Scope:** `location`
**Data objects:** ConsignmentStock (held, not owned)
**Integrations:** none
**Cross-leaf:** mirrors Wine Cellar held stock; depletion triggers a payable
**Analytics:** depletion reconciles but with distinct cost
**Guardrails:** ownership = consignor until sold
**Function:** view consignment stock held, aging, value-at-risk, sell-through.

#### Settlement & billing
**Type:** `/accounting/consignment/settlement` · leaf · **Status:** ⬜ to build
**Category:** admin · consignment
**Permissions:** `finance` · **Scope:** `client`
**Data objects:** Settlement, Payable, SalesSinceLast
**Integrations:** ERP (payables), distributor
**Cross-leaf:** sale → payable to consignor; settlement → Finance Export
**Analytics:** COGS recognized at sale
**Guardrails:** pay-on-sale; periodic settlement reconcile
**Function:** periodic settlement — what sold, what's owed, generate payables, reconcile with consignor.

#### Returns
**Type:** `/accounting/consignment/returns` · leaf · **Status:** ⬜ to build
**Category:** admin · consignment
**Permissions:** `finance`, sommelier · **Scope:** `location`
**Data objects:** ConsignmentReturn
**Integrations:** none
**Cross-leaf:** removes from Wine Cellar; adjusts settlement
**Analytics:** no COGS (unsold, returned)
**Guardrails:** balanced (return ≠ depletion ≠ variance)
**Function:** return unsold consignment stock to consignor; adjust on-hand & settlement.

---

# Candidate leaves — proposed, not yet placed

- **Approvals inbox** *(cross-cutting)* — unified queue of everything awaiting the current user's approval (recipe corrections, transfers, manual depletions, price changes). Reads from Recipes/Accounting; gated by Team Manager roles.
- **Notifications / alert center** *(cross-cutting)* — aggregates variance alerts, lifecycle anomalies, approval requests, sync failures.
- **Comms hub** *(admin)* — management oversight + messaging, originally specced around staff consumption.
- **Onboarding / data import** *(admin, P6-M2)* — SKU/vendor bulk import; the adoption bottleneck for this category.
- **System / utility** — Auth/login, Profile, Settings. Deferred to production (auth explicitly out of scope this phase).

# Optional field — proposed, not yet added

- **Events emitted / consumed** — domain events each leaf fires or listens for. High value for the job-runner and integration architecture; premature until the data model exists.
