# PRD Quality Review — AI-Powered Lead Intake Platform for Contractors

## Overall verdict

This is a strong planning-stage PRD: decisions are stated as decisions, trade-offs are named rather than smoothed, assumptions and open questions are tagged and indexed with near-perfect roundtrip, and the Glossary discipline holds across FRs/UJs/SMs. The main risk is that the platform's single biggest blocker — which Trades/Services make up Phase 1 (§8 OQ1) — is still unresolved and gates MVP scope, the verification mechanism, and the AI spike itself; and a handful of FR consequences (FR-7, FR-9) leave numeric thresholds undefined without the `[ASSUMPTION]` discipline applied elsewhere. Neither is fatal — both are fixable in a follow-up pass, not a rewrite.

## Decision-readiness — strong

Decisions are stated as decisions, not hedged as "considerations": the single-match model over lead-auction (§1), the hybrid intake split (§4.1), the Tier/Matching-Priority boundary (§4.8, reinforced by SM-C3), and rejecting full AI auto-assignment (`addendum.md`, "Rejected/Deferred Alternative") are all committed positions with stated rationale, not open musings. `[NOTE FOR PM]` callouts land on genuine tensions — e.g. FR-2's undefined Skip-AI eligibility criteria, and §4.8's warning that the Tier/Matching-Priority boundary "should be preserved as tier specifics get filled in" (a real risk of scope creep toward pay-to-play). Open Questions are actually open: §8 OQ8 ("what specifically makes this platform's analysis 'best-in-class'... This needs an answer before the AI vision-analysis spike... can be scoped") states a real unresolved dependency, not a rhetorical question with the answer already given.

### Findings
- **high** Phase 1 trade/service subset is undecided but load-bearing (§8 OQ1, §6.1) — MVP scope (§6.1 says "Repair/Emergency Work Types" with no Trades named), the verification mechanism (FR-11), the AI vision-analysis spike scope, and go-to-market all wait on this single decision. The PRD is honest about the gap (OQ1 names all four downstream dependents explicitly), but as written no one can start architecture or the spike without it. *Fix:* force a provisional decision now — `addendum.md`'s own directional read (Plumbing, Electrical, HVAC) is a reasonable working default — and let the PRD carry it as `[ASSUMPTION]` rather than leaving it fully open.

## Substance over theater — strong

No findings. Three personas (Homeowner, Contractor, Admin/Dashboard User) each drive distinct requirements — Admin/Dashboard User specifically forks FR-12/FR-13's two named UX failure modes (information overload, click count), not a cosmetic addition. The Vision statement (§1) is not swappable boilerplate — it names the specific trust mechanism (single-match vs. auction) and explicitly confronts competitive erosion of novelty ("AI Diagnosis is not a feature to hedge on — competitors have already entered this space"). Differentiation is reframed honestly in `addendum.md` ("frame differentiation as superior lead-quality/exclusivity... not 'first AI diagnosis' — that claim no longer holds") rather than claiming false novelty. No generic NFR boilerplate ("must be scalable/secure") was found; the two Feature-specific NFRs present (FR-1 latency, §4.6 mobile usability) are product-specific and honestly tagged as lacking numeric targets rather than dressed up as firm commitments.

## Strategic coherence — strong

The thesis is explicit and singular: incumbent marketplaces monetize by flooding contractors with shared, unqualified leads (documented with specifics in `addendum.md` — Angi's BBB "F" rating, FTC's $7.2M HomeAdvisor fine for overstated conversion), and this platform bets on pre-qualified single-match instead. Feature sequencing follows the thesis: MVP keeps the diagnostic core (FR-1/2, FR-5/6, FR-7-10, FR-11-14) and defers monetization depth (FR-16 pricing engine, FR-19 analytics, full Tier detail) rather than front-loading revenue features. Counter-metrics are named and specific to the thesis, not generic: SM-C1 blocks optimizing lead *volume* at the expense of quality (the exact incumbent failure mode), SM-C2 blocks rewarding AI overconfidence, SM-C3 blocks Tier revenue from breaking the qualification-band boundary. This is the opposite of a DAU/MAU vanity-metric tell.

### Findings
- **low** MVP scope kind (problem-solving / experience / platform / revenue) is never named explicitly. The scope logic reads as problem-solving-first (diagnostic core in, monetization depth deferred), which is coherent, but stating it would make the §6.1/§6.2 split legible to a reader who doesn't infer it. *Fix:* one sentence at the top of §6 naming the MVP scope kind.

## Done-ness clarity — adequate

Most FRs carry a "Consequences (testable)" block, which is good discipline and substitutes for a separate Acceptance Criteria section as the rubric allows. Concrete bounds exist where it counts most for UX-critical paths (FR-13: "no more than two taps/clicks"; FR-1: "at least one photo before AI Diagnosis is attempted"). But the PRD's own convention — tag missing numbers as `[ASSUMPTION]` (as done correctly for FR-1's latency and SM-1/SM-7's targets) — is not applied consistently, leaving a couple of consequences silently vague.

### Findings
- **medium** FR-7's testable consequence — "description under a minimum length is flagged as lower-quality" — leaves "minimum length" undefined with no `[ASSUMPTION]` or Open Question tag, unlike the PRD's own pattern elsewhere (e.g. FR-1's latency gap is explicitly tagged and routed to §8 OQ2). *Fix:* either supply a directional number or tag it and add to §8/§9 like the other numeric gaps.
- **medium** FR-9's consequence — "budget significantly below market expectation... is flagged" — has the same untagged vagueness. The Glossary (§3, Budget-Realism Check) tags the *single-check-vs-two-cause* design as an assumption but never tags the threshold itself. *Fix:* same as above — tag or quantify.
- **low** FR-14's "expedited alert (e.g. push notification)" is appropriately tagged `[ASSUMPTION]`, so no action needed, but note it as an example of the pattern done right, for contrast with FR-7/FR-9 above.

## Scope honesty — strong

§5 Non-Goals and §6.2 Out of Scope for MVP do real work rather than gesturing — each exclusion carries a reason (e.g. "Contact-Reveal Event tracking beyond basic logging — deferred richer reporting on this data"; the Budget-Realism cause-split deferral cross-references the full mechanism captured in `addendum.md` for later). The `[ASSUMPTION]` tag / §9 Assumptions Index roundtrip is essentially complete (see Mechanical notes). Open-items density (11 Open Questions, 17 inline assumptions, 5 `[NOTE FOR PM]` callouts) is high, but this PRD is explicitly scoped as a pre-architecture planning document (§0: "align on scope before architecture and build," not a green light to build), so the density is proportionate to genuine unresolved technical/business questions rather than a sign of an unready document.

## Downstream usability — strong

Glossary (§3) terms are used with consistent capitalization across FRs, UJs, and the Assumptions Index (Lead, Job, Trade, Service, Work Type, Contractor, Tier, etc.). FR IDs (FR-1–FR-19) are contiguous with no gaps or duplicates; UJ IDs (UJ-1–UJ-4) are contiguous; SM IDs (SM-1–SM-8 plus SM-C1–C3) are all present, just non-sequentially ordered across Primary/Secondary groupings (see Mechanical notes). "Realizes UJ-X" cross-references in FRs all resolve to defined UJs. Every UJ has a named protagonist carrying context inline (Maria, James, Devon, Priya) — no floating UJs.

## Shape fit — strong

This is a consumer-facing, multi-stakeholder marketplace with meaningful UX on both sides — UJs with named protagonists are correctly load-bearing here, not overhead. The PRD/addendum split itself is good shape judgment: capability decisions, FRs, and testable consequences stay in `prd.md`; technology choices, vendor research, and rejected alternatives are pushed to `addendum.md` (§0 states this explicitly). This keeps the PRD focused on "what" rather than "how" without losing the supporting research, and downstream architecture work can consume the addendum without re-litigating capability scope.

## Mechanical notes

- **Broken cross-reference (addendum.md → prd.md):** `addendum.md`'s "Salvaged ideas" section (under "Superseded Framing: Single-Sided Widget Model") refers to the Outcome Log and Self-Improving Pricing Engine as "PRD §4.6 FR-12" and "FR-13." In `prd.md`, §4.6 is the Contractor Job Queue Dashboard (FR-12/FR-13 are the card queue and low-click acceptance); Outcome Logging and Estimate Refinement are actually §4.7, FR-15/FR-16. An engineer following this reference would land on the wrong feature. *Fix:* correct the addendum's cross-references to §4.7/FR-15/FR-16.
- **Assumptions Index near-perfect roundtrip, one soft merge:** FR-4 (§4.2) carries two inline `[ASSUMPTION]` tags (visualization labeled as a concept; declining doesn't block submission). The second is indexed only under "§2.3 UJ-4," not separately under "§4.2 FR-4" — a minor consolidation rather than a missing entry, but worth flagging if strict 1:1 roundtrip is required downstream.
- **SM ordering:** Success Metrics are grouped Primary/Secondary/Counter rather than numbered sequentially within each group (Primary jumps SM-1, SM-2, SM-3, SM-7; Secondary picks up SM-4, SM-5, SM-6, SM-8). All eight IDs are present and none are duplicated — this is a readability quirk, not an ID-continuity defect.
- **Working title unconfirmed:** The document title carries "*Working title — confirm.*" (line 9) — flag for closure before this PRD is treated as final.
- **Minor Glossary-casing drift:** §1 Vision prose uses lowercase "homeowners"/"contractors" before the Glossary formalizes the capitalized terms (expected for vision-register prose, but worth a pass if the doc is used for term extraction verbatim from §1).
