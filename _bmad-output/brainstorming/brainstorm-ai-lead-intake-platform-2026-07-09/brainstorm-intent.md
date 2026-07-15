# Project Intent: AI-Powered Lead Intake Platform for Contractors

## Core Concept
A platform that uses AI photo/video analysis to pre-diagnose home service problems, matching homeowners to trustworthy contractors while giving contractors an affordable virtual admin layer to manage inbound leads. Phase 1 targets a defined subset of trades (multi-trade expansion is a later phase).

## Core Tension to Solve
The central trust-gap risk: homeowners fear AI misdiagnosis or a wide gap between the AI's preliminary estimate and the contractor's final quote, which erodes trust in the whole platform. The hybrid intake flow addresses this directly — new/unsure users get the full guided AI-assessment (with a clear misdiagnosis disclaimer), while returning/knowledgeable users can skip straight past AI diagnosis to book the service they already know they need. This protects vulnerable users from a bad AI guess without forcing friction on users who don't need it.

## Deepest Job to Be Done
- **Homeowner side:** hired to relieve the anxiety of not knowing what's wrong or who to trust — the job is "give me confidence and insight to book with certainty," not merely "get a quote."
- **Contractor side:** hired to replace the cost of dedicated intake/admin staff at a fraction of the price, but the deepest layer is alleviating the overwhelm of running a business — AI-driven insight should actively help drive revenue, not just cut admin overhead.
- Both threads converge on one mechanism: card-queue/low-click dashboard is the literal delivery vehicle for "relieve overwhelm," and job-outcome logging (the data flywheel) is the shared feedback loop that both tightens estimate accuracy and improves lead-quality filtering. Outcome-logging should be wired up early even though the full self-improving pricing engine stays a Should.

## MVP Scope (MoSCoW)

### Must Have
- AI misdiagnosis disclaimer shown to homeowners
- Hybrid skip-AI-diagnosis path (alongside full guided AI-assessment flow)
- Lead-quality signals: detail/completeness of submission (photos, video, description depth) and budget-realism check
- Cold-lead filtering for contractors
- Contractor legitimacy/verification system
- Card-based job queue dashboard with expandable detail view
- Low-click job acceptance flow
- Core tech stack decisions locked (see Tech Stack Direction)
- AI vision-analysis research spike (resolve the biggest technical unknown before further build-out)

### Should Have
- Self-improving pricing engine: learns from a contractor's historical job costs/materials to auto-generate more accurate estimates over time
- Tiered pricing model: lower tiers = contractor self-manages dashboard; higher tiers = dedicated admin staff + premium dashboard features (tier details deferred)
- Revenue-driving analytics/insights in the dashboard (not just admin-cost reduction)
- Contact-reveal event tracking: logs when a homeowner calls/contacts a contractor directly, keeping the contractor anchored to the platform for job tracking/management regardless of contact method

### Could Have
- Brand/marketing moat strategy to differentiate against copycats (positioning and creative differentiation rather than relying on tech alone)

### Won't Have (this time)
- Multi-trade expansion beyond the Phase 1 trade subset
- 10-year local-services vision (barbers, stylists, mechanics, etc.)
- 100-year full-automation speculation (AI autonomously fulfilling the underlying need)

## Tech Stack Direction
- Hosting: AWS
- Mobile: Swift (iOS), Kotlin (Android)
- Backend APIs: TypeScript
- AI implementation: Python
- Website: Angular

## Business Model Notes
Homeowners are free users. Monetization is entirely contractor-side via tiered pricing (self-serve dashboard at lower tiers, dedicated admin support at premium tiers). Specific tier structure and pricing are deferred beyond MVP.

## Key Open Risks
- Competitor copycat risk: concept could be replicated once proven, eroding first-mover advantage
- Bypass-the-platform risk: users may abandon the full intake flow and contact contractors directly, skipping the platform (mitigated via contact-reveal event tracking, a Should)
- Contractor gaming risk: fake reviews, self-referrals, or manipulated lead/job data require a legitimacy/verification system
- AI analysis technical unknown: how the photo/video AI diagnosis will actually work in practice is unresolved and must be validated via a dedicated spike before further build-out

## Immediate Next Step
Nail down system architecture and define the detailed MVP feature set. Target: MVP live within 6 months.
