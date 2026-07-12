# KLAMMER Engine — Grill v2 (combined, corrected)

Single engine document. Supersedes and replaces `KLAMMER_ENGINE.md` and
`KLAMMER_ENGINE_best_in_world.md` (both deleted; history in git). Method: union of the two
v1 engine grills (Fable 5 capability ladder · Opus geometry red-team · Opus engine
architecture · Sonnet competitive benchmark · 1-click intake spec), then corrected against
the findings of `KLAMMER_company_grill_v2.md` — the adversarial audit that broke several
claims the v1 engine docs relied on. Where v1 and the audit disagree, the audit wins.
Corrections are marked **[v2]** and collected in the delta log at the end.

Goal unchanged: **best in the world at automated Revit modeling of ventilated facades (НВФ)
across all cladding types and all levels of building-architecture complexity** — with the
claim allowed to grow only cell-by-cell on a public benchmark, never ahead of evidence.

---

## 0. Verdict on today's engine

The current Core solves **Tier 0** cleanly — and Tier 0 is the one case that never occurs in
practice: a blank rectangular wall with no openings, no corners, no neighbors.

The lie is in the type system: `Substrate` (src/RevitNvf.Core/Model/Substrate.cs) holds
exactly two doubles. `PlanarFaceSubstrate.ToSubstrate()` collapses a Revit face to its **UV
bounding box** — losing the real outline (gables, cut corners), all window/door holes, and
world orientation (face UV is not guaranteed gravity-aligned; the bbox origin may not even
lie on the face). `BracketLayout`/`PanelLayout` are Cartesian products that cannot express a
hole.

**Every algorithm written against the bare rectangle is rework.** This is not a refactor; it
is a restart of the Core's data contracts — keeping the two things that are genuinely right:
the pure-deterministic-Core / thin-adapter split, and the idempotence discipline. Those are
the correct bones; the flesh must be rebuilt on a polygon-with-holes substrate in a
world-anchored frame.

**[v2] This document is the only engine calendar.** The company grill's K1 kill-shot found
the company doc selling in M0–3 a product this document schedules 6–9 solo months out. The
resolution adopted there ("re-sequence truth-first") makes Phase 1 below the master schedule:
every business deliverable is downstream of it, and no sibling document may promise anything
earlier.

## 1. Current state (Tier 0), precisely

- `Substrate(WidthMm, HeightMm)` from a `PlanarFace` UV bounding box — ignores the real
  outline **and** all holes.
- `BracketLayout` uniform grid; `GuideLayout` one vertical per bracket column; `PanelLayout`
  rect grid + joint + start mode + optional edge fillers; `LayerStackup` pie offsets;
  shared `GridAxis` kernel.
- `FacadeSystem` = bracket/guide steps only; `Cladding` = panel W×H + joint + start mode +
  edge-panel flag; `FacadePreset` names a triple.
- `TakeoffReport` = counts, lengths, areas — no SKUs, no provenance, no norm references;
  displayed via `TaskDialog.Show`.
- ~1.6k LOC total, 32 plain xUnit facts. No openings, corners, curves, inclination,
  multi-face, thermal, wind zoning, cladding-type specifics, stable keys, or validation
  beyond positive-number argument guards.

## 2. The complexity ladder (what "all complexity" decomposes into)

| Tier | Scope | New problems that must be solved |
|---|---|---|
| **T0** | Flat rectangle | DONE — axis subdivision, start modes, edge fillers, takeoff |
| **T1** | True boundary + openings | Polygon-with-holes region; anchor exclusion zones + edge-distance minima (~100 mm); jamb/lintel bracket densification; guide splitting at openings; panel clipping; **reveal generation (откос/отлив/lintel) with depth derived from LayerStackup**; joint-to-jamb snapping. *Where every Dynamo script dies. Minimum bar for a real building.* |
| **T2** | Multi-face + corners | Face-adjacency graph with dihedral classification; **building-global datums** (courses align across faces — per-face solving is conceptually wrong from here); corner strategies (wrap / miter / L-element / trim, per cladding); corner bracket doubling + thermal gap; corner SKUs with unfold |
| **T3** | Full-building engineering | Parapet caps, цоколь, deformation joints; **thermal fixed/sliding runs** (exactly one несущий per segment, splice gaps, max run length); **wind-zone densification as parameter fields** (corner strips ≈ b/10, parapet bands, height bands — zone maps as data, no verification calc); fire рассечки/окантовка (АКП); weak-substrate mode (газобетон → attachment at slab lines = a layout-topology switch). *Converts output from "a picture" to рабочая документация — pricing power lives here.* |
| **T4** | Inclined / faceted / cylindrical / soffits | Developed-surface panelization (cylinder unrolls exactly); chord-tolerance faceting (`δ = c²/8R ≤ δ_allow`); gravity ≠ face-up (fixed/sliding and admissibility change); engine must **refuse correctly** where systems aren't certified overhead |
| **T5** | Free-form / double-curved | Mesh planarization, per-panel fabrication, scan-to-BIM absorption. **A different product generation — deliberately not promised.** |

## 3. The three existential problems (red-team consensus)

1. **Hole-aware, world-oriented, multi-face substrate extraction** from messy real models
   (joined wall segments, sweeps, masses, curtain systems, links). Everything is downstream
   of this, and it has not been started. `Substrate = W×H` actively prevents it.
2. **Editable associativity via diff-and-reconcile.** "Regenerate from scratch" fails in
   practice: every ElementId churns → every tag/dimension/override detaches → schedules diff
   as delete-all/add-all → one window move = full facade re-issue. This is what separates a
   *modeler* from a one-shot screenshot generator, and it **cannot be retrofitted** — element
   identity must be designed into Core output contracts now.
3. **Engineering-correctness validation that fails loudly.** Plausible-but-wrong output
   passes visual review and fails экспертиза/site: anchors on mortar joints, zero
   fixed/sliding logic, 20 mm slivers at corners, uniform grid under-fastening the
   wind-corner strips. Encode СП 426 / ГОСТ Р 58154-55 constraints as a validation layer
   that refuses or flags — never renders silently.

(Corners — coupled-face course alignment — is the close 4th, and is really a facet of #1's
data model.)

## 4. How it breaks — red-team catalog

Ranked failure modes of any naive build:

1. **Substrate lie (HIGH).** A UV bbox invents area over gables, ignores openings, origin
   isn't on a non-rect face, UV basis isn't guaranteed horizontal → tilted/mirrored grids;
   "the wall" is often many segments/masses/curtain/links. → hole-aware polygon-with-holes
   in a **world-anchored frame (gravity-derived, not face UV)** from real EdgeLoops. Fixing
   `PlanarFaceSubstrate.GetBoundingBox()` is the **#1 correctness debt**.
2. **Openings & corners (HIGH).** Hole vs regular grid is incompatible; corners couple faces
   (courses must align turning the corner) → boolean-subtract openings with clearance + cell
   classifier with min-panel rule (never emit slivers); building **graph** with shared corner
   edges; courses on a global datum.
3. **Associativity (HIGH).** See existential problem #2 — stable keys, key→UniqueId storage,
   delta-only apply, override preservation at key granularity (never freeform geometry merge).
4. **Correctness trap (MED, CRITICAL).** See existential problem #3 — a validation layer
   with min panel width, max thermal run, wind-zone density, anchor-into-valid-material.
5. **Revit scale.** 20–60k elements → DirectShape + batching, or the tool dies in demos.
6. **Curved/inclined.** Scope out of v1; refuse correctly rather than guess.
7. **Cladding combinatorics.** Shared kernel + per-family strategies, not flags (see §5.5).
8. **Dirty-model intake.** The catalog below — each entry is detect + repair + question/refuse
   **+ a regression `.rvt` fixture**; an entry without a fixture counts as unimplemented.

**Dirty-model failure catalog:** F1 split walls (auto-merge) · F2 linked model (read-only,
place in host) · F3 void-openings (batched confirm) · F4 non-vertical (≤0.5° project /
0.5–3° ask / >3° refuse) · F5 sweeps/rusts (absorb or ask) · F6 stacked walls (band split) ·
F7 curtain-as-substrate (refuse + junction node) · F8 mass/DirectShape (refuse → audit path) ·
F9 mirrored families (truth = the wall cut) · F10 no level datum (the one un-removable
question).

**[v2] Intake reality check (company grill K4):** this catalog is 100% Revit pathology, and
it assumes the input *is* a Revit model. The wedge market's tender packages arrive as
DWG/PDF. The engine's contract stays geometry-in (`GeometryInput`) — DWG/PDF intake is
explicitly **not an engine problem**; it is handled commercially (concierge modeling of the
substrate, audit-path refusal) until a dedicated intake front-end earns its place. The v1
docs' silent assumption that LOD-200 Revit input simply exists was optimism; naming the gap
is the fix, not stretching the engine to swallow PDF.

## 5. Engine architecture (the rebuild)

### 5.1 Substrate model — three layers, all pure Core types
- **`PlanarRegion`** = outer `Ring` + hole `Ring`s + `FaceFrame` (world origin +
  gravity-aligned U/V + normal). Built by the adapter from **real `EdgeLoops`** (never
  `GetBoundingBox`), coplanar neighbor faces unioned first. Coordinates snapped to a fixed
  grid (0.1 µm) for exact, deterministic boolean ops; one shared `Tol` policy.
- **`BuildingEnvelope`** = faces + `CornerEdge` half-edge adjacency (dihedral → ExternalL /
  Internal / Coplanar / Parapet / Plinth / Expansion) + `LevelBand` shared datums (the
  coordination backbone: courses, splices, wind bands all reference building levels so
  joints align across faces).
- **`ISurface`** abstraction: Planar / Cylindrical / Conical evaluate as analytic pure
  structs (developable → unroll, panelize in developed 2D unchanged); freeform arrives
  pre-faceted by the adapter under an explicit
  `FacetingPolicy { MaxChordMm, MaxWarpDeg, FlatFacet|ColdBend|Reject }`.

### 5.2 Pipeline — seven pure stages behind immutable DTOs
```
GeometryInput → [1] SubstrateExtraction → [2] RegionModel → [3] SubstructureLayout
→ [4] Panelization → [5] CornerOpeningResolve → [6] ThermalWindAssign → [7] Takeoff
→ LayoutResult
```
Each stage independently golden-tested and swappable. `GeometryInput` (faces/edges/levels/
system spec) in; `LayoutResult` (brackets, guide segments with splices+fixed/sliding, panel
cells with footprint rings + SKUs, trims, BOM) out. No Revit types anywhere inside. The
adapter is a geometry courier + diff materializer.

### 5.3 Opening-aware panelization
Grid per strategy (start modes generalized to `LayoutDatum { Anchor, BondPattern,
CourseSource }` — running bond = per-row phase; courses-from-levels hits floor lines) →
polygon-clip each nominal cell against outline+holes → classify `Full / CutByOpening /
CutByEdge / Reveal / CornerFiller / Sliver` → **never emit a Sliver** (absorb into neighbor
or convert to filler via `ResolutionPolicy` with `minPanelFraction`) → generate reveal
assemblies per hole. SKU assignment v1 = deterministic quantize-and-count (5 mm buckets),
histogram in takeoff; global optimization deferred as a swappable stage-7 consumer.

### 5.4 Substructure with wind/thermal correctness
`SubstructureSpec` carries `WindZone[]` as **declarative predicates** (CornerStrip(w) /
ParapetBand(h) / HeightAbove(z) / OpeningMargin(d)) evaluated to masks from the envelope
graph — densify inside masks, snap brackets to guide columns, subtract opening margins.
Guides segmented by max run / floor lines; **exactly one fixed bracket per segment**, rest
sliding, splice gaps emitted — an assignment solver with invariants, directly BOM-countable.

### 5.5 Cladding-system-as-data — the polymorphism boundary
One engine, strategy interfaces resolved from a data-driven `SystemDefinition`:
`IPanelizationStrategy` (module authority; grid = list of courses), `ICornerStrategy`
(open-joint profile / bent-SKU-with-unfold / miter / vendor extrusion), `IAttachmentStrategy`
(klammer rows shared between courses / cassette interlock imposing install order / rivet
patterns with one fixed hole; also yields `PreferredGuideStep`). ~90% of new systems = a
data row over shared strategies.

Four axes fracture any "one grid fits all" abstraction, and the schema must express all four:
1. **Module authority:** catalog-fixed (keramogranit, terracotta) vs made-to-order (cassette,
   АКП) → the solver runs in two modes (fit fixed modules vs derive from wall).
2. **Joint authority:** parametric (keramogranit, fibrocement) vs tooling-fixed (cassette
   folds, terracotta profiles).
3. **Attachment is relational:** рядовой klammer belongs to two courses; interlocking
   cassettes impose install order; fibrocement imposes hole patterns with one fixed point →
   output needs attachment *entities* linking panels↔guides, not per-panel flags.
4. **Cut policy** (free / factory-only / refabricate) — changes what the optimizer even
   minimizes.

**Acid test:** linear/planken has **no vertical grid**. If the panelization strategy
expresses "courses only" without hacking the rectangular panelizer, the abstraction is right.

**[v2]** The v1 docs disagreed on when the second system lands (Phase 1 vs Phase 2). Resolved:
implementation of metal cassette moves to Phase 2 (Phase 1 stays small — the company grill
showed the 18-month arc was overstacked), but the `SystemDefinition` schema is **designed and
paper-validated against both module modes and the planken acid test in Phase 1**, so the
klammer implementation cannot bake in catalog-fixed assumptions.

### 5.6 Associativity — the biggest moat (D1)
- **Stable keys relative to datums**: `PanelKey { FaceId, CourseIndex, PositionIndex,
  SystemId }` — moving a window changes keys only for touched cells; every untouched panel
  keeps its key and its Revit element.
- **`MaterializationRecord` in Extensible Storage**: domain key → `UniqueId` (never
  ElementId) + `OverrideFlags` + `InputHash` (identical input ⇒ true no-op).
- **Diff-apply in one TransactionGroup**: Added/Removed/Changed/Unchanged; unchanged elements
  are never touched; manual overrides tracked at **key granularity** (replace/keep — no
  freeform geometry 3-way merge, ever) and conflicts flagged, not clobbered.
- **Never touch the architect's elements** — an invariant, not a setting.

### 5.7 Revit materialization at scale (20–60k elements)
- Cut/non-rect panels → **DirectShape** (solids pre-built outside the transaction); repeated
  rectangular panels → parametric FamilyInstance; curved → adaptive components;
  brackets/guides → point-placed FamilyInstances. **Own the placement — no face-hosting**
  (hosting fights the diff engine and shatters on host edits).
- Batch everything; **zero `Regenerate` in loops** — one regen at commit; symbols activated
  once; transactions split per element class under one TransactionGroup; fasteners are
  **counted, never instanced**.
- Shared parameters (`NVF_System, NVF_PanelSku, NVF_BracketType, NVF_FixedOrSliding,
  NVF_WindZone…`) bound at startup → schedules agree with `BillOfMaterials` by construction.

### 5.8 Eval harness (cross-platform, no Revit needed)
- **Golden buildings**: hand-authored `GeometryInput` JSON fixtures spanning tier × cladding;
  canonicalized snapshot approval tests.
- **Determinism gates**: run twice → byte-identical; shuffle input face order → identical.
- **Property invariants** (CsCheck): area conservation (panels+joints+openings == region),
  no overlaps, exactly one fixed point per run, every bracket on a guide, zero slivers.
- **Metamorphic**: move a window ±ε → only local keys change (bounds the associativity blast
  radius); scale face → SKU-count invariant; mirror corner → mirrored resolution.
- Windows-only smoke: materialize a golden into a doc, element counts == BOM, schedule
  totals == takeoff. Provable on Linux CI before Revit ever opens.

## 6. Intake and the honest "1 click"

**1 click = facade or diagnosis. No silent decisions.** Formula: 1 intent + K confirmations
+ 0 surprises, K measured & shown (target ≤5/facade, 1 mandatory = the layout datum). The
first click always yields a completed artifact — a full draft facade + BOM + паспорт, OR an
audit with площади, per-m² estimate, and a concrete fix recipe. Inside a compiler, outside a
review (pins in 3D closed by a click).

**Intake agent — LLM translates, Core measures.** Hard rule: the LLM never emits a
coordinate, dimension, or count. It classifies ambiguity, groups scope, prioritizes
questions, and phrases in the engineer's language. Confidence = **provenance tier** (green
auto / yellow assume / red refuse), not LLM self-rating. Question budget harness-forced;
over-budget → yellow assumption logged in the паспорт, never silent.

**Trust architecture:** provenance on every element (ruleId + version + input face + params
+ normRef + assumption ancestors + input hash); паспорт расчёта byte-reproducible
("пересчитайте — получите то же"); verify 20k elements in 15 min via 4 lenses (assumptions
list, violations panel ≤5 groups, statistical fingerprint histograms, guided sample walk).

**[v2] LLM honesty (company grill, Step 5):** zero LLM integration code exists today, and no
evidence exists that a local 14–32B model clears colloquial-Russian dirty-model triage — so
the on-prem SKU is an *aspiration*, not a design. Consequence for sequencing: **the
deterministic path ships first and must be complete without any LLM** (defaults + explicit
dialogs). The intake agent is an ergonomics layer added after Phase 1, never a dependency of
it. The F1–F10 catalog is implemented as deterministic detectors either way — the LLM only
phrases the questions.

**[v2] Deliverable honesty (company grill, MBT-1):** the BOM is **quantified, not priced** —
prices live in the manufacturer's ERP, negotiated and discount-laden; printing list prices is
margin exposure or fiction. Price-book joins are the buyer's (or a per-deal integration), not
an engine claim. And the паспорт расчёта earns **auditability, not legal weight** — legal
standing exists only under a licensed engineer's signature inside project documentation. Both
corrections propagate to every artifact this engine emits.

## 7. The 10× differentiators (ranked)

| # | Differentiator | Why make-or-break |
|---|---|---|
| **D1** | Identity-stable associative recompute | Everyone can generate; almost no one can *re*-generate into a documented model. Existential. |
| **D2** | Corner + opening intelligence (узлы, not field) | ~80% of facade labor is details — откосы, отливы, corner cassettes, parapets, fire окантовка. Where customers decide the tool is real. Existential. |
| **D3** | Parameters-as-fields (wind/height/zone densification) | Real drawings never have one bracket step; absence = un-submittable above ~10 m. Wins deals. |
| **D4** | SKU minimization + cutting stock | A number to sell: "waste 12% → 6%." Wins deals. |
| **D5** | Thermal/movement constraint solver | One violated rule and the facade oil-cans in the first winter. Wins deals. |
| **D6** | Explainable, norm-annotated output | Per-element provenance + СП/ГОСТ pass/fail audit → converts экспертиза into a sales channel. Wins institutions. |
| **D7** | Tolerance-driven curved panelization | Known math — narrow moat alone; unbeatable combined with D1+D4. Wins landmark projects. |

D1, D2, D6 are the durable moats — deep model architecture, not copyable algorithms.
D1 and D6 are **architectural invariants built in Phase 1**, not features added later.

## 8. Competitive reality **[v2 — rewritten]**

The v1 engine docs claimed a "white space nobody occupies" and closed with "nobody else does
it." The company grill's strongest finding broke that: **the market has had an incumbent for
over a decade.**

- **Kadet-Ventfasad** (cad-cadet.ru; AutoCAD-based; distributed via manufacturer SIAL):
  automatic panel/cassette layout on arbitrary facades **with openings**, specifications,
  guide cutting tables, cost calculation, cassette unfolds, Excel export, multiple
  manufacturers' systems. 10+ years shipping.
- **GRADAS / BIM2B**: a manufacturer directly commissioned 22 adaptive Revit families +
  Dynamo auto-placement + auto-specs — proof that the Revit-native version has been built at
  least once, manufacturer-funded.
- ~10 vendor web calculators fill the shallow end; Hilti/EJOT/AGACAD occupy adjacent niches.

**The honest claim** is therefore not "first to automate facade layout." It is: **the
BIM-native, multi-manufacturer, associative, norm-cited successor to a dated AutoCAD-era
incumbent.** What the incumbent demonstrably lacks — and where the engine's bar is set:
BIM-native associative recompute into a documented model (D1), norm-annotated provenance
(D6), building-global datums and corner/узлы intelligence beyond flat AutoCAD layouts (D2),
declarative multi-system schema rather than per-vendor hardcoding, and operability inside
the authoring tool. Each of these must be *demonstrated against Kadet-Ventfasad output*, not
asserted. The de-risk experiment stands: buy/demo the incumbent, learn its real price and its
users' complaints (~2 weeks, ~₽50k).

The structural white space survives in narrower form: norm-correct ПОК + all-cladding + high
geometric complexity + auto-BOM + associativity + **operable by a normal facade engineer,
not a computational-design expert** — inside the BIM authoring tool. Incentives still keep
the giants out: Autodesk monetizes the platform, hardware vendors monetize their systems,
consultants monetize the expertise the tool would erase. But entry is contested, and the
market backdrop is a forecast **decline** (~−10% in 2026), not a greenfield opening.

## 9. Portability **[v2 — restated per MBT-2]**

The v1 claim "the Core is genuinely portable to Renga/nanoCAD/IFC in months" is true but
vacuous: the netstandard2.0 math library was never the risk. Honest restatement:

- **The cheap 20% ports in months** — the pure pipeline, strategies, validation, BOM.
- **The differentiating 80% is per-platform quarters** — substrate extraction from messy
  models (existential problem #1) lives in the adapter, per platform; the F1–F10 catalog is
  100% Revit pathology and each platform grows its own; D1 is built on Revit UniqueId +
  Extensible Storage with **no proven Renga equivalent**.
- **IFC is effectively one-way**: you can compute from it, but you cannot materialize an
  editable, associative model back into the authoring tool — the IFC path delivers
  drawings + BOM only and **forfeits D1**. Say so wherever the IFC front-end is promised.
- **De-risk (E2), scheduled in Phase 1:** a two-week Renga API spike — extract one wall face
  with an opening, place 200 elements, find the UniqueId/Extensible-Storage equivalent.
  Until E2 passes, no document may claim CAD-independence as fact.

## 10. The benchmark — making "best in the world" measurable

Public **NVF Facade Benchmark**: 5 tiers × 8 claddings = **40 cells**, each a downloadable
reference building (RVT/IFC) + golden solution — an MLPerf/SWE-bench for facade modeling.
Score published per-cell (0 = unsupported / 1 = passes with violations / 2 = full pass) —
aggregates can't hide weak cells, and gaps are visible by design so expanding claims never
requires hiding anything.

| Metric | World-class target |
|---|---|
| Norm-compliance (parameter constraints) | 100% elements in допуск (0 violations / ≥500 elements) |
| Clash-free (pie layers, ПОК, openings) | 0 clashes on reference buildings |
| Min-panel-width discipline | ≥98% panels ≥ min; remainder = flagged доборные only |
| Waste % / unique-SKU ratio | ≤5–8% waste T0–T2 (≤12% T3+); SKUs ≤15% of panel count |
| Recompute after an edit | <5 s @ 500 m² facade; <30 s full building |
| Manual-touch rate | ≤1 fix/100 m² (T0–T1), ≤3/100 m² (T2–T3) |
| Full generation @ 5,000 m² | <3 min incl. Revit materialization |
| BOM accuracy vs golden | ≥99% by count/type |

*Targets are engineering orientations to calibrate on the first real benchmark run, not
measurements of the current Core.*

**Proof motion [v2 — incumbent added]:** side-by-side runs on the same building — this
engine vs Revit-native curtain system by an expert vs a Dynamo expert with
LunchBox/PanelingTools **vs Kadet-Ventfasad where its scope allows** — publishing
time-to-model+BOM, manual edits, and post-hoc norm compliance, with the raw files
downloadable. First publisher defines the terms of comparison; competitors either play by
the rubric (legitimizing it) or stay unmeasured. Benchmarking against the incumbent is now
mandatory: a benchmark that omits the decade-old shipping tool would be read — correctly —
as hiding from it.

## 11. Conquest sequence **[v2 — re-anchored as the master calendar]**

**Phase 1 — T1 × керамогранит (кляммер). Realistically 6–9 solo months. The only calendar.**
Replace `Substrate` with `PlanarRegion` **now**; ship opening exclusion + perimeter
densification, guide splitting, panel clipping, reveal generation, joint-to-jamb snapping.
**D1 (stable keys + diff adapter) and D6 (provenance) are built in this phase —
architectural invariants, not features.** `SystemDefinition` schema paper-validated against
cassette (made-to-order) and planken (courses-only) even though only klammer ships. E2 Renga
spike executed. Revenue in this window is services sold as services (concierge layouts, paid
development partnership) per the company grill — never a product sale of what doesn't exist.
Claim on the public benchmark: *best in the world at automated ПОК+cladding modeling for
keramogranit facades with openings, norm-conformant, auto-BOM.* Narrow, verifiable, true.

**Phase 2 — T2 + металлокассета (+ АКП).** Face graph, building frame, global datums, corner
strategies incl. bent-cassette unfold. Cassettes force the schema to be real (made-to-order
module mode, install-order interlock). Fibrocement/HPL ride along nearly free. First SKU
minimization (needs whole-building coordinates). Most direct incumbent competition — worth
the most to win, and the first phase where the benchmark-vs-Kadet motion has teeth.

**Phase 3 — T3.** Zone-map fields (D3), fixed/sliding solver (D5), parapets/цоколь/
деформационные швы, fire окантовка, slab-line attachment mode. This converts users from
"layout tool" to "рабочка tool" — pricing power.

**Phase 4 — T4 + реечные/терракота.** Inclined/soffit admissibility, faceting solver (D7),
1D cutting-stock maturity, vendor-constrained corner SKU logic.

**Deliberately deferred, said out loud:** T5 free-form; natural stone & glass (per-panel
structural liability collides with the no-verification-calc doctrine); FEM/wind *calculation*
(stay parameters-per-СП; partner); scan-to-BIM (keep bracket adjustment range as a data
field so the door stays open); generative/stochastic layout search (determinism is a feature
D1 depends on — don't trade it before the constraint model is complete); auto-selecting the
facade system (pre-fill; the engineer presses the button); DWG/PDF intake (commercial
problem, not an engine stage); face-hosted placement (forever).

## 12. Honesty clauses (each one is load-bearing)

1. **"Norm-compliance" = conformance to the norms' parametric constraints**, never
   engineering load verification. Stated on page 1 of every паспорт.
2. **The BOM is quantified, not priced.** Pricing is the buyer's join. **[v2]**
3. **The паспорт buys auditability, not legal weight** — standing requires a licensed
   engineer's signature. **[v2]**
4. **"All claddings × all complexity" is a decade of edge cases** — the benchmark matrix is
   published with its gaps visible, and the claim grows cell-by-cell, never ahead of the
   evidence.
5. **"Not greenfield."** The pitch names the incumbent and argues the succession; it never
   claims "first." **[v2]**
6. **CAD-independence is a hypothesis until E2 passes.** IFC path = compute-only, no D1. **[v2]**
7. **The LLM is optional.** Every workflow completes deterministically without it; the
   agent only reduces K, never adds capability. **[v2]**
8. **A dirty-model catalog entry without a `.rvt` regression fixture counts as
   unimplemented.**

## 13. Immediate file-level implications

| Today | Becomes |
|---|---|
| `src/RevitNvf.Core/Model/Substrate.cs` (W×H) | `PlanarRegion` + `BuildingEnvelope` + `ISurface` |
| `src/RevitNvf.Revit/Geometry/PlanarFaceSubstrate.cs` (`GetBoundingBox` — the #1 correctness debt) | Real `EdgeLoops` + hole extraction + gravity-aligned `FaceFrame`, coplanar union |
| `src/RevitNvf.Core/Model/FacadeSystem.cs` (4 scalars) | `SystemDefinition` + `SubstructureSpec` (constraints/zones as data) |
| `src/RevitNvf.Core/Layout/PanelLayout.cs` | `AxisCells` survives as the inner strip kernel; top-level API becomes strategy-driven opening-aware `Panelization` |
| `src/RevitNvf.Core/Reporting/FacadeTakeoff.cs` | Per-SKU / per-zone `BillOfMaterials` with provenance |

The executable form of this table is `docs/SPECIFICATION.md` (v2) — the specification is the
contract; this document is the argument for it.

---

## Delta log: v1 → v2

| # | v1 claim (either engine doc) | v2 correction | Source |
|---|---|---|---|
| 1 | "Nobody else does it" / "white space nobody occupies" | Incumbent named (Kadet-Ventfasad, 10+ yrs, openings+BOM+cutting tables); position = BIM-native successor, not first mover | Company grill Step 3 / K2 |
| 2 | Two calendars (company M0–3 pilot vs engine Phase 1 = 6–9 months) | Engine Phase 1 is the only calendar; business deliverables are downstream | K1 |
| 3 | "Priced ведомость" as the deliverable | BOM quantified, not priced; pricing is the buyer's join | MBT-1(a) / K3 |
| 4 | "Legally defensible artifact" | Auditability, not legal weight | MBT-1(b) |
| 5 | LOD-200 Revit input assumed | Tender reality is DWG/PDF; named as a commercial gap, out of engine scope | MBT-1(c) / K4 |
| 6 | "Core portable in months" | 20% ports in months; differentiating 80% is per-platform quarters; IFC forfeits D1; E2 spike gates the claim | MBT-2 |
| 7 | On-prem local-LLM SKU "viable because geometry is deterministic" | No LLM code exists; local-model capability unproven; deterministic path must be LLM-free | Step 5 |
| 8 | Phase 1 scope ambiguity (cassette in or out) | Cassette implementation → Phase 2; schema paper-validated against both module modes in Phase 1 | Overstacking finding, Step 5 |
| 9 | Benchmark proof motion (3-way) | 4-way: incumbent added where scope allows | K2 |
| 10 | Market timing implied "opening" | Market forecast −10% (2026); urgency argument dropped, succession argument kept | Step 3 |

*Grill v2 (engine) assembled 2026-07-12, aligned with `KLAMMER_company_grill_v2.md` and repo
state at commit `941708d`.*
