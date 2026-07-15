---
title: Reconciliation — PRD/Addendum vs. Brainstorming Session
source_checked: brainstorm-ai-lead-intake-platform-2026-07-09 (brainstorm-intent.md + brainstorm.html)
targets_checked:
  - prd.md
  - addendum.md
generated: 2026-07-14
---

# Reconciliation: PRD + Addendum vs. Original Brainstorming Session

Method: read `prd.md`, `addendum.md`, `brainstorm-intent.md`, and the full raw `brainstorm.html` session (Role Playing, Six Thinking Hats, Job To Be Done, What-If Scenarios, Time Horizon Ladder, MoSCoW Convergence, Final Synthesis). Verified absence via targeted grep across both PRD documents before listing each gap below. Items that were consciously superseded by later decisions (e.g. competitive-differentiation framing after the Thumbtack research) are **not** included — those are intentional evolutions, already documented in `addendum.md`.

## Gaps Found

### 1. 1-Year vision milestone content — entirely dropped
The brainstorm's Time Horizon Ladder (technique 05) has a dedicated "1-year vision" rung: *"Real paying contractor customers, with concrete financial and technical KPIs — user count and daily engagement targets in place."* Neither `prd.md` nor `addendum.md` restates this milestone or its metrics categories anywhere. The PRD's Success Metrics (§7) define *what* to measure (conversion, completion rate, renewal, latency, etc.) but never anchors any of it to a 1-year checkpoint or mentions user-count/daily-engagement as metric categories at all. The MVP's 6-month target *did* carry over (§0 context, MVP scope), but the very next rung on the same ladder — the natural "MVP success looks like X in year one" statement — has no home in the PRD. Confirmed absent via grep for "1-year", "1 year", "daily engagement", "user count" — zero hits in either file.

### 2. 100-year full-automation speculation — not restated as a Non-Goal
`brainstorm-intent.md`'s MoSCoW explicitly lists as **Won't Have (this time)**: *"100-year full-automation speculation (AI autonomously fulfilling the underlying need)"* — i.e., AI eventually replacing the human-contractor-match model entirely, per the Time Horizon Ladder's 100-year rung ("AI becomes advanced enough to autonomously fulfill the underlying need itself — potentially collapsing the 'match human to human via AI' model entirely"). PRD §5 Non-Goals only restates the *10-year* local-services expansion and the *AI auto-assignment* What-If (a much narrower idea — AI auto-selecting/scheduling jobs for a still-human contractor, addressed in `addendum.md`'s "Rejected/Deferred Alternative" section). The 100-year full-automation idea is a distinct, more extreme speculation from the same MoSCoW "Won't Have" bucket and isn't mentioned anywhere in the PRD or addendum. Likely low-stakes given its speculative horizon, but it was a named, deliberately-scoped-out MoSCoW item and doesn't appear.

### 3. Red Hat risk: production-scale AI reliability — not carried into PRD risk/open-question framing
Six Thinking Hats' Red Hat (feelings/intuition) technique names a distinct unease: *"the product ships glitchy, or the core AI doesn't hold up reliably at production scale."* This is different in kind from the White Hat's "biggest unknown: how the photo/video AI analysis will actually work" (which the PRD does carry forward as the AI vision-analysis spike, §6.1/§8 OQ2) — the Red Hat concern is specifically about *production-scale reliability after* the mechanism is known to work, not about resolving the initial technical unknown. Neither `prd.md`'s Open Questions/Non-Goals nor `addendum.md`'s risk-adjacent sections (Rejected Alternative, Market Research) mention scale/reliability risk as a named concern. Confirmed absent via grep for "glitchy", "production scale", "reliably at" — zero hits.

### 4. Homeowner-facing budget-fit framing — narrowed to a contractor-side filter only
Role Playing's Homeowner hopes list: *"Finds contractors that fit their budget without shopping blind."* This frames budget-matching as a homeowner-facing benefit (the platform helps them find contractors in their price range, sparing them from comparison-shopping). The PRD carries the budget signal forward only as a contractor-side screening mechanism (FR-7 Budget-Realism Check, filtering Cold Leads) — the reciprocal "helps the homeowner avoid shopping blind for a contractor that fits their budget" framing doesn't appear in the Vision, JTBD, or feature descriptions. This is a minor but genuine loss of the original two-sided framing of the same signal.

## Not Flagged (verified as intentionally carried forward or superseded)

For completeness — these were checked and found adequately represented, not gaps:
- Hybrid skip-AI-diagnosis flow, misdiagnosis disclaimer, cold-lead filtering, contractor verification, card-queue/low-click dashboard, contact-reveal event tracking, tiered pricing, self-improving pricing engine, revenue analytics, brand/marketing moat (Could-have) — all present with FR numbers.
- Budget-realism two-cause mechanism (education vs. lowball) — captured verbatim in addendum's "Budget-Realism Check: Two-Cause Mechanism" section, correctly marked deferred.
- Admin/Dashboard User as distinct persona with two named UX failure modes (information overload, too-many-clicks) — captured in addendum's "Persona Depth" section and PRD §2.1/§4.5.
- Reciprocal trust ("both directions") — FR-8.
- Data-flywheel / outcome-logging-wired-up-early argument from Final Synthesis — captured near-verbatim in PRD §4.6 FR-12 Notes and brainstorm-intent's "Deepest JTBD" section.
- Core tension blockquote ("give people an honest choice about how much they lean on the guess") — paraphrased faithfully into PRD §1 Vision's hybrid-intake reasoning.
- "First AI diagnosis" differentiation claim — deliberately superseded by Thumbtack competitive research (addendum), not a silent drop.
- Yellow Hat aspirational brand framing ("default first-thought platform," "proves LLM-driven analysis can improve lives") — captured verbatim in addendum's "Aspirational Framing" section.
- Bypass-the-platform risk — implicitly addressed via FR-11 Contact-Reveal Event and UJ-3's "stays in the loop regardless of how he ultimately follows up."
- Tech stack decisions, AI vision-analysis spike priority, Phase-1 trade-subset narrowing — all carried forward.
