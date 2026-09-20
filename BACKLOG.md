# Backlog

Open requests, **grouped into feature clusters and roughly ranked** (a cluster is
a set of small requests that ship together as one `FEATURES.md` feature). When an
item ships it moves to `IMPLEMENTED.md` with how it was done; when one is declined
it moves to `REJECTED.md` with why. See the `detronics-app` skill's
`references/backlog.md` for the pipeline. Ranking is a starting point — say the
word to reprioritise.

**Priority order below** (top = do first). The reasoning: fix the data-loss bug
first; then the pricing-model clarity work, because the commercial-share confusion
undermines trust in every number the app shows; then the missing-multiple-parts
project blocker; then the big unifying "step-by-step flow" for the three estimate
surfaces; then inventory/movements and per-roll identity; then analytics, print
settings, nav and the smaller polish. Say the word to re-rank.

## Batch raised 2026-09-13 (bugs first, then features)

**Bugs** — all cleared; see IMPLEMENTED.md (v1.0.52–v1.0.54).

**Features** — all cleared. Compile-email + checklist + big-file guidance shipped v1.0.55;
partial-height failure → real material loss shipped v1.0.59; purge-tower infill investigated and
closed (the tower is already costed as SPACE, not material — the plastic charged is the real purge
volume; v1.0.59 adds a clarifying Expert note). See IMPLEMENTED.md.

## Pricing model: clarity and correctness

- ~~**Commercial-share panel → full-invoice categories that reconcile top-to-bottom**~~
  — DONE (weight→percent commercial-categories model). `commercialAdjustment`
  (`js/pricing.js`) prices each category from its **actual computed base** in
  `categoryBases` (`js/engine.js`: material, machine, electricity, labour, hardware,
  scrap, packaging=real cost, shipping, handling, profit, growth, and marketing/admin/
  R&D/storage from the real general-allowance components). The company sets a
  **percentage per category** (100% as-is, 110% adds 10%, 90% removes 10%); a custom
  category with no source adds a **percentage of the whole order's total**. The
  categories are the invoice's real components, so the panel reconciles to the money
  diagram — the old "packaging 10% notional vs R90 real" complaint is gone.
  (Raised 2026-09-10; display v1.0.4; general allowance v1.0.6; percent model closed this.)

## Project page & project part editor

_Add-parts, model-first layout, colour-by-height and the %-split removal shipped
in v1.0.5. Remaining:_

- ~~**Auto-estimate the colour split on quote → project**~~ — SHIPPED v1.0.60: on
  save-as-project each head's grams are seeded from the estimate grams × the colour
  split, marked `estimated` (a banner + the production gate still ask for the real
  slice); editing a head clears the flag. See IMPLEMENTED.md.
- ~~**Nozzle size affects print time + a nozzle-change operation**~~ — SHIPPED v1.0.61.
  Company setting `nozzle` (off by default). Per-part nozzle in advanced settings on the
  estimate and project. Finding on part (1): a bigger nozzle does NOT print faster at the
  same layer height — it lays wider lines (thicker/stronger walls + more material) and
  *allows* a taller layer, which is the only speed gain (and lowers the finish; aesthetics
  already track layer height in `scores.js`, so no profile changes needed). Part (2): a
  non-default nozzle books a swap-both-ways labour, amortised across the run. Layer taller
  than the nozzle allows is flagged. See IMPLEMENTED.md.

## The three estimate surfaces — one clear step-by-step flow

_Substantially DONE. The client request form (`js/ui/portal.js`) is a numbered,
decision-ordered stepper, and it doubles as the internal-employee form via
`portalConfig(settings, { internal:true })` (priced at cost, no buffer/expedite).
The estimate tool remains the operator's power surface — an ordered, collapsible
sidebar in the same decision order, not a numbered stepper. Details:_

- ~~**Decision-ordered, no-scroll-back flow**~~ — DONE on the client/employee form
  (stepped top-to-bottom). The estimate tool keeps its sidebar in the same order
  (printer/filament → model/parts → order → pricing → export).
- ~~**Internal-employee estimate form**~~ — DONE: the portal's `internal` mode.
- ~~**Auto-email the estimate/request**~~ — DONE v1.0.55 (compile-email on the form).
- ~~**Colour-change-by-height in all surfaces**~~ — on the estimate and project (the
  operator surfaces); deliberately NOT on the client form, which stays minimal
  (see the "client form minimal" principle). Considered done.
- **Auto-minimise sibling sections (estimate tool only, optional)** — the `section()`
  `group` mechanism already collapses siblings; the estimate's sidebar sections are
  not yet grouped. Small, but wants A/B testing before turning on, so left as an
  opt-in refinement. (Raised 2026-09-10.)

## Inventory & stock movements

_Movement signs-by-reason shipped v1.0.11. Remaining:_

- ~~**Orders record movements on completion**~~ — DONE (verified). Recording a print
  (`bookAttempt` → `movementsForRun`, `js/ui/tools/projects.js`) books the filament,
  parts and resin movements as production happens (tagged `production`/`scrap`), and
  `recordCompletion` tops the record up to the whole job on completion — so by close
  the movements are booked, and they already feed the Dashboard via inventory balances.
- **Keep large dropdowns usable (cascading / filtered pickers)** — as the catalogue
  grows, single long dropdowns for material/colour and stock items become unusable
  (endless scrolling). Break them down: pick **material first, then colour** filtered
  to that material; for a stock line pick the **kind** first (filament / hardware),
  then a category (filament by material; hardware by type, e.g. magnets), so each
  dropdown stays short even years out with many rolls and parts. (Raised
  2026-09-10.)
- **Add-by-filter inventory flow with tick boxes** — replace the current per-type
  add buttons with a filter on the on-hand block: choose a type (filament/spools,
  resin, tools, hardware, packaging), see everything of that type on hand, and
  **tick** the ones you want then press one **Add** — same pattern for packaging,
  etc. The spool-label print sheet likewise becomes tick-boxes: tick the rolls to
  export/print. (Raised 2026-09-10.)

## Per-roll filament identity, labels & CSV backfill

_Extends the existing filament cluster; the CSV items tie inventory to real usage._

- **Per-roll filament tracking, labelling and guided roll selection** — track each
  physical roll individually, right through to finished, even when several rolls
  are the same supplier + colour (e.g. five rolls of "SA Filaments White PLA").
  Pieces:
  - _Per-roll identity & human-readable ID_ — replace the long random spool id with
    a **legible code** with logic: e.g. `PLA` + first 4 of the supplier + the colour
    + a sequence (`001`, `002`, …) counting how many of that spool the company has
    had. Show this roll ID on the roll, in the inventory list, and on the label.
  - _Supplier field_ — add a supplier/brand to a spool (materials have
    `manufacturer`; spools have only batch/location). Report consumption and
    remaining **rolled up by colour + supplier** and per individual roll.
  - _Guided selection / roll spanning_ — the app says which roll to load: prefer the
    nearly-empty roll that can still cover the job (`spoolsFor` already sorts
    emptiest-first); if one roll cannot cover the print, name the next roll to
    continue on; when a later smaller print fits what's left on an earlier roll,
    direct back to that roll to use it up. (Raised 2026-09-07; extended 2026-09-10.)
- **Custom filament-roll labels → downloadable PDF (label-printer sizes)** — a
  company setting turns on a custom label per roll, each carrying that roll's
  human-readable id. Choose the label size to match the label printer (common sizes
  + custom w×h). "Print spool labels" generates a **downloadable PDF** at that size;
  allow **tick-selecting** several rolls into **one PDF**. Builds on the existing
  `buildSpoolLabels` sheet. (Raised 2026-09-07.)
- **Colour swatches on filament colours** — SHIPPED v1.0.62 for the main surfaces:
  `colourHex(material)` (`js/materials.js`) maps the colour name → hex (12 common
  names) with an optional per-material `colourHex` override, so swatches show with no
  migration; reusable `swatch()` in `js/ui/dom.js` (`.swatch` in components.css).
  Wired into the Materials catalogue (list + an editable colour picker), the filament
  slot rows, and the material picker. _Remaining: spool labels and a gradient marker
  for multi-colour spools — small follow-ups._ (Raised 2026-09-07.)
- **CSV import → roll-ID matching, one aggregate movement, order type & cost
  impact** — grow the printer-history / usage CSV import so it:
  - _Matches by roll ID_ — instead of picking material + colour, the CSV references
    the **roll ID** that was used and matches the physical roll.
  - _Books one movement per filament_ — a CSV with, say, ten "PLA yellow" lines
    creates **one** aggregate "used in production" movement for the total grams of
    that filament, not ten.
  - _Carries the order type_ — add the client / internal-employee / internal-company
    selection (already in the app) to the CSV import.
  - _Extrapolates cost impact_ — from the printer history (total time + grams per
    filament) and the roll's purchase type, estimate the cost impact on the company
    and add it to the Dashboard stats.
  - _Is the backfill path when the app falls behind_ — if the one person who logs
    prints is out sick while others keep printing, the CSV import is how the missed
    prints (and their stock draw and cost) get entered later; ensure it flows to the
    Dashboard, and document the "what do we do when we've fallen behind" story in the
    How-to. (Raised 2026-09-10.)

## Company analytics & economics

- **Company stats, trends and ROI** — most-used filament (`byMaterial`), most-used
  hardware (`byHardware`, v1.0.20), overall profit/margin, the ROI-vs-investment
  "has the machine paid for itself" view and a **profit-by-month trend** beside
  revenue (v1.0.42) already show on the Dashboard. Remaining: a what-to-buy /
  reinvestment trend, and longer-horizon trend lines. (Raised 2026-09-10; most-used
  + profit + ROI + profit trend shipped.)
- **Inflation on stock and labour** — apply a yearly inflation rate (South Africa
  especially) to both bought-in stock cost and the salary behind the labour rate,
  so older figures don't understate today's cost. (Tool depreciation is already
  handled via resale value.) (Raised 2026-09-10.)
- **Maintenance & fix hours reduce profit** — hours spent on printer maintenance
  and on fixing failed prints are real labour that isn't quoted to any customer, so
  they should reduce overall profit. Provide a way to log those hours (see failure
  logging below). (Raised 2026-09-10.)

## Failure logging

- **Partial print-fail count** — when logging a failed print of a multi-part plate,
  allow "N of M failed" rather than the whole print failing (pause/skip means you
  can salvage the rest). (Raised 2026-09-10.)
- **Log the fix time against a failure** — when logging a fail you state the core
  issue; also let the operator log the **solution** and **how long** the fix/
  problem-solving took, feeding the maintenance-hours-reduce-profit item above.
  (Raised 2026-09-10.) _Note: the broader issue↔solution knowledge base (connect an
  issue to solutions already found, so the next fix takes 10 min not an hour) should
  be its **own separate 3D-printing app**, not part of this cost estimator — see
  "Separate apps" below._

## Print-setting model additions

- **Adaptive layers in the client form** — adaptive layers shipped as a flag factor
  on the estimate and the project (v1.0.17, ~15% time uplift). The simplified client
  form does not expose the advanced print-setting toggles (ironing, fuzzy skin, …)
  at all, so adding just adaptive layers there needs a decision on whether to surface
  the advanced flags on the client form. (Raised 2026-09-10; est/project shipped v1.0.17.)

## Cross-cutting UX

- **Hover-to-source links on cost figures** — on the project cost figures (cost to
  company, part price, final invoice, …), hovering a value that is derived should
  offer a link to the section that breaks it down (e.g. hover "cost to company" →
  jump to its breakdown). Not in Simple; optional in Advanced, definitely in Expert.
  (Raised 2026-09-10.)
- **UX / clarity audit of every function** — scan through each function the app has
  and find where the user's understanding and visualisation of what's happening in
  the company could be made easier; produce a list of **recommendations for the
  owner to approve or reject** before any implementation. (Raised 2026-09-10.)

## Workflow & dashboard

- **Dashboard refresh on close, with a Reopen that logs edits** — a closed project
  can go stale on the Dashboard: close with quantity 1 → Dashboard shows 1; later
  change to 4 → it updates everywhere else but the Dashboard still reads 1 (likely
  the price-lock freezing a closed project's figures while the rest reads the live
  part). Desired: a **Reopen** button that logs the reopen to the event history;
  editing a reopened project is allowed; **closing it again refreshes the
  Dashboard** to the new figures; any edit to a closed/reopened project is captured
  in the event history so nothing is silent. Investigate whether the staleness is a
  bug or the intended price-lock the reopen/re-close cycle should govern. (Raised
  2026-09-07.)

## Communication

- **Plus-addressing on the company email** — route different correspondence through
  sub-addresses on the one inbox (`shop.detronics+sales@…`, `…+feedback@…`), with a
  setting mapping tags to purposes so the right address shows in the right place
  (sales on the quote/portal, feedback on aftercare) — all landing in one mailbox.
  (Raised 2026-09-08.)

## How-to / guide additions

- ~~**"What to do when you've fallen behind" guide**~~ — SHIPPED v1.0.42: a
  "Catch up when prints went un-logged" how-to (record each missed print; CSV
  printer-history import for a whole machine's history) in the How-to tab.
- **Order-flow flowchart in How-to (decision-driven, by section)** — a detailed
  flowchart of the whole process, estimate → quoting → payment → production →
  post-processing → packaging → delivery → aftercare/feedback, laid out in **columns
  per section**, showing how the company's configured options and the client's
  selections route an order — which decision makes it jump from which section to
  which. Specific, not generic: expedite (skip quote/pay → production), order type
  (customer vs internal-employee vs internal-company), packaging vs pickup vs none,
  post-processing routing, plus hold/cancel/reopen off-ramps. Reflects what THIS
  company has switched on, and is shareable with the client. Builds on
  `js/workflow.js` PHASES / `advance` / `clientProgressReport`; a diagram, so
  consider the diagramming approach. (Raised 2026-09-08.)
- **Suppress/reword the "save a backup" reminder while team sync is connected** —
  redundant once sync auto-saves to the shared file; soften or hide it when a sync
  file is connected. (Raised 2026-09-07.)

## Separate apps (not this cost estimator)

- **3D-printing troubleshooting knowledge base** — a catalogue of failure issues
  versus solutions found (with the time each took), so a recurring problem is solved
  from the log in minutes instead of re-diagnosed for an hour. The owner wants this
  as its **own dedicated 3D-printing app**, not bolted onto the cost estimator. The
  *cost* side of it (logging fix/maintenance hours against a project so they reduce
  profit) stays in this app — see "Failure logging" above. (Raised 2026-09-10.)

## Sanity checks to run (validation — user to do)

- **Estimate vs a real sliced part** — slice a real part and compare the app's time
  and material estimate against the slicer's (feeds the calibration loop and the
  estimator assumptions). (Raised 2026-09-08.)
- **Resin used on an NFC tag** — measure the actual grams of resin used to
  coat/embed an NFC tag and record that tag's size, so the resin-coat op's
  grams-per-cm² can be set from real data rather than a guess. (Raised 2026-09-08.)
- **Snapmaker print-history upload** — upload all the Snapmaker prints (Settings →
  Backup & restore → Printer history import) to compare the app's filament-usage
  estimate against actual; prompted by running out of white PLA on a roll before its
  tracked amount said it should. Validates the grams estimate and roll tracking.
  (Raised 2026-09-08.)
