---
title: UX Spine vs. PRD Reconciliation
created: 2026-07-14
sources:
  - DESIGN.md
  - EXPERIENCE.md
  - ../../prds/prd-maitence-app-2026-07-14/prd.md
  - ../../prds/prd-maitence-app-2026-07-14/addendum.md
---

# Reconciliation: UX Design Pair vs. PRD

## Method

Checked every FR (FR-1–FR-19), every Feature (§4.1–4.9), and UJ-1–UJ-4 against DESIGN.md and EXPERIENCE.md for a corresponding surface, component, state, or interaction rule. Cross-checked Should-Have/MVP-scope exclusions against PRD §6.2 before flagging absence as a gap. Paid particular attention to the three areas called out: FR-15/16/19 scope correctness, the FR-18 Tier-Matching-Priority boundary, and the FR-11 Verification/Trade-Service onboarding flow.

## Confirmed correct (not gaps)

- **FR-16 (Estimate Refinement)** and **FR-19 (Performance Insights)** are correctly absent/deferred. FR-19 is explicitly listed in EXPERIENCE.md's IA table as "Nav (post-MVP, FR-19) ... Should-Have, not in MVP build" — matches PRD §6.2. FR-16 has no UX surface at all, which is correct since PRD §6.2 defers it entirely (no consuming UI exists yet, only the logging foundation).
- **FR-15 (Outcome Logging)** is correctly *included* — PRD §6.1 explicitly scopes it into MVP ("wired up early"), and EXPERIENCE.md's "Job Completion" surface captures "actual final cost + AI Diagnosis match/no-match into the Outcome Log (FR-15)." Correct inclusion, not a gap.
- UJ-1 through UJ-4 all have dedicated Key Flow walkthroughs in EXPERIENCE.md, matching the PRD's journeys step-for-step (including each journey's stated edge case).

## Gaps found

1. **FR-5's manual/fallback categorization path has no UX surface anywhere.** FR-5's testable consequence requires: "If the mapping cannot be made with confidence, the Lead is routed to a manual/fallback categorization step rather than silently dropped or mis-routed." Neither EXPERIENCE.md nor DESIGN.md defines what a Homeowner or Contractor/Admin sees when this triggers — no state pattern, no surface, no component. This is a real behavioral requirement with a testable consequence and zero corresponding UX.

2. **No Homeowner-facing state exists for a Lead that doesn't produce a match** (Cold Lead per FR-8, or unmapped-Service per FR-5 above). EXPERIENCE.md's IA table treats "Contractor Match" as an unconditional next step after Diagnosis/Quick Submit ("Single matched Contractor profile, why-matched context"), and the only Cold-Lead state defined ("Cold Lead filtered," State Patterns) is Contractor-side only ("no rejection notification to the Homeowner, no error state"). That tells us the Homeowner sees *nothing* — but there's no defined "no match yet / under review" screen or state for the Homeowner side of this, which is a real terminal path in the PRD's own trust/quality logic (FR-8, FR-10) and currently undesigned.

3. **FR-11's Trade/Service registration against the Taxonomy is named but not designed.** The task specifically flagged this to check. EXPERIENCE.md has an IA row ("Verification / Onboarding ... Trade/Service selection") and a gate component/state (hard block until verification + Trade/Service confirmation passes), so the *gate* itself is real. But the actual registration interaction — how a Contractor selects potentially multiple Trades and, within each, multiple Services from a ~20-category Taxonomy (per FR-6's Contractor↔Trade↔Service data model and addendum's schema) — has no component pattern, no picker/search/browse mechanism, and no partial-completion or validation state. This is thinner than FR-12/FR-13's Job Queue treatment, which got two dedicated component-pattern rows apiece. The verification *mechanism* (licensing/background check) being undesigned is correctly out of scope per PRD §4.5, but Trade/Service selection is explicitly in MVP scope (§6.1) and deserves its own IA/interaction treatment, not a single table cell.

4. **FR-18's Tier-Matching-Priority boundary is protected only as a visual convention, not as a behavioral/interaction rule.** DESIGN.md's Tier Badge do/don't ("Tier Badge visually neutral, never trust-coded... protects the FR-18 within-quality-band boundary from reading as 'paid = better'") is a real and good protection, but it lives entirely in DESIGN.md's component/branding layer. EXPERIENCE.md — the behavioral spine — never mentions FR-18, Tier, or Matching Priority in its Component Patterns, State Patterns, or Key Flows. Since the product shows the Homeowner exactly one matched Contractor (no visible ranking or competing list), there may genuinely be little else to design — the boundary is mostly backend logic with no UI surface to leak it. But as written, nothing in EXPERIENCE.md confirms this was a deliberate omission (e.g., no explicit "Contractor Match never exposes rank or competing Contractors" rule) — it reads as unaddressed rather than intentionally out of scope. Worth an explicit line in EXPERIENCE.md stating the single-match model is itself what keeps FR-18 invisible to end users, rather than leaving it implicit.

## Minor / secondary observations

- FR-3 (Condition Assessment) and FR-4 (Space Visualization) are Should/Could-Have features gated out of MVP per PRD §6.2 unless Phase 1 trade selection includes Inspection/Improvement Work Types (still an open question, §8 OQ1). EXPERIENCE.md includes full IA rows and Key-Flow treatment (UJ-4) for both without any "(post-MVP)" or "(conditional)" annotation — inconsistent with how Performance Insights (FR-19) was explicitly flagged as post-MVP in the same table. Not a contradiction of the PRD's content, but a labeling inconsistency that could cause these to be built as if committed-MVP when the PRD has not locked that decision.
- Contractor-side state for an in-flight async video attaching to an already-accepted Lead (FR-1's async video enrichment) is defined only on the Homeowner side ("Video still processing," State Patterns). No corresponding Contractor/Lead-Card state describes what the Contractor sees if they open a Lead before its video has finished processing.

## Bottom line

No PRD Must-Have feature or FR is entirely missing from the UX spines, and the Should-Have exclusions (FR-16, FR-19) are correctly scoped out. The four gaps above are specific, testable-consequence-level omissions: two intake failure-path states (FR-5 fallback, FR-8/no-match Homeowner-side) and one under-designed but in-scope onboarding flow (FR-11 Trade/Service picker), plus one design-integrity boundary (FR-18) that's protected visually but not behaviorally documented.
