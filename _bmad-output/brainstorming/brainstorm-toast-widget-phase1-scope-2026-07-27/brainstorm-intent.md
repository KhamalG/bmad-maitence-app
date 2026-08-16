# Brainstorm Intent: Toast-Style Single-Contractor Widget — Phase 1 Scope

## Context / Purpose

This session revives the **"single-sided widget" framing**, previously documented in the project's `addendum.md` as a **superseded framing** — an embeddable AI intake widget on an individual home-service contractor's own website (Toast/Uber-Eats/DoorDash-style ordering-button model), with **no cross-contractor marketplace matching**.

This reverses the direction of the existing `prd.md` and architecture spine, which were built around a **multi-contractor marketplace-matching model**. Those documents will need **reconciling** against this new single-contractor-widget direction before Phase 1 build.

This doc is a tight, PRD-ready summary of the chosen/critical outputs of this session — not a full report. Feed directly into `bmad-product-brief` or `bmad-prd`.

---

## Scope Parameters — Chosen Values

| Parameter | Chosen Value |
|---|---|
| **Entry point** | (1) Button on contractor's own website, redirecting to our hosted platform for request submission. (2) Button/listing on Google (restaurant-order-button style). (3) Direct discovery via our own platform/app or Google search, independent of the contractor's site — Uber Eats/DoorDash-style. |
| **AI response speed/accuracy boundary** | Homeowner waits at most ~60 seconds for a likely-issue response. This window applies **only to the AI analysis step itself** — checklist/description/photo-video capture happens first, untimed, before the AI clock starts. Speed and accuracy are treated as independently controllable, but the untimed capture step exists specifically so the clock never forces a speed/accuracy tradeoff. |
| **Post-submission contractor flow** | Request routes to the contractor's own dashboard/intake. Contractor can accept+schedule directly, or communicate with the client to gather more info first. All actions happen through the contractor's dashboard. |
| **Capture depth** | Detailed description + photos/videos + a dedicated diagnostic checklist/to-do-list page, then handoff to AI analysis. |
| **Branding/templating** | Contractor-brandable templates: logo, service listings, job details, direct job request/claim, and surfaced customer reviews/feedback from prior jobs. |
| **Monetization/onboarding** | Monthly subscription, multiple contractor tiers. Onboarding mix: self-serve signup + sales-assisted setup. |

### Deferred/Phased Detail
- Discovery-tier crossing: platform-side discovery section would rank higher-tier contractors above others, but **tiering is not needed for MVP** — rolls out post-MVP alongside more features.
- The entry-point option values form a **phased roadmap**: redirect-to-hosted-page → platform-side discovery/directory → tiering-on-discovery (single-contractor widget → fuller platform).

---

## JTBD Findings

**Contractor**
- Functional: Cuts administrative cost — replaces a hired person for finding leads/work, makes admin work more seamless.
- Emotional: Relief from time wasted on dead leads and customers who can't communicate real scope/impact of the work needed.

**Homeowner**
- Functional: Diagnose the issue and get a realistic project cost sense before ever calling.
- Emotional: Lifts overwhelm of calling a contractor without foundational knowledge of the problem.
- **Root fear (insight)**: Being overcharged/taken advantage of due to lacking knowledge of what's fair. The diagnosis+cost-estimate job functionally exists to avoid this feeling — this is the deepest driver behind the homeowner side of the product.

---

## What to Amplify

- **Accuracy of the AI analysis is the core differentiator** — make-or-break, since the platform lacks Toast/DoorDash-level pre-existing user trust/familiarity to lean on.
- **UI polish + contractor's ability to "sell their business"** (branding, service listings, reviews) enriches the platform and drives desirability.
- Platform familiarity itself (recognizable Toast/DoorDash-style flow) lowers customer friction and builds trust/expectations even before brand trust exists.
- Seamless integration into a contractor's existing website + a platform-level directory for discovering other providers is worth expanding on eventually.

---

## Cross-Pollination Mechanics (Borrowed, Adopted)

- **ER-inspired urgency tagging**: visible urgency tags on every job (like ER triage levels), shown to both contractor and homeowner. AI analysis itself determines urgency level, not just diagnosis content.
- **ER-inspired dynamic queue reorder**: contractor's job queue dynamically re-orders as urgent requests come in — not a static emergency flag.
- **Casino-inspired vibe**: overall app look/feel functions as a comfort-and-engagement lever independent of analysis content.
- **Casino-inspired post-diagnosis social proof**: after AI names the likely job, show examples of that same job type plus reviews from other customers (contractor-permitting) — before the homeowner talks to the contractor.

---

## Closing Synthesis / Insights

- **Center of gravity**: (1) platform integration/entry-point model, and (2) the AI analysis engine. Everything else (branding, tiering, dashboard) hangs off these two.
- Speed-boundary + accuracy-protection is **one design principle**, not two separate decisions: untimed capture exists so the AI clock never trades accuracy for speed.
- AI-determined urgency tags and post-diagnosis social proof were **each independently invented by two different techniques** — signal they are structural, not nice-to-haves.
- The entry-point parameter's option values already form the phased roadmap from single-contractor widget to fuller marketplace platform.
- **Named next step**: research how the AI engine actually works before locking Phase 1 scope — using `addendum.md`'s existing Rekognition/Bedrock/SageMaker direction as a **starting point** to validate against the narrower single-contractor scope, not from a blank page.

---

## Open Questions / Not Yet Decided

- Contractor tiering structure and features — explicitly deferred to post-MVP.
- AI engine mechanics/architecture — needs dedicated research (per named next step) before Phase 1 scope can be locked; existing `addendum.md` AWS direction (Rekognition/Bedrock/SageMaker) is a starting point, not validated for this narrower scope.
- Reconciliation of existing `prd.md` and architecture spine (built for the multi-contractor marketplace-matching model) against this single-contractor widget direction — not addressed in this session.
