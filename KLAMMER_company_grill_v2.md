# KLAMMER — Grill v2: the hardened thesis, re-grilled

Second adversarial pass over `KLAMMER_facade_ai_company.md` — the version that already survived
grill v1. Method: one orchestrator model set the attack plan and wrote every verdict; three
subordinate passes gathered evidence independently — an internal claim-by-claim evidence audit
against the live repo, a web-verified market/competitor investigation (July 2026), and a hostile
unit-economics re-derivation. Every external claim below carries a source or an explicit
"unverified" flag; every internal contradiction carries file:line citations. Interactive dossier:
https://claude.ai/code/artifact/588f744d-6db7-4a4b-8a93-b8162e072e10

## Verdict up front

**Grill v1 produced a better story, not a better plan.** The hardened thesis fixed the three
original killers by introducing five new ones: it inventories software that does not exist, runs
on two irreconcilable clocks, books one-time fees as ARR, sells against a free substitute its own
premise created, and claims a greenfield that has had an incumbent for over a decade. What
remains is real and worth having — but it is a **bootstrappable ~$150–250k/yr domain business
gated on one unverified variable (the founder's warm access to an anchor manufacturer)**, not the
venture-shaped "Stripe for facades" the document describes.

---

## Step 1 — Evidence audit: the thesis vs. its own repository

Of 28 material claims audited, **1 is supported, 7 are contradicted, 20 are unsupported** by
anything in the repository.

### The architecture section describes software that does not exist

`KLAMMER_facade_ai_company.md:36` lists, in one sentence and one breath, "existing
`BracketLayout`/`GuideLayout`/`PanelLayout`/`FacadeTakeoff` + NormEngine/ClashEngine/
SystemConstraintValidator". Grep-verified reality:

| Named component | Exists in repo? |
|---|---|
| `BracketLayout`, `GuideLayout`, `PanelLayout`, `FacadeTakeoff` | Yes — Tier-0 versions, 723 LOC of Core |
| `NormEngine`, `ClashEngine`, `SystemConstraintValidator` | **No — zero references outside that sentence** |
| `*.fsys.json` ("the moat", L2) | **No such file or schema anywhere** |
| `JobSpec`, `ILlmClient`, `ModelSnapshot`, `PlacementPlan` | **No** |
| Run-ledger, golden-project evals, property tests (L1, "the trust product") | **No** — tests are 32 plain xUnit facts, 467 LOC |
| `TenderPilot`, `NormaGPT`, `ModelAudit`, `FamilyForge` | **Named once each, defined nowhere** |

The five-layer architecture reads as inventory but is a wishlist. Total actual source: 1,574 LOC
across two commits — a wholesale migration snapshot dated the same day as this grill, with no
development history behind any "already built" claim.

### Seven internal contradictions (the sharpest four)

1. **The Core paradox.** Line 12: "the clean Core/adapter split you already built *is* the
   company." `KLAMMER_ENGINE.md:21-24`, same repo: "Every algorithm written against the bare
   rectangle is rework. This is not a refactor; it is a restart of the Core's data contracts."
   Both cannot be true. The engine document is the honest one: `Substrate` holds two doubles,
   and `PlanarFaceSubstrate.ToSubstrate()` still collapses a face to its UV bounding box — the
   engine doc's own "#1 correctness debt," confirmed at `PlanarFaceSubstrate.cs:23`.
2. **Two clocks.** The company doc's M0–3 milestone is "ship deterministic one-click first."
   `docs/ROADMAP.md` shows the Tier-0 one-click already shipped (steps 1–8 complete) — and the
   engine doc schedules the work needed to handle *any real building* (openings, i.e. Tier 1, a
   data-contract restart) as its entire Phase 1, realistically 6–9 solo months. The M0–3 pilot
   window silently absorbs a restart its sibling document says cannot fit there.
3. **Pricing contradicts itself.** Line 30 cites "₽50k/seat" as the baseline being escaped;
   line 44 prices the seat at ₽240–360k/yr. Never reconciled.
4. **"3D is a byproduct" is inverted.** Today 3D geometry is the *only* substantive output.
   `TakeoffReport.cs` contains counts, lengths and areas — no price, no norm reference, no
   provenance, no passport fields — and is displayed via `TaskDialog.Show`
   (`ShowTakeoffCommand.cs:47`). The claimed primary product (a priced, citable document) exists
   in no form; the claimed byproduct is the whole product.

Also: line 29 claims output is already "deterministic and auditable" against "executable
constraints" — no constraint validation exists beyond positive-number argument guards, and both
engine documents list norm validation among the *unsolved existential problems*.

**Every business number in the document is underived** — TAM $0.7–1.3M, 150–400 seats, all four
price points, $500k/$2M ARR, NRR 140–160%, "flat-wall 80%", "10–20%/yr" decline, 20–30 firms.
The engine docs at least flag their numbers as "orientations, not measurements"; the business
numbers carry no such caveat.

## Step 2 — The three "things that must be true," stress-tested

**MBT-1: "The paid deliverable is the priced BOM / calculation passport, generated from LOD-200
input."** Three cracks. (a) *"Priced" is someone else's data* — prices live in the manufacturer's
ERP, negotiated and discount-laden; printing list prices is either margin exposure (manufacturer
hostility) or fiction. The realistic deliverable is a *quantified* BOM; pricing is a join the
buyer already performs. (b) *"Legally defensible" overclaims* — a calculation passport has
standing only under a licensed engineer's signature inside project documentation; determinism
plus citations buys auditability, not legal weight. (c) *LOD-200 is still optimistic* — the
thesis's own workflow killer (line 24) says the market runs on Excel/AutoCAD/PDF. The wedge
buyer, a manufacturer's technical department, receives tender packages as DWG/PDF, not as Revit
surfaces. **The "invert the input" pivot relocated the dirty-intake problem; it did not solve
it.** Survives only as: quantified BOM from whatever the tender package actually contains — a
harder intake and a weaker pricing story than written.

**MBT-2: "The Core is genuinely portable to Renga/nanoCAD/IFC in months."** True but vacuous.
The netstandard2.0 math library was never the risk; the engine docs' own #1 existential problem
— substrate extraction from messy models — lives in the adapter, per platform, and the F1–F10
dirty-model catalog is 100% Revit pathology. The associativity moat (D1) is built on Revit
UniqueId + Extensible Storage, with no proven Renga equivalent; and IFC is effectively one-way —
you can compute from it, but you cannot materialize an editable, associative model back into the
authoring tool, so the IFC path silently delivers drawings + BOM only and forfeits D1. Honest
restatement: *the cheap 20% ports in months; the differentiating 80% is per-platform quarters.*
The test as written would return a false positive.

**MBT-3: "Reach 20–30 paying firms via founder-led per-project sales without a manufacturer's
blessing."** Even if true, it proves willingness-to-pay, not a company: 25 firms × ~4
projects/yr × ₽25k ≈ ₽2.5M/yr (~$27k). And the test is expensive — roughly six months of the
same founder who must simultaneously execute the Core restart — while contradicting the strategy
it belongs to (anchor-manufacturer-first). Right instinct, wrong bar, unpriced cost.

## Step 3 — Market reality (web-verified, July 2026)

| Thesis claim | Finding | Confidence |
|---|---|---|
| Manufacturers do layout free as pre-sales, in Excel/AutoCAD | **Holds.** Free/refunded design tied to supply contracts documented (rontmetall.ru, alpicagroup.ru; rusnvf.ru calculator "for tender shortlisting"). Same automation pain discussed on dwg.ru forums in 2010 and 2020 — never solved by BIM adoption. Turnaround "2–3 wks": not found. | MED |
| Manufacturers will pay ₽1–5M/yr | **Unverifiable.** They demonstrably invest *something*: GRADAS commissioned BIM2B for 22 adaptive Revit families + Dynamo auto-placement + auto-specs; SIAL hosts a layout tool for its network; U-kon distributes families via BIMLIB. No public price point anywhere near ₽1–5M/yr; several manufacturers solve it with headcount instead. | LOW-MED |
| Market context | Ventilated facades ≈ 30M m²/yr, ≈ ₽120bn (2025), >90% domestic supply, U-kon + Kraspan ≈ ⅓ of market. But growth ~1% in 2025 and a **forecast decline up to 10% in 2026** (Q1 −15%). The "why now" market is shrinking, not opening. | MED |
| Revit pool shrinks 10–20%/yr | **Not found anywhere — invented precision.** Direction is real (Autodesk halted RU operations, licenses blocked March 2024; domestic-software deadline for state-funded orgs moved to Jan 2028; Renga claims >1,000 orgs). But the one adoption snapshot found says 10% switched/switching, 10% planning, **80% not considering** — and Revit is still described as the dominant practical tool. | LOW on magnitude |
| "No incumbent able/allowed to serve the facade layer" | **Broken — the strongest finding of the grill.** *Kadet-Ventfasad* (cad-cadet.ru, AutoCAD-based, 10+ years on market, distributed via manufacturer SIAL): automatic panel/cassette layout on arbitrary facades **with openings**, specifications, guide cutting tables, cost calculation, cassette unfolds, Excel export, multiple manufacturers' systems. Plus the GRADAS/BIM2B case proves the Revit-native version has been built once, manufacturer-funded. Vendor web calculators (~10 found) fill the shallow end. | HIGH |
| ₽10–40k/project pricing | **Tight, not obviously attractive.** Market rates for *full* design documentation: 50–150 ₽/m², ~₽15–45k for a minimum-size (~300 m²) object; flat ₽60k packages under 350 m²; one vendor charges 30 ₽/m² refunded on purchase. An auto-generated layout priced at what full-service engineering already costs is not a price wedge — the differentiator must be speed and consistency. | MED |

Consequence: KLAMMER's honest differentiation is **not** "first to automate facade layout." It is
"the BIM-native, multi-manufacturer, associative, norm-cited successor to a dated AutoCAD-era
incumbent" — a real position, but one that must be argued against a tool that already ships
openings, BOMs and cutting tables, in a market that gives the service away and is contracting.

## Step 4 — Unit economics, re-derived hostile

At ₽/$ = 95, the stated mix ("5 manufacturer deals + ~250 per-project accounts = $500k ARR"):

- **Read literally** (250 accounts = 250 one-time jobs): worst $75k, midpoint **$224k**, best
  $389k. The best case — top pricing *and* favorable FX — misses the claim by 22%.
- The claim closes only with a **smuggled assumption**: every account buys ~5.2 projects/yr
  (₽130k/account/yr), with every other variable simultaneously at midpoint.
- **ARR quality:** at midpoint, $158k (manufacturer contracts) is genuine recurring revenue;
  $342k is transactional fees that must be re-won every year. **~68% of the claimed "ARR" is
  not ARR.** $500k of true ARR needs ~16 manufacturer deals, not 5.
- **Sales capacity:** a solo technical founder closing ₽1–5M deals into conservative RU
  industrial buyers (3–9 month cycles; tender procurement an M0 legal entity often cannot even
  enter) while building the product: realistic 2–4 closes in 18–24 months. The 250 SMB accounts
  imply ~14 paying signups/month with zero marketing on a high-touch product; realistic organic
  is 2–5/month → 40–90 accounts. The only motion that yields 250 is the manufacturer pushing
  its network — the exact channel the next point poisons.
- **Structural channel conflict:** the thesis's own 10× insight — manufacturers give engineering
  away free to win supply contracts — is the free substitute that caps subcontractors'
  willingness to pay ₽10–40k. 68% of the revenue model competes with the free channel the other
  32% funds. This is not a tuning problem; it is baked into the premise.
- **The seat is a discount:** upgrade trigger ~20 runs × ₽25k = ₽500k > ₽360k top seat price —
  the most engaged customers *reduce* spend at peak usage. Negative expansion by design, which
  alone makes **NRR 140–160% implausible** (realistic blended: ~100–115%).
- **What genuinely works:** the burn. ~₽12M (~$126k) covers founder + one facade engineer +
  inference for 18 months; net cash need $80–150k after pilot revenue. A real $500k top line is
  cash-flow positive for a 3–4 person shop with no venture money. But the $2M/NRR-140% second
  leg requires growth capital the sanctioned, high-rate RU 2026 environment does not supply —
  the plan is bootstrappable at $500k and capital-dependent at $2M, in a market with no capital.
- **The pilot is a network bet:** a ₽1–3M paid pilot from a *cold* anchor manufacturer inside
  one quarter has a base rate under 10%; warm, ~30–50%. The dossier never states whether the
  founder has that relationship. **The entire plan inherits this single point of failure.**

## Step 5 — Moat and timeline realism

- **`*.fsys.json` is not a moat; it is a file format.** A constraints-JSON schema is
  reproducible by any competitor with one facade engineer and a few weeks; manufacturers'
  system albums are public PDFs. The doc conflates the schema (copyable) with the corpus of
  verified encodings + edge-case ledger + manufacturer trust (compounding, but a function of
  years in market — at t=0 the moat is zero). The "two-sided lock" needs architects speccing
  through the free front-end, which ships M10–12: no lock exists during the entire plan.
- **The 18-month arc is refuted by its own annex.** It stacks a Core data-contract restart,
  Tier-1, three encoded systems, an IFC front-end, ModelAudit, a free architect front-end,
  FamilyForge, a public benchmark, and geographic expansion on 1.5–2 people — while the engine
  document in the same repo calls the full matrix "a decade of edge cases" and budgets its own
  Phase 1 alone at most of a year. Where the two documents disagree, the engine document is
  grounded in the code; the company document is grounded in the pitch.
- **The on-prem local-model SKU is asserted, not designed.** "Viable because geometry is
  deterministic" covers the part that was never at risk; the LLM's actual job — dirty-model
  triage and ambiguity classification in colloquial Russian engineering language — is the
  judgment-heavy part, and no evidence exists that a 14–32B local model clears it. Zero LLM
  integration code exists to test either SKU.

## Step 6 — Kill-shots, ranked, each with its cheapest disproof

| # | Kill-shot | Severity | Cheapest de-risk experiment |
|---|---|---|---|
| K1 | **Fiction-as-inventory + two clocks.** The pilot sells a product the sibling doc schedules ~9 months out; the architecture section lists nonexistent components as existing. | FATAL to credibility & schedule | Rewrite the doc against the audit table; adopt the engine doc's Phase 1 as the only calendar. Cost: one day. |
| K2 | **Not greenfield.** Kadet-Ventfasad has shipped openings + BOM + cutting tables for a decade; GRADAS proves manufacturers fund Revit automation directly. | HIGH | Buy/demo Kadet-Ventfasad, get its real price and its users' complaints; interview one mid-size manufacturer (Kraspan/Diat/U-kon) on current pre-sales tooling spend. 2 weeks, ~₽50k. |
| K3 | **ARR fiction + channel conflict.** 68% of "$500k ARR" is one-time; the free-engineering premise caps the paying segment. | HIGH | Rebuild the model counting only recurring contracts as ARR; decide *one* primary wallet (manufacturer OR subcontractor) and price the other as its channel. Cost: one day. |
| K4 | **Input reality is DWG/PDF, not LOD-200.** The pivot moved the intake problem to a worse substrate. | HIGH | E1: hand-produce the passport artifact for 3 real tender packages (DWG/PDF), show it to 2 manufacturer technical departments, ask "would you pay, per object?" 2–4 weeks, no code. |
| K5 | **Wallet unverified + market contracting.** ₽1–5M/yr found nowhere; 2026 facade market forecast −10%. | MED-HIGH | Same interviews as K2/E1 — the wallet question is answerable in two conversations. |
| K6 | **Founder-network single point of failure.** Pilot economics only work warm. | MED-HIGH | Write down, honestly, which anchor-manufacturer decision-maker takes the founder's call today. If the list is empty, the M0–3 milestone is fiction and sequencing must change. |
| K7 | **Moat at t=0 ≈ 0; seat pricing produces negative expansion.** | MED | Reprice the seat above ~20-run equivalent value or keep pure metering; treat the moat as earned corpus, not designed schema. |

Additional experiments worth their cost: **E2** — a two-week Renga API spike (extract one wall
face with an opening, place 200 elements, find the UniqueId/storage equivalent) to make MBT-2
falsifiable for the part that matters; **E3** — 30 cold approaches to facade subcontractors
selling manual-behind-the-curtain layout at ₽25k/object (a concierge MVP that tests K3/K4
willingness-to-pay with zero product risk and generates revenue during the Core restart).

## Step 7 — Final verdict

**As written: not fundable, and not because the idea is bad — because the document is not
load-bearing.** Its inventory is partly fictional, its two calendars contradict each other, its
headline metric dissolves under arithmetic, its market section contains an incumbent it does not
mention, and its wedge depends on an unstated personal relationship.

**What survives the grill, and is genuinely valuable:**
- The Core/adapter discipline as an engineering practice (the split is real; the contents need
  the restart the engine doc already specifies).
- The manufacturer-as-buyer instinct — *directionally confirmed*: manufacturers demonstrably
  spend on exactly this class of tooling (GRADAS, SIAL), just at unverified magnitudes.
- Determinism + norm-cited provenance as differentiation against a dated AutoCAD incumbent and
  against any pure-LLM entrant.
- The benchmark-as-category-artifact idea.
- The cost side: this is one of the rare plans that is genuinely bootstrappable ($80–150k net).

**The honest shape of this company:** a bootstrapped, founder-sold engineering-automation
business that reaches ~$150–250k/yr with one anchor manufacturer and a concierge service
funding the Tier-1 engine build — with venture-scale optionality only after CAD-independence is
*demonstrated* (E2) and the wallet is *priced* (K2 interviews), not asserted.

**The single pivot, if one is taken:** re-sequence truth-first. Months 0–9 belong to the engine
doc's Phase 1 (the PlanarRegion restart, openings, one cladding system, stable keys, provenance)
— because every deliverable the business promises is downstream of it. Revenue in that window is
services sold as services: concierge layouts (E3) and a pilot explicitly contracted as a paid
development partnership, not disguised as a product sale. "ARR" counts only manufacturer and
seat contracts. Positioning drops "first" and "Stripe for facades" for the claim the evidence
supports: *the BIM-native, multi-system, associative successor to the tools this market
outgrew.*

---

### Sources (external claims)
Market size/decline: stroi-baza.ru (id=16402), prcs.ru facade-systems analytics, marketing.rbc.ru/articles/16272. Autodesk/Revit status: rbc.ru 22.03.2024. Domestic-software deadline: pro-ability.ru; BIM mandate: realty.rbc.ru. Renga adoption: rengabim.com (vendor-claimed). Free pre-sales workflow: rontmetall.ru, alpicagroup.ru, rusnvf.ru. Competitors: cad-cadet.ru + old.sial-group.ru (Kadet-Ventfasad), bim2b.ru + archi.ru/tech/81803 (GRADAS), bimlib.pro (U-kon), hilti.com, ejot.com, agacad.com. Design-service pricing: facade.ru, proekt-fasad.ru, ibfm.ru, zavod-fasadov (30 ₽/m² refund model). Most RU industry pages block direct fetch (HTTP 403); findings rest on search-snippet evidence — flagged MED confidence throughout. Not found despite explicit search: manufacturer software budgets, Kadet-Ventfasad pricing, facade-engineer headcounts, any source for "10–20%/yr" Revit attrition.

*Grill v2 executed 2026-07-12. Internal citations reference repo state at commit `23847e7`.*
