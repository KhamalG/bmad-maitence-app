---
title: AI-Powered Lead Intake Platform for Contractors
status: final
created: 2026-07-14
updated: 2026-07-14
---

# PRD: AI-Powered Lead Intake Platform for Contractors
*Working title — confirm.*

## 0. Document Purpose

This PRD is an internal planning document for Khamal and early collaborators (co-founders, early engineers) to align on scope before architecture and build. It builds directly on the [2026-07-09 brainstorming session](../../brainstorming/brainstorm-ai-lead-intake-platform-2026-07-09/brainstorm-intent.md) and the [Property Services Taxonomy](../../../../docs/Property_Trades_Taxonomy.md) — this doc distills and structures those inputs' decisions; it does not re-litigate them. Vocabulary is Glossary-anchored (§3); features are grouped with functional requirements (FRs) nested under them and numbered globally; inferred content is tagged `[ASSUMPTION]` inline and indexed in §9. Technology choices, rejected alternatives, and supporting research live in the companion `addendum.md`, not here.

## 1. Vision

Homeowners with a home service problem — a leaking pipe, a flickering breaker, a broken unit — don't just need a quote, they need confidence that they understand what's wrong and who to trust to fix it. This platform uses AI photo and video analysis to give homeowners a fast, preliminary diagnosis of their problem before they ever talk to a contractor, then matches them to a single, vetted, well-suited contractor rather than auctioning their contact information to a handful of competing bidders. Speed matters as much as accuracy here — a homeowner mid-crisis with a leaking pipe should not be left waiting on a diagnosis (see FR-1 NFRs).

On the other side, small and mid-size trade contractors are hired to fix homes, not to run a call center. Today they either absorb the overhead of dedicated intake staff or drown in unqualified, duplicated leads from marketplaces that bill them regardless of whether the lead converts. This platform replaces that admin burden with a low-click, card-based job queue that only surfaces leads worth their time — pre-qualified by completeness of detail and budget realism — and gives them tools to grow revenue, not just cut cost.

The product succeeds by solving the trust gap that undermines every AI-diagnosis attempt in this space: a hybrid intake flow gives new or unsure homeowners the full guided AI assessment (with a clear misdiagnosis disclaimer), while homeowners who already know what they need can skip straight to booking. AI Diagnosis is not a feature to hedge on — competitors have already entered this space (see `addendum.md`), so the differentiating bar is making the analysis itself best-in-class, not merely offering it. Phase 1 deliberately narrows to a defined subset of trades, selected from the full [Property Services Taxonomy](../../../../docs/Property_Trades_Taxonomy.md) — provisionally **Plumbing, Electrical, and HVAC** `[ASSUMPTION: directional, not locked — see §8 Open Question 1]` — with multi-trade expansion (the rest of the taxonomy) as a later phase.

## 2. Target User

### 2.1 Jobs To Be Done

**Homeowner:**
- Functional: get an accurate-enough preliminary diagnosis of a home service problem without needing to already know the vocabulary or the market — including a realistic sense of whether their budget fits the job, so they're not shopping blind on price (see FR-9).
- Emotional: relieve the anxiety of not knowing what's wrong or who to trust — the job is "give me confidence and insight to book with certainty," not merely "get a quote."
- Social/repeat: once a homeowner finds a contractor they trust through the platform, become a repeat customer — the platform is the on-ramp to an ongoing relationship, not a one-time transaction. `[ASSUMPTION: repeat-usage retention is a secondary funnel worth designing for even though it isn't a Must-have MVP feature — flagged for future consideration, not scoped in §6.]`

**Contractor:**
- Functional: replace the cost of dedicated intake/admin staff at a fraction of the price.
- Emotional: alleviate the overwhelm of running a small trades business — the deepest job is not "reduce admin cost" but "reduce operational overwhelm," with AI-driven insight actively helping drive revenue rather than only cutting cost.
- Trust: identify homeowners who are serious and well-qualified, not just avoid obviously fake leads — trust runs in both directions on this platform (see FR-10).

**Admin / Dashboard User** *(distinct from "the contractor" as a persona — the premium-tier staff member who actually works the queue day to day):*
- Functional: process a high volume of leads without missing time-sensitive ones or getting buried in detail.
- Emotional: avoid two distinct failure modes — information overload on a card's detail view, and a job-acceptance flow with too many clicks to keep up during a busy day.

### 2.2 Non-Users (v1)

- Homeowners and contractors in trades outside the Phase 1 subset (provisionally Plumbing, Electrical, HVAC — §8 Open Question 1, not locked).
- Enterprise/multi-location trade businesses requiring multi-seat account hierarchies (tier structure is deferred, see §6.2).
- DIY homeowners who want step-by-step repair guidance instead of a contractor match — this platform routes to a professional, it does not replace one. `[ASSUMPTION]`

### 2.3 Key User Journeys

- **UJ-1. Maria gets a same-day diagnosis for a leaking water heater.**
  - **Persona + context:** Maria, a first-time homeowner, notices water pooling near her water heater and has never hired a contractor for this before.
  - **Entry state:** Unauthenticated, arrives via the platform's mobile app after a search.
  - **Path:** She's guided through the full AI-assessment flow — prompted to take photos and a short video of the leak, answer a few clarifying questions, and provide a rough budget expectation. The AI produces a preliminary diagnosis ("likely a failed pressure relief valve or tank corrosion") with a clear misdiagnosis disclaimer, and maps the issue to the Plumbing Trade's "Water heaters" Service.
  - **Climax:** She's matched to a single verified contractor whose profile shows relevant experience, and sees why this contractor was chosen rather than a list to sift through herself.
  - **Resolution:** She books a visit; the contractor receives her lead with her photos, video, and AI diagnosis attached, so she doesn't have to re-explain the problem on a call.
  - **Edge case:** If her photos are too low-detail for a confident diagnosis, the AI asks for one more angle rather than guessing — this preserves the lead-quality signal (FR-8) that gets passed to the contractor. `[ASSUMPTION]`

- **UJ-2. James skips straight to booking an electrician he's used before.** James, a returning authenticated user who knows he just needs an outlet replaced, chooses the Skip-AI Path from the home screen, describes the job in a line or two, and is matched and booked without the full guided assessment — the Contractor still receives a lead that's passed Service-mapping and completeness/budget-realism checks, just without an AI Diagnosis attached.

- **UJ-3. Devon (contractor/admin) clears the morning queue in under five minutes.**
  - **Persona + context:** Devon runs a two-person plumbing outfit and checks the dashboard between jobs.
  - **Entry state:** Authenticated, web dashboard on a phone browser.
  - **Path:** He opens the card-based job queue; each card shows lead-quality signal, trade, and urgency at a glance. He taps a card to expand full detail (photos, AI diagnosis, budget) only for leads worth a closer look, and skips past ones already filtered as cold.
  - **Climax:** He accepts a well-qualified lead in two taps — no multi-screen form.
  - **Resolution:** The accepted job appears in his active queue with the homeowner's contact revealed and a contact-reveal event logged (FR-13) so the platform stays in the loop regardless of how he ultimately follows up.
  - **Edge case:** If Devon is on a premium tier with dedicated admin staff, the admin user performs this same flow on Devon's behalf — same mechanism, different seat. `[ASSUMPTION]`

- **UJ-4. Priya envisions her kitchen remodel before contacting anyone.** Priya isn't fixing anything broken — she wants to see what a kitchen remodel could look like in her space before reaching out to anyone. She submits photos and describes what she's imagining ("open up the island, lighter cabinets"); the AI returns a visualization rendered in her actual kitchen, clearly labeled as a concept, not a binding design or quote. She submits it with her Lead, giving the matched Contractor a concrete starting point. Realizes FR-4. Declining to wait for a visualization doesn't block submission — it's additive, not a gate. `[ASSUMPTION]`

## 3. Glossary

- **Homeowner** — The requester of a home service job. Free user of the platform.
- **Contractor** — A verified trades business that receives and fulfills leads through the platform. Registered against one or more Trades and Services. Pays for platform access via a Tier.
- **Admin / Dashboard User** — A staff member (at a premium Tier) who operates the Contractor's job queue on the Contractor's behalf. Distinct persona from the Contractor account owner, same underlying dashboard.
- **Lead** — A homeowner's submitted request for service, carrying its mapped Service(s) and Lead-Quality Signal, before a Contractor accepts it. Becomes a Job once accepted.
- **Job** — An accepted Lead, now assigned to a Contractor and tracked through completion.
- **Trade** — A top-level category of contracting work (e.g. Plumbing, Electrical, Roofing), per the [Property Services Taxonomy](../../../../docs/Property_Trades_Taxonomy.md). A Contractor may belong to multiple Trades.
- **Service** — A specific offering within a Trade (e.g. "Leak repair" within Plumbing), carrying a Work Type. A Lead's AI Diagnosis maps to one or more Services, which in turn determine which Contractors qualify to receive it.
- **Work Type** — The nature of a Service: Repair, Maintenance, Improvement/Renovation, Inspection, or Emergency, per the Taxonomy. Note: "Inspection" also names a standalone Trade ("Inspection Services") in the Taxonomy — the Trade field and the Work Type field are distinct, and FR-5's mapping logic must not conflate the two.
- **AI Diagnosis** — The preliminary assessment of a homeowner's problem generated from submitted photos/video/description during the Guided Assessment Flow, for Repair/Emergency Work Types. Always shown with the Misdiagnosis Disclaimer.
- **Condition Assessment** — The Inspection Work Type's equivalent of an AI Diagnosis: a preliminary read on a space's condition rather than a diagnosis of damage. See FR-3.
- **Space Visualization** — An AI-generated visual concept of a requested Improvement/Renovation rendered in the Homeowner's own submitted photos, clearly labeled as a concept, not a binding design or quote. See FR-4.
- **Misdiagnosis Disclaimer** — The required, clearly visible notice that an AI Diagnosis, Condition Assessment, or Space Visualization is preliminary/conceptual and may be inaccurate, shown to every Homeowner who receives one.
- **Hybrid Intake Flow** — The two-path intake mechanism: the Guided Assessment Flow (full AI diagnosis) for new/unsure Homeowners, or the Skip-AI Path for returning/knowledgeable Homeowners. Both paths produce a Lead.
- **Guided Assessment Flow** — The full AI-assisted intake path: photo/video capture, clarifying questions, AI Diagnosis, Misdiagnosis Disclaimer.
- **Skip-AI Path** — The shortened intake path for Homeowners who already know what they need; bypasses AI Diagnosis but still produces a Lead subject to Service mapping and Lead-Quality Signal checks.
- **Lead-Quality Signal** — The composite read on a Lead's worth pursuing: submission completeness (photos/video/description depth) and budget realism. Cold Leads fail this signal.
- **Cold Lead** — A Lead that fails the Lead-Quality Signal check and is filtered out before reaching a Contractor's queue.
- **Budget-Realism Check** — The check comparing a Homeowner's stated budget against market expectations for the described Job. `[ASSUMPTION: MVP implements as a single check; see FR-9 for the deferred education-vs-lowball split.]`
- **Verification System** — The process and status by which a Contractor's legitimacy (licensing, reviews, identity) and Trade/Service qualifications are confirmed before they can receive Leads.
- **Job Queue** — The Contractor/Admin-facing, card-based view of incoming Leads and active Jobs. Emergency Work Type Leads receive expedited priority within this view (FR-14).
- **Contact-Reveal Event** — The logged event marking the moment a Homeowner's contact information is revealed to a Contractor, regardless of the channel subsequently used to communicate.
- **Tier** — A Contractor's subscription level (Tier 1, 2, or 3), determining dashboard features, Matching Priority, and whether the account includes dedicated Admin/Dashboard User seats. Exact feature/pricing boundaries deferred (§8).
- **Matching Priority** — The rank boost a Contractor's Tier confers among Contractors already qualified and fit for a Lead. Operates within qualification bands only — never lets a lower-fit Contractor outrank a better-fit one regardless of Tier.
- **Outcome Log** — The record of a Job's actual final cost/outcome, captured after completion, feeding the Self-Improving Pricing Engine (FR-16).

## 4. Features

### 4.1 Hybrid AI-Guided Intake

**Description:** The entry point for every Repair/Emergency Lead. New or unsure Homeowners are guided through the full AI Diagnosis flow with a mandatory Misdiagnosis Disclaimer; returning or knowledgeable Homeowners can take the Skip-AI Path straight to booking. Realizes UJ-1, UJ-2.

**Functional Requirements:**

#### FR-1: Guided Assessment Flow with AI Diagnosis

Homeowner can submit photos, video, and a description of a problem and receive a preliminary AI Diagnosis. Realizes UJ-1.

**Consequences (testable):**
- System requires at least one photo before AI Diagnosis is attempted.
- Misdiagnosis Disclaimer is displayed alongside every AI Diagnosis result, with no path to dismiss it permanently.
- If submitted media is judged too low-detail for a confident diagnosis, the system prompts for additional detail rather than returning a low-confidence diagnosis silently.
- AI Diagnosis is returned from photo and description input without waiting on video processing; if the Homeowner also submits video, it enriches the Lead but does not delay delivery of the initial AI Diagnosis.

**Feature-specific NFRs:**
- Photo-based AI Diagnosis is delivered synchronously, at speed the Homeowner does not perceive as a wait. `[ASSUMPTION: no numeric latency target set yet — see §8 Open Question 2; directionally, low seconds not minutes.]`
- Video analysis, where submitted, is processed asynchronously and surfaced to the Contractor once ready, without blocking the Homeowner's initial diagnosis or booking flow.

**Out of Scope:**
- The underlying AI vision-analysis mechanism/model is not specified here — see `addendum.md` and the AI vision-analysis research spike noted in §6.1.

#### FR-2: Skip-AI Path

Returning or knowledgeable Homeowner can bypass the Guided Assessment Flow and submit a Lead directly. Realizes UJ-2.

**Consequences (testable):**
- Leads submitted via the Skip-AI Path still pass through Issue-to-Service Mapping (FR-5) and the Lead-Quality Signal check (FR-8) before reaching a Contractor's queue.
- No AI Diagnosis or Misdiagnosis Disclaimer is generated for Skip-AI Path Leads.

**Notes:** `[NOTE FOR PM]` Criteria for who is offered the Skip-AI Path by default (e.g. "used the platform before" vs. self-declared confidence) is not yet defined — see §8.

### 4.2 Improvement & Inspection Intake

**Description:** The Guided Assessment Flow (§4.1) is built to diagnose a problem from photos — it doesn't fit Leads mapped to Inspection or Improvement/Renovation Work Types, where there's no damage to diagnose, only a space to assess or a change to visualize. This is a second intake variant for those Work Types, reusing the same photo/video capture mechanism but changing what the AI produces from it. `[NOTE FOR PM]` Both FRs below are gated behind Phase 1 trade/Work-Type selection (§8 Open Question 1) — if Phase 1 stays Repair/Emergency-focused (the addendum's directional read), neither ships in MVP; see §6.2.

**Functional Requirements:**

#### FR-3: Condition Assessment for Inspection Leads *(Should Have)*

Homeowner can submit photos/video and a description of what they want inspected and receive a preliminary Condition Assessment.

**Consequences (testable):**
- The AI output for an Inspection-mapped Lead is labeled and framed as a Condition Assessment, not a damage diagnosis — same accuracy caveat as the Misdiagnosis Disclaimer, different framing. `[ASSUMPTION]`
- Subject to the same Issue-to-Service Mapping (FR-5) and Lead-Quality Signal (§4.4) checks as any other Lead.

**Out of Scope:**
- Does not apply to Repair/Emergency Work Types — those continue through FR-1.

#### FR-4: Space Visualization for Improvement/Renovation Leads *(Could Have)*

Homeowner can submit photos/video of a space and describe a desired Improvement/Renovation, and optionally receive an AI-generated Space Visualization of the described change rendered in their actual space. Realizes UJ-4.

**Consequences (testable):**
- Visualization output is clearly labeled as an AI-generated concept, not a binding design or quote — the same trust-preserving disclaimer principle as the Misdiagnosis Disclaimer, applied to a generative rather than diagnostic output. `[ASSUMPTION]`
- Declining or skipping visualization does not block Lead submission — it is additive, not a gate. `[ASSUMPTION]`

**Out of Scope:**
- The generative mechanism (which model/service produces the visualization) is not specified here — this is a materially different AI capability (generative image/design rendering) than the diagnostic AI Diagnosis (FR-1) and needs its own feasibility spike, separate from and likely after the AI vision-analysis spike in §6.1. See `addendum.md` and §8 Open Question 11.

### 4.3 Issue-to-Service Matching

**Description:** The mechanism connecting intake to Contractor eligibility. Every Lead — regardless of which intake variant produced it — is mapped to one or more Services within a Trade, per the Property Services Taxonomy, before it can be routed to a Contractor. Contractors register the Trades and Services they're qualified for (confirmed via the Verification System, FR-11), and only matching Contractors ever see a given Lead. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-5: Issue-to-Service Mapping

System maps a Lead's AI Diagnosis/Condition Assessment (or, for Skip-AI Path Leads, the Homeowner's self-described issue) to one or more Services within a Trade, per the Taxonomy. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Every Lead carries at least one mapped Service before it can be routed to a Contractor.
- If the mapping cannot be made with confidence, the Lead is routed to a manual/fallback categorization step rather than silently dropped or mis-routed. `[ASSUMPTION]`

**Out of Scope:**
- The specific mapping algorithm (rules-based vs. model-driven) is not specified here — see `addendum.md`.

#### FR-6: Trade/Service-Qualified Contractor Matching

System routes a Lead only to Contractors registered and verified for the Lead's mapped Service(s). Realizes UJ-3.

**Consequences (testable):**
- A Contractor never sees a Lead for a Service outside their registered Trades/Services.
- A Contractor can register for multiple Trades and multiple Services within each Trade, per the Taxonomy's Contractor↔Trade↔Service data model.

### 4.4 Lead Quality & Trust Signals

**Description:** Every Lead, regardless of intake path, is scored against completeness and budget realism before it reaches a Contractor's Job Queue. This is also where reciprocal trust — a Contractor's ability to trust the Homeowner, not just the reverse — is addressed: the same completeness and budget signals that filter Cold Leads for Contractors double as the platform's read on whether a Homeowner is a serious, well-qualified requester. Realizes UJ-1, UJ-3.

**Functional Requirements:**

#### FR-7: Submission Completeness Signal

System evaluates a Lead's detail depth (photos, video, description) as part of its Lead-Quality Signal.

**Consequences (testable):**
- A Lead with no photos and a description under a minimum length is flagged as lower-quality before reaching the queue. `[ASSUMPTION: minimum length not yet numerically defined — directional only.]`

#### FR-8: Cold-Lead Filtering

System filters Leads that fail the Lead-Quality Signal check before they reach a Contractor's Job Queue. Realizes UJ-3.

**Consequences (testable):**
- Cold Leads do not appear in the Contractor's default Job Queue view.
- Contractor can optionally view filtered Cold Leads in a separate, clearly labeled view (not deleted, just deprioritized). `[ASSUMPTION]`

#### FR-9: Budget-Realism Check

System compares a Homeowner's stated budget against market expectations for the described Job as part of the Lead-Quality Signal.

**Consequences (testable):**
- A Lead with a budget significantly below market expectation for the described Job is flagged as lower-quality. `[ASSUMPTION: "significantly below" not yet numerically defined — directional only, likely a percentage-below-market-range threshold.]`
- The check is not purely a silent Contractor-side filter: a Homeowner whose stated budget appears unrealistic sees a brief realism note before submitting, helping them self-correct or proceed with clear expectations rather than shopping blind. `[ASSUMPTION]`

**Out of Scope:**
- Distinguishing *why* a budget is unrealistic (Homeowner doesn't know market pricing vs. intentional lowballing) is deferred past MVP — MVP implements a single undifferentiated check. The two-cause mechanism is captured in `addendum.md` for future design.

#### FR-10: Reciprocal Trust via Lead-Quality Signal

Contractor's view of a Lead's quality score reflects the same signal used to filter Cold Leads — no separate homeowner-reliability feature. Realizes UJ-3.

**Consequences (testable):**
- No standalone "homeowner reputation" feature ships in MVP; Contractors rely on FR-7/FR-9 signals as their vetting mechanism for the Homeowner.

**Notes:** `[NOTE FOR PM]` If Contractor feedback post-launch indicates the completeness/budget signal isn't sufficient reciprocal-trust coverage, revisit as a distinct feature — flagged in §8.

### 4.5 Contractor Verification & Legitimacy

**Description:** Before a Contractor can receive Leads, their legitimacy — and their Trade/Service qualifications — are verified. This is the platform's primary defense against fake reviews, self-referrals, and manipulated lead/job data (contractor gaming risk, see `addendum.md` risk register), and it is what makes Issue-to-Service Matching (§4.3) trustworthy.

**Functional Requirements:**

#### FR-11: Contractor Verification Gate

Contractor cannot receive Leads until they pass the Verification System, including confirmation of the Trades/Services they're claiming to offer.

**Consequences (testable):**
- An unverified Contractor account has no visible Job Queue and cannot appear in Issue-to-Service Matching (FR-6).

**Out of Scope:**
- The specific verification mechanism (licensing lookup, manual review, background check vendor) is not specified here — flagged in §8.

### 4.6 Contractor Job Queue Dashboard

**Description:** The card-based, low-click dashboard that is the literal delivery vehicle for relieving Contractor/Admin overwhelm. Explicitly designed against two distinct UX failure modes surfaced during discovery: information overload on the card/detail view, and too many clicks to accept a job. Realizes UJ-3.

**Functional Requirements:**

#### FR-12: Card-Based Queue with Expandable Detail

Contractor/Admin can view incoming Leads as summary cards and expand any card to full detail. Realizes UJ-3.

**Consequences (testable):**
- Default card view shows lead-quality signal, trade, and urgency without requiring a tap.
- Expanding a card reveals AI Diagnosis (if present), photos/video, and budget without navigating to a separate screen.

#### FR-13: Low-Click Job Acceptance with Contact-Reveal Tracking

Contractor/Admin can accept a Lead in minimal taps, and the system logs a Contact-Reveal Event when the Homeowner's contact information is revealed. Realizes UJ-3.

**Consequences (testable):**
- Accepting a well-qualified Lead from the queue takes no more than two taps/clicks.
- A Contact-Reveal Event is recorded at the moment contact info becomes visible to the Contractor, independent of whether the Contractor ultimately calls, texts, or messages in-app.

#### FR-14: Emergency-Priority Queue Routing

Leads mapped to a Service with Emergency Work Type bypass standard Job Queue ordering and are surfaced to qualified Contractors with expedited notification.

**Consequences (testable):**
- An Emergency-Work-Type Lead appears at the top of the Contractor/Admin's default Job Queue view regardless of submission time relative to other Leads.
- Contractor/Admin receives an expedited alert (e.g. push notification) for Emergency-Work-Type Leads, distinct from standard new-lead notification. `[ASSUMPTION]`

**Out of Scope:**
- SLA commitments (e.g. "Contractor must respond within X minutes") are not specified here — flagged as a possible future Tier-differentiated feature, not committed in MVP.

**Feature-specific NFRs:**
- Job Queue must remain usable on a phone browser, not just desktop — Admin/Dashboard Users check between jobs, often on mobile. `[ASSUMPTION]`

### 4.7 Self-Improving Pricing Engine *(Should Have)*

**Description:** Learns from a Contractor's historical Job costs and materials (via the Outcome Log) to auto-generate more accurate estimates over time.

**Functional Requirements:**

#### FR-15: Outcome Logging

System captures a Job's actual final cost/outcome after completion into the Outcome Log.

**Consequences (testable):**
- Every completed Job prompts for actual final cost before being marked closed. `[ASSUMPTION]`
- For Jobs that originated from an AI Diagnosis or Condition Assessment, the Outcome Log also captures whether it matched the Contractor's actual finding — this feeds both estimate refinement (FR-16) and AI Diagnosis accuracy tracking (SM-8).

**Notes:** Outcome Logging (FR-15) should be wired up early even though the full pricing engine that consumes it stays a Should-have — the data flywheel needs a head start. Its role as the ground-truth source for AI Diagnosis accuracy (not just pricing) makes it more load-bearing than the "Should Have" framing alone suggests.

#### FR-16: Estimate Refinement from Outcome Log

System uses a Contractor's accumulated Outcome Log to improve future estimate accuracy for that Contractor's Leads.

**Out of Scope:**
- Cross-Contractor learning / platform-wide pricing models are not in scope for this feature.

### 4.8 Tiered Pricing Model *(Should Have)*

**Description:** Contractor-side monetization, structured as three Tiers:

- **Tier 1** — the basics a new Contractor needs to run leads through the platform, self-managed.
- **Tier 2** — more dashboard features, undefined additional premium features (TBD, see §8), and a Matching Priority boost.
- **Tier 3** — the top package: maximum Matching Priority boost (typically first shown to Homeowners among qualified matches) plus dedicated Admin/Dashboard User seats.

`[NOTE FOR PM]` Matching Priority is deliberately scoped to operate *within* Lead-Quality/fit-qualified matches only (FR-18) — a higher Tier increases a Contractor's rank among comparably-qualified Contractors, but never lets a worse-fit, lower-quality-scored Contractor outrank a better-fit one. This boundary is what keeps the model from recreating the pay-to-play dynamic that drives the incumbent complaints documented in `addendum.md` — it should be preserved as tier specifics get filled in.

**Functional Requirements:**

#### FR-17: Tier-Gated Access

Contractor's Tier determines available dashboard features and Admin seat allowance.

**Out of Scope:**
- Specific Tier names, pricing, and Tier 2/3 premium feature boundaries are deferred — see §8. `addendum.md` notes an AI price estimator (beta/premium) as one candidate Tier 2/3 feature, not yet decided.

#### FR-18: Tier-Based Matching Priority

Contractor's Tier influences their rank/visibility within the set of Contractors already qualified for a Lead via Issue-to-Service Matching (FR-6) and passing the Lead-Quality Signal (§4.4) — Tier boosts rank among comparably-qualified matches, it does not override qualification or fit.

**Consequences (testable):**
- Among Contractors equally qualified and fit for a given Lead, a higher-Tier Contractor is presented before a lower-Tier one.
- A Tier 3 Contractor never outranks a better-fit Tier 1 Contractor for a Lead the Tier 1 Contractor is more qualified for — Matching Priority operates within qualification bands, not across them.

**Out of Scope:**
- The precise ranking algorithm/weighting within a qualification band is not specified here — flagged for architecture.

### 4.9 Revenue-Driving Analytics *(Should Have)*

**Description:** Dashboard insights aimed at helping Contractors grow revenue, not just reduce admin cost — directly addresses the deepest contractor JTBD (§2.1).

**Functional Requirements:**

#### FR-19: Contractor Performance Insights

Contractor can view analytics on Lead-to-Job conversion, response time, and Job volume trends.

**Out of Scope:**
- Specific metrics and visualizations are not specified here — flagged for UX work.

## 5. Non-Goals (Explicit)

- This platform is not expanding beyond the Phase 1 trade subset in v1 (multi-trade expansion is an explicit later phase, drawing from the rest of the Property Services Taxonomy).
- This platform is not pursuing the broader "10-year local-services" vision (barbers, stylists, mechanics, etc.) in this PRD's scope.
- This platform is not pursuing "100-year" full-automation speculation (AI autonomously fulfilling the underlying home-service need without a human contractor) — named explicitly as a Won't-Have in the brainstorming session, distinct from and further out than the AI auto-assignment idea below.
- This platform is not attempting full AI-driven job auto-assignment (AI automatically selecting and scheduling jobs on a Contractor's behalf without human review) — this was explicitly considered and set aside as premature until the AI trust gap is further resolved. See `addendum.md`.
- This platform does not charge Homeowners; monetization is entirely Contractor-side.

## 6. MVP Scope

MVP scope is problem-solving-first: it keeps the diagnostic/matching/trust core intact end-to-end and defers monetization depth (pricing engine, analytics, full tier detail) rather than front-loading revenue features.

### 6.1 In Scope

- Hybrid AI-Guided Intake: Guided Assessment Flow + Skip-AI Path, for Repair/Emergency Work Types (FR-1, FR-2)
- Issue-to-Service Matching: mapping Leads to Services and routing to Trade/Service-qualified Contractors (FR-5, FR-6)
- Lead Quality & Trust Signals: completeness signal, budget-realism check (with homeowner-facing realism note), Cold-Lead filtering, reciprocal trust via existing signals (FR-7–FR-10)
- Contractor Verification & Legitimacy gate, including Trade/Service qualification (FR-11)
- Card-based Job Queue Dashboard with expandable detail, low-click acceptance, and Emergency-priority routing (FR-12–FR-14)
- Outcome Logging foundation (FR-15) — wired up early even though the consuming pricing engine is a Should-have
- AI vision-analysis research spike, run before further build-out on the Guided Assessment Flow, to resolve the platform's biggest technical unknown

### 6.2 Out of Scope for MVP

- Improvement & Inspection Intake (FR-3, FR-4) — gated behind Phase 1 trade/Work-Type selection (§8 Open Question 1); ships only if Phase 1 includes Inspection or Improvement/Renovation Work Types. FR-4 (Space Visualization) additionally requires its own feasibility spike regardless of Phase 1 scope (§8 Open Question 11).
- Self-Improving Pricing Engine estimate refinement (FR-16) — deferred to post-MVP once Outcome Log has accumulated data.
- Full Tiered Pricing Model detail — Tier names, pricing, and Tier 2/3 premium feature boundaries deferred (FR-17 gate and FR-18 priority-boost concept exist, specifics don't).
- Revenue-Driving Analytics (FR-19) — deferred; MVP dashboard focuses on queue/acceptance, not insights.
- Contact-Reveal Event tracking beyond basic logging — deferred richer reporting on this data.
- Budget-Realism Check cause-differentiation (education vs. lowball) — single check with a light realism note ships in MVP; the full split is a post-MVP refinement.
- Trade/Service catalog beyond the Phase 1 subset — the full Property Services Taxonomy exists as a long-term data model, but only the Phase 1 subset is active for matching (FR-6) and verification (FR-11) in MVP.
- Emergency-response SLA commitments — expedited routing/notification ships (FR-14), but no committed response-time SLA.
- `[NOTE FOR PM]` Brand/marketing moat strategy (differentiation via positioning/creative rather than tech) is a Could-have — worth revisiting once the competitive landscape in `addendum.md` is validated with real market research, since Thumbtack has already entered this space.

## 7. Success Metrics

**Primary**
- **SM-1**: Contractor Lead-to-Job conversion rate — target meaningfully above the shared-lead-marketplace baseline (Angi/Thumbtack industry conversations cite conversion complaints as a chief driver of churn — see `addendum.md`). Validates FR-7, FR-8, FR-9. `[ASSUMPTION: no numeric target set yet — see §8.]`
- **SM-2**: Guided Assessment Flow completion rate — percentage of Homeowners who start the flow and reach a submitted Lead. Validates FR-1.
- **SM-3**: Contractor Tier renewal rate — validates that the dashboard delivers enough overwhelm-relief and lead quality to retain paying Contractors. Validates FR-8, FR-12, FR-13.
- **SM-7**: AI Diagnosis latency — time from photo/description submission to a delivered AI Diagnosis. Speed is treated as core to the value proposition, not a secondary quality attribute. Validates FR-1. `[ASSUMPTION: no numeric target set yet — directionally "low seconds," see §8 Open Question 2.]`

**Secondary**
- **SM-4**: Median clicks-to-accept on the Job Queue. Validates FR-13.
- **SM-5**: Contact-Reveal Event rate relative to Lead volume — an early read on whether Homeowners are bypassing the platform post-match. Validates FR-13.
- **SM-6**: Issue-to-Service mapping accuracy — rate at which FR-5's mapping requires manual/fallback categorization vs. confident auto-mapping. Validates FR-5.
- **SM-8**: AI Diagnosis accuracy rate — percentage of AI Diagnoses that matched the Contractor's actual finding, measured via the Outcome Log's diagnosis-match capture. Validates FR-1, FR-15.

**Counter-metrics (do not optimize)**
- **SM-C1**: Lead volume delivered to Contractors — do not optimize this upward at the expense of Lead-Quality Signal accuracy (SM-1); flooding Contractors with more, lower-quality Leads recreates the exact complaint driving Homeowners and Contractors away from incumbents. Counterbalances SM-1.
- **SM-C2**: AI Diagnosis confidence/assertiveness — do not optimize the AI toward more confident-sounding diagnoses to improve completion rate (SM-2); an overconfident AI Diagnosis that turns out wrong is the core trust failure this product is built to avoid. Counterbalances SM-2.
- **SM-C3**: Tier-driven revenue (Tier 2/3 upsell rate) — do not optimize Tier revenue by loosening FR-18's within-quality-band boundary on Matching Priority; letting a paying Contractor outrank a genuinely better-fit one recreates the pay-to-play dynamic this platform is positioned against. Counterbalances any future Tiered Pricing revenue metric.

## 8. Open Questions

1. Which specific Trades/Services from the Property Services Taxonomy make up the Phase 1 subset? Provisionally set to **Plumbing, Electrical, and HVAC** `[ASSUMPTION]` — a directional read from the Vision's own examples (leaking pipe, flickering breaker, broken unit), not a validated decision. This still needs confirmation (and Service-level granularity within each Trade) before architecture work and the AI vision-analysis spike can proceed with confidence — it also determines whether Improvement & Inspection Intake (§4.2) ships in MVP at all, since Plumbing/Electrical/HVAC lean Repair/Emergency rather than Inspection/Improvement.
2. AI vision-analysis spike: technical direction is set (see `addendum.md`). What remains open is the exit criteria: concrete numeric targets for AI Diagnosis latency (SM-7) and accuracy (SM-8) that define "best-in-class" for this platform, and validation that the synchronous-photo-analysis latency target is achievable with the recommended architecture.
3. What is the Contractor Verification System's actual mechanism (licensing lookup, manual review, background check vendor), and how does it verify Trade/Service qualifications specifically (self-attestation vs. license-per-Service validation)?
4. Criteria for defaulting a Homeowner into the Guided Assessment Flow vs. offering the Skip-AI Path (§4.1 FR-2 notes).
5. Tier structure specifics: the 3-Tier shape and Matching Priority boundary are now set (§4.8), but Tier names, price points, and Tier 2/3's undefined "other premium features" remain open. (`addendum.md` notes an AI price estimator as one candidate Tier 2/3 feature.)
6. Numeric targets for SM-1 (conversion rate) and other Success Metrics — none set yet; White Hat discussion in the brainstorm flagged that real market research, not just directional competitive scans, is still needed before committing to numbers.
7. Is the FR-10 reciprocal-trust approach (folding it into existing Lead-Quality Signals) sufficient long-term, or will Contractor feedback post-launch surface a need for a distinct homeowner-reliability feature?
8. Given Thumbtack's 2025-26 AI-diagnosis launch (see `addendum.md`), the platform is committing to AI Diagnosis as core rather than differentiating away from it — what specifically makes this platform's analysis "best-in-class" (accuracy, speed, breadth of detectable issues, confidence calibration)? This needs an answer before the AI vision-analysis spike (Open Question 2) can be scoped.
9. What is the Issue-to-Service mapping algorithm (FR-5) — rules-based keyword/category matching, model-driven classification, or a hybrid? Affects both the AI vision-analysis spike and the Taxonomy's suggested data model in `addendum.md`.
10. The brainstorming session's Red Hat critique raised a distinct risk from diagnostic accuracy/latency: the platform "ships glitchy" or doesn't hold up reliably at production scale (uptime, error handling, graceful degradation under load). This isn't tracked as an NFR or Open Question anywhere else — does it need a dedicated reliability/operational-readiness workstream ahead of launch, separate from the AI vision-analysis spike?
11. Space Visualization (FR-4) needs its own feasibility spike — is this a near-term priority (parallel to the AI vision-analysis spike) or deferred until Phase 1's diagnostic core is proven? What's the minimum viable technical approach (a full generative image model vs. a simpler reference/mood-board matching mechanism)?

## 9. Assumptions Index

- §2.1 — Repeat-usage retention loop for Homeowners is a secondary funnel worth designing for, not a scoped MVP feature.
- §2.2 — DIY-guidance-seeking Homeowners are treated as non-users; this platform routes to a professional rather than replacing one.
- §2.3 UJ-1 — Low-detail photo submissions trigger a re-prompt rather than a low-confidence diagnosis.
- §2.3 UJ-3 — Admin/Dashboard User performs the same queue flow as the Contractor account owner, same mechanism, different seat.
- §2.3 UJ-4 — Space Visualization is additive; a Homeowner can submit a Lead without waiting for or generating one.
- §1 / §8 OQ1 — Phase 1 trade subset provisionally set to Plumbing, Electrical, and HVAC; directional, not a validated decision.
- §3 — Budget-Realism Check is implemented as a single undifferentiated MVP check (education-vs-lowball split deferred).
- §4.2 FR-3 — Condition Assessment output is framed distinctly from AI Diagnosis (assessment vs. damage diagnosis) though the underlying disclaimer principle is the same.
- §4.2 FR-4 — Space Visualization output carries the same trust-preserving disclaimer principle as the Misdiagnosis Disclaimer, applied to a generative rather than diagnostic output; declining or skipping visualization does not block Lead submission.
- §4.3 FR-5 — Leads that can't be confidently mapped to a known Service are routed to manual/fallback categorization rather than dropped or mis-routed.
- §4.4 FR-7 — "Minimum length" for a flagged-low description is not yet numerically defined.
- §4.4 FR-8 — Cold Leads remain viewable in a separate, deprioritized view rather than being deleted outright.
- §4.4 FR-9 — "Significantly below market expectation" is not yet numerically defined; the Budget-Realism Check surfaces a homeowner-facing realism note, not just a silent Contractor-side filter.
- §4.6 — Job Queue Dashboard must be usable on a phone browser, not just desktop.
- §4.6 FR-14 — Emergency-Work-Type Leads trigger an expedited alert distinct from standard new-lead notification.
- §4.7 FR-15 — Every completed Job prompts for actual final cost before closing, to populate the Outcome Log.
- §4.1 — Photo-based AI Diagnosis latency has no numeric target yet; directionally "low seconds, not minutes."
- §7 SM-1 — No numeric conversion-rate target is set yet; framed only directionally against incumbent baselines.
- §7 SM-7 — No numeric AI Diagnosis latency target is set yet.
