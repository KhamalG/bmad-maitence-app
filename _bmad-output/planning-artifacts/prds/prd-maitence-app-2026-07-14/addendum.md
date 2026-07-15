# Addendum: AI-Powered Lead Intake Platform for Contractors

Supporting material for `prd.md` — technical direction, rejected alternatives, and research detail that inform the PRD's decisions but don't belong in a capabilities-focused document.

## Property Services Taxonomy — Data Model

Source: [`docs/Property_Trades_Taxonomy.md`](../../../../docs/Property_Trades_Taxonomy.md), supplied 2026-07-14. Defines the long-term multi-trade catalog (~20 Trade categories from General Contracting to Pest Control, each with a set of Services carrying a Work Type: Repair / Maintenance / Improvement / Inspection / Emergency) and a suggested relational schema:

```text
Trades(TradeId, Name)
Services(ServiceId, TradeId, Name, WorkType)
Contractors(ContractorId, Company, License)
ContractorTrades(ContractorId, TradeId)
ContractorServices(ContractorId, ServiceId)
```

This is the backing data model for PRD §4.3 (Issue-to-Service Matching, FR-5/FR-6): a Contractor belongs to multiple Trades and offers multiple Services; the AI Diagnosis (or Skip-AI Path description) maps a Lead's issue to a Service, which filters to qualified Contractors via `ContractorServices`.

Phase 1 trade/service selection (PRD §8 Open Question 1) is a subset of this catalog. The PRD now carries **Plumbing, Electrical, and HVAC** as a provisional `[ASSUMPTION]` — drawn from the brainstorming session's home-repair framing (leaking pipes, breakers, HVAC units) — but Service-level granularity within each Trade is still unresolved, and the provisional choice itself hasn't been validated against real market/demand data.

## AI Vision-Analysis Spike — Technical Direction

Source: [`AI_Photo_Video_Analysis_Guide.md`](../../../../docs/AI_Photo_Video_Analysis_Guide.md), supplied 2026-07-14. Addresses PRD §8 Open Question 2 (spike mechanism) and grounds the photo-sync/video-async decision behind PRD §4.1's Feature-specific NFRs.

**Recommended architecture:** backend-centric, not on-device. Mobile app (Swift/Kotlin, per Tech Stack Direction below) handles capture and upload only; a backend API orchestrates media storage, AI provider calls, and result persistence, returning a unified response to the app. Rationale: keeps the app lightweight, keeps API keys server-side, allows swapping AI providers without app updates, and allows queuing/retrying long-running video analysis.

**Service mapping (AWS-native, consistent with the Hosting: AWS decision below):**
- **Amazon Rekognition** — structured computer vision (object detection, text/OCR, scene classification, custom labels). Good fit for a first pass at identifying visible damage/issue types.
- **Amazon Bedrock** (or another multimodal LLM, e.g. GPT-5.5 Vision, Gemini, Claude) — higher-level reasoning: summarizing the scene, answering "what's likely wrong," generating the AI Diagnosis narrative. On Bedrock specifically, video support is via extracted frames only, not native video reasoning.
- **Amazon Textract / Transcribe** — available if OCR or spoken description transcription becomes relevant (e.g. a Homeowner narrates the issue on video, or OCR reads a model/serial plate off equipment in a photo to sharpen diagnosis specificity).
- **Amazon SageMaker** — a candidate lever for the "best-in-class AI Diagnosis" differentiation goal (PRD §1, §8 Open Question 8) if Rekognition + a general-purpose multimodal LLM prove insufficiently accurate for the Phase 1 trade subset — supports training industry-specific custom vision models rather than relying solely on general-purpose services. Not committed; a candidate for the spike to evaluate, not a decision.

**Sync vs. async — this is the load-bearing constraint behind the PRD's photo-real-time / video-async split:**
- Images: typically synchronous, 1–5 second response times industry-wide. This is what makes FR-1's "no waiting around" requirement achievable for the common photo-only case.
- Video: better suited to async processing (upload → job ID → backend processes → poll or push notification). Native multimodal video reasoning is still limited (Bedrock explicitly handles video via extracted frames, not full native understanding); treating video as an async enrichment rather than a blocking step in the Guided Assessment Flow sidesteps this limitation rather than fighting it.

**Cost/scale levers noted in the source guide, relevant to future NFR work:** resize/compress media before analysis, cache repeated results, route to lower-cost models where full reasoning isn't needed, process video asynchronously with retry/queue support (API Gateway → Lambda/ECS/EKS → S3 → Rekognition/Bedrock → database, in an AWS-native build).

**Still open** (per PRD §8 Open Question 2 and Open Question 9): the specific Issue-to-Service mapping algorithm, and numeric latency/accuracy targets that would make the AI Diagnosis "best-in-class" rather than merely functional.

## Tech Stack Direction

Locked during brainstorming as a Must-have (core tech stack decisions):

- **Hosting:** AWS
- **Mobile:** Swift (iOS), Kotlin (Android)
- **Backend APIs:** TypeScript
- **AI implementation:** Python
- **Website:** Angular

This is a starting direction, not yet validated by the AI vision-analysis research spike (PRD §6.1, Open Question 2) — the spike may surface constraints that affect the AI implementation stack specifically.

## Rejected / Deferred Alternative: Full AI Auto-Assignment

Explored via a "What If" prompt during brainstorming: if the AI trust gap were fully solved, the platform could have AI auto-select which jobs a Contractor should take and auto-assign/schedule them, rather than surfacing Leads for manual review and acceptance (as scoped in PRD §4.6).

**Why set aside:** explicitly framed as unlocked-only-if-trust-were-solved. Trust in the AI Diagnosis is the platform's core unresolved risk (see PRD §7 SM-C2), so removing the human-in-the-loop acceptance step now would compound that risk rather than reduce it. Revisit only after the platform has track record showing AI Diagnosis accuracy holds up at scale.

## Budget-Realism Check: Two-Cause Mechanism (Deferred Design)

The brainstorming session split "unrealistic budget" into two distinct root causes, each requiring different handling:

- **Homeowner doesn't know market pricing** → should be *educated* (e.g. shown a market-rate range before submitting).
- **Homeowner is intentionally lowballing** → a genuine bad-fit signal that should be surfaced to the Contractor as such.

PRD §4.4 FR-9 ships a single undifferentiated check for MVP. When this is revisited, the design fork above is the starting point — likely requiring either a pre-submission pricing-education step (for cause 1) distinct from a post-submission flag (for cause 2), rather than one check trying to serve both purposes.

## Persona Depth: Admin/Dashboard User

Surfaced as a distinct persona from "the Contractor" during a role-playing exercise — the person actually working the dashboard day to day, which may or may not be the Contractor account owner (see PRD §2.1, §6.2 tier-gated Admin seats).

Two distinct UX failure modes were named for this persona, not one general "dashboard is bad" concern:
1. **Information overload** on the card/detail view.
2. **Too many clicks** to accept a job.

PRD §4.6 (FR-12, FR-13) addresses these as two separate requirements rather than folding them into a single "good UX" statement — keep them separate in downstream UX/architecture work too.

## Market & Competitive Landscape Research (2026-07-14)

Research conducted to ground PRD §7 and §8 Open Question 8. Not exhaustive — flagged in the brainstorming session's own White Hat critique as an area needing real market research beyond this directional scan.

**Major incumbents (lead-gen marketplaces):**
- **Angi** (formerly HomeAdvisor + Angi Leads, merged 2023): ~$287–500 annual membership *plus* $15–120 per lead, billed regardless of contact/conversion. Leads shared across 3–5 competing pros.
- **Thumbtack**: pay-per-lead-response, $8–150+ per contact, no mandatory annual fee.
- **Porch**: largely exited pure lead-gen, pivoted to insurance (~67% of revenue).
- Shared theme: monetize via pay-per-lead or membership+lead-fee, leads typically shared across multiple competing pros — the structural driver of the complaints below.

**Direct competitive signal — AI diagnosis-before-match (important, not whitespace):**
- **Thumbtack launched an AI diagnostic flow in 2025-26**: homeowners describe an issue via text/photo/voice; AI interprets and recommends matched pros. Thumbtack reports 87% of users found photo/voice input valuable and higher confidence vs. traditional search.
- **Angi's "AI Helper"** is an AI-assisted booking concierge — assumes the homeowner already knows the problem, not true diagnostic triage. This remains a differentiation gap this platform can target.
- **Smaller AI-native entrants**: fixRAgent (diagnosis + parts/step guides + property tracking), AptRepair (BYU-built diagnostic + cost estimate + contractor connection), FiXA (Philly, March 2026, DIY-or-hire diagnosis). Early-stage, unproven, unclear trade-scope or monetization.
- **Takeaway:** frame differentiation as superior lead-quality/exclusivity + contractor-side affordability (PRD decision, §8 Open Question 8), not "first AI diagnosis" — that claim no longer holds.

**Contractor-side CRM/lead-management tools (competing for the same budget):**
- **Jobber**: $49–349/mo, best fit for 1–10 person shops.
- **Housecall Pro**: $65–450/mo per user, strong marketing automation/review-request features.
- **ServiceTitan**: enterprise, $500–800/seat/mo + $5K–20K onboarding, targets 10+ tech operations.
- These are not direct competitors for lead-gen, but compete for the same Contractor budget/attention — differentiation must be lead-intake/admin-replacement, not full field-service management (out of scope per PRD §5 Non-Goals framing).

**Pain points validating the differentiation thesis:**
- Widespread contractor complaints that Angi/HomeAdvisor leads are fake, unanswered, or price-shoppers-only, while still billed per lead. BBB "F" rating, 1,800+ complaints in 3 years; Trustpilot 2.1/5; ConsumerAffairs 1.4/5.
- FTC fined HomeAdvisor up to $7.2M in 2023 for deceptively overstating lead-to-job conversion rates.
- Root cause is structural: shared/duplicated leads sold to multiple competing pros regardless of homeowner intent — exactly the gap a pre-qualified, single-match lead model (PRD §1 Vision) targets.

## Superseded Framing: Single-Sided Widget Model (`Ai Lead Intake.md`)

Source: `docs/Ai Lead Intake.md`, found 2026-07-14 in `docs/` but never previously discussed in this PRD's working sessions. It describes a materially different Phase 1 architecture — an embeddable AI intake widget on an individual Contractor's own website, with no platform-side matching, verification-for-matching, or tiered ranking (it explicitly frames avoiding two-sided-marketplace complexity as a deliberate simplification, with marketplace/consumer-facing expansion as a later phase).

**Resolution:** the user confirmed the marketplace model already scoped in `prd.md` (§4.3 Issue-to-Service Matching, §4.5 Verification, §4.8 Tier-Based Matching Priority) is the current direction. This document is treated as an earlier or superseded framing, not adopted as-is. It should not be used to justify removing platform-side matching from Phase 1 without an explicit new decision.

**Salvaged ideas, non-conflicting with the marketplace model:**
- **Broader data-flywheel field set.** The document envisions the Outcome Log-style dataset (PRD §4.7 FR-15) eventually spanning pricing, job types, service areas, project duration, availability, and seasonality — not just final cost. Worth considering as the Self-Improving Pricing Engine (FR-16) matures past MVP, even though FR-15 itself stays narrowly scoped to cost/outcome for now.
- **AI price estimator as a candidate Tier 2/3 premium feature.** The document's own caution against showing homeowners an AI-generated price they might treat as a guaranteed quote (a specific instance of the misdiagnosis-trust risk this PRD already treats as central) led it to suggest gating a price estimator as a beta / premium-tier feature. This is a concrete candidate to help fill the "TBD premium features" gap in PRD §4.7 / Open Question 5, worth surfacing to whoever resolves that question next — but not decided here.

## Aspirational Framing (Brand/Tone Input)

From the brainstorming session's Yellow Hat (2-year best case) exercise — not decisions, just raw material for future brand/voice work:
- "The default first-thought platform for home service needs" in a region (regional-dominance framing).
- "Proves LLM-driven analysis can meaningfully improve people's lives" (mission-level framing).

No concrete brand voice/tone direction exists beyond this — flagged as a Could-have (brand/marketing moat strategy) in the PRD, not yet developed.
