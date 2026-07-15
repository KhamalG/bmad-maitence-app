# Reconciliation: DESIGN.md / EXPERIENCE.md vs. Property_Trades_Taxonomy.md

Source taxonomy: `docs/Property_Trades_Taxonomy.md` — ~20 Trade categories, each with a multi-row Service list, each Service tagged with one or more of 5 Work Types (Repair, Maintenance, Improvement/Renovation, Inspection, Emergency). Relational model: `ContractorTrades` (many) + `ContractorServices` (many), i.e. a contractor's catalog membership is a two-level, many-to-many selection, not a single dropdown.

Both DESIGN.md and EXPERIENCE.md list the taxonomy as a source. This reconciliation checks whether the UX spine actually surfaces the taxonomy's structure, or only references it by name/FR-number.

## Gap 1 — Contractor Trade/Service selection has no interaction pattern

EXPERIENCE.md mentions the taxonomy-driven step in exactly three places, all one-line references:

- IA table: `Verification / Onboarding | First login, pre-Job-Queue | Trade/Service selection, licensing/legitimacy verification gate (FR-11)`
- Component Patterns: `Verification gate | Contractor: first login | Hard gate — no Job Queue is reachable until verification (including Trade/Service confirmation) passes (FR-11)`
- State Patterns: `Unverified Contractor ... single Verification surface only, with clear status (pending / needs info / rejected)`

None of these specify: what component renders the Trade/Service picker, whether Trade selection happens before/separately from Service selection (the taxonomy's data model implies a two-level choice — pick Trades, then pick Services within each chosen Trade), how a contractor with multiple Trades (explicitly supported by `ContractorTrades` being many-to-many) manages that, or whether/how a contractor edits their Trade/Service list after initial onboarding (the spine only covers "first login" — there's no surface or component for later catalog edits, e.g., a Roofer who later adds Gutters). DESIGN.md's Components section has no picker/selector/checklist component at all — only buttons, cards, and badges. This is "verification happens" gestured at via FR-11, not a UI interaction.

## Gap 2 — No browse/search pattern for ~20 Trades × many Services, despite "doesn't overwhelm" being the stated brand goal

Neither document contains a search field, autocomplete, category/accordion, multi-select-with-chips, or any other pattern suited to selecting from a catalog of this size. DESIGN.md's Layout & Spacing section explicitly calls out "doesn't overwhelm" as most load-bearing in the *homeowner* intake flow — but the homeowner side is actually insulated from the taxonomy's volume by design (AI Diagnosis and Quick Submit both do silent Service-mapping from photos/free text, per the State Patterns row "Skip-AI submission ... passes Service-mapping ... silently"). The volume risk is entirely on the contractor onboarding side, which the "doesn't overwhelm" principle is never applied to. A raw 20-Trade × N-Service picker, if built literally from the taxonomy table without a defined browse/search/progressive-disclosure pattern, is the kind of dense, form-like, decision-heavy surface DESIGN.md's own Do's/Don'ts table warns against elsewhere ("Dense, form-like layouts during diagnosis" is a Don't) — but that guardrail is scoped to diagnosis, not onboarding, leaving onboarding unprotected by any explicit rule.

## Gap 3 — Work Type has one visual treatment (Emergency) out of five

DESIGN.md defines exactly one Work-Type-specific component: `emergency-badge`, and explicitly reserves Emergency Red "exclusively for Emergency Work Type" (Colors section, Do's/Don'ts table: "Emergency Red reserved for Emergency Work Type only"). EXPERIENCE.md's Lead Card pattern says the collapsed card shows "quality signal, trade, and urgency (Emergency Badge if applicable)" — trade is shown, but Work Type (Repair vs. Maintenance vs. Improvement/Renovation vs. Inspection) has no named badge, color, icon, or copy treatment anywhere in either file. Since a single Service can map to multiple Work Types (e.g., taxonomy's "Gutters & downspouts: Repair, Maintenance, Improvement" or "Water heaters: Repair, Improvement"), a Contractor scanning the Job Queue has no zero-tap way to distinguish a routine Maintenance lead from a Repair or an Improvement/Renovation lead — only Emergency is visually flagged; everything else is presumably plain trade-name text, if shown at all. This also affects the homeowner side: the IA table references "Inspection-mapped" and "Improvement-mapped" flow branches (Condition Assessment Result, Visualization Result) but never ties those branches to a consistent visual Work Type indicator the way Emergency has one.

## Summary

All three checks surfaced real, specific gaps — not just documentation thinness. The common thread: the spine documents cite the taxonomy's FR-numbers and use its vocabulary (Trade, Service, Work Type) correctly, but the taxonomy's two structural properties that most affect UX — (a) it's a two-level many-to-many catalog large enough to need a deliberate selection pattern, and (b) Work Type is a 5-value enum where only 1 value currently has a visual language — are not carried through into component or interaction specs.
