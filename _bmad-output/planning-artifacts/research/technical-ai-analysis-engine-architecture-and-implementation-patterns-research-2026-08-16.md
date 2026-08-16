---
stepsCompleted: [1, 2, 3, 4, 5, 6]
inputDocuments:
  - docs/compass_artifact_wf-3a6d58f1-d775-5a82-8ebe-9b7ec47061ee_text_markdown.md
  - docs/gemini-vision-analysis-roadmap.md
  - _bmad-output/brainstorming/brainstorm-toast-widget-phase1-scope-2026-07-27/brainstorm-intent.md
workflowType: "research"
lastStep: 6
research_type: "technical"
research_topic: "AI Analysis Engine Implementation Architecture — Phase 1 (Gemini-based vision/video diagnostic pipeline for the single-contractor widget)"
research_goals: "Get a concrete, implementation-ready grasp of: the end-to-end architecture (orchestrator, service boundaries, data flow from photo/video capture through structured diagnosis to contractor dashboard); implementation patterns for the sync photo path (≤60s) vs. the async video path (File API + webhook); structured-output/schema design for diagnosis (issue, confidence, ER-style urgency tag, homeowner narrative); outcome-logging pipeline feeding future fine-tuning; failure handling, fallbacks, and hybrid-routing patterns as volume grows; and how this sits inside an AWS-native backend calling out to Google Gemini (auth, latency, quota)."
user_name: "Khamal"
date: "2026-08-16"
web_research_enabled: true
source_verification: true
---

# From Photo to Diagnosis in 60 Seconds: The Gemini-Powered AI Analysis Engine Architecture

## Executive Summary

A homeowner points a phone camera at a leaking pipe or a cracked panel, and within 60 seconds gets a structured, ER-style triage: what's likely wrong, how confident the system is, how urgent it is, and a plain-language explanation. That promise is the product; this report is the engineering blueprint for keeping it. It picks up where two prior documents left off — the vendor/cost research that chose Gemini multimodal, and the roadmap gap-check that sketched a phased build — and answers the question neither fully resolved: what does the implementation actually look like, end to end, verified against current public documentation rather than assumption.

The core finding is that almost nothing here requires inventing new patterns. The sync photo path is a "Direct Lambda to LLM" call with inline base64 image data (now up to 100MB per request); the async video path is a File API upload feeding a signed, idempotent webhook pipeline; the diagnosis itself is Gemini's guaranteed-shape `responseSchema` JSON, not a hopeful prompt instruction. What _is_ novel to get right is the seam around these well-trodden pieces: a circuit breaker and bounded retry around the one dependency (Gemini) that sits outside AWS's network and could otherwise eat the entire 60-second budget on a single degraded call; a queue-based leveling layer between the webhook receiver and the video-processing worker so bursty homeowner traffic never becomes a bursty backend traffic pattern; and a hexagonal (ports-and-adapters) boundary around the model call so that neither a second vision vendor nor the Phase 4 hybrid-routing cascade requires reworking the orchestrator's core logic.

**Key Technical Findings:**

- Gemini's `responseSchema` + `responseMimeType: "application/json"` together _guarantee_ schema-shaped output — this is the single implementation detail the entire diagnosis engine hinges on, and it's confirmed current, not a prompt-engineering hope.
- The photo path and video path are architecturally different problems (inline synchronous call vs. File API + signed webhook + queue), and treating them identically would misapply either the 60-second budget or the decoupling the video path needs.
- Circuit Breaker + Retry + Queue-Based Load Leveling are the three resilience patterns that matter most, and they map onto three distinct places in the architecture — not one generic "add retries everywhere" policy.
- Hexagonal architecture around the Gemini call is what makes the two-vendor posture and the Phase 4 cost-optimization cascade cheap to add later instead of a rewrite.
- Cost has two concrete, already-available levers before any custom optimization work: Gemini's automatic implicit context caching, and routing non-latency-sensitive workloads (fine-tuning dataset prep) through Batch/Flex inference tiers.

**Technical Recommendations:**

1. Build the orchestrator behind a ports-and-adapters boundary from day one — even in Phase 1, before a second vendor or hybrid routing exists — so those additions are adapter swaps, not rewrites.
2. Wrap the Gemini call in a circuit breaker layered with bounded retry, tuned specifically against the 60-second ceiling, not a generic retry-everything policy.
3. Treat the video path as a queue-based load-leveling problem from the start: signature/timestamp-verified webhook receiver → SQS → idempotent worker → S3-then-DynamoDB write order.
4. Adopt AWS SAM for infrastructure-as-code and local testing given the pure-serverless stack, with Step Functions deliberately deferred to Phase 4's cascade logic rather than introduced early.
5. Wire circuit-breaker state and SQS queue depth into CloudWatch alarms as the two earliest operational signals of Gemini-side degradation or volume outgrowing Phase 1 assumptions.

## Table of Contents

1. Technical Research Introduction and Methodology
2. AI Analysis Engine Technical Landscape and Architecture Analysis
3. Implementation Approaches and Best Practices
4. Technology Stack Evolution and Current Trends
5. Integration and Interoperability Patterns
6. Performance and Scalability Analysis
7. Security and Compliance Considerations
8. Strategic Technical Recommendations
9. Implementation Roadmap and Risk Assessment
10. Future Technical Outlook and Innovation Opportunities
11. Technical Research Methodology and Source Verification
12. Technical Appendices and Reference Materials

## 1. Technical Research Introduction and Methodology

### Technical Research Significance

An AI diagnosis engine that gets the architecture wrong doesn't fail loudly — it fails as a slow, occasionally-wrong widget that homeowners quietly stop trusting. The 60-second sync boundary and the "accuracy is the core differentiator" positioning (from the originating brainstorm intent) make the plumbing around the AI call — not just the AI call itself — the thing that determines whether the product feels trustworthy. This research exists to close the gap between "we picked Gemini" and "here is exactly how the request/response, failure-handling, and data-logging flow works," verified against documentation current as of August 2026 rather than general LLM-integration folklore.

_Technical Importance: the difference between a generically "AI-powered" widget and one that reliably hits a hard latency ceiling while handling third-party outages gracefully is almost entirely in the integration and resilience patterns, not the model choice._
_Business Impact: every pattern recommended here (circuit breaker, queue leveling, hexagonal boundary) is chosen specifically because it keeps a Gemini-side problem from becoming a homeowner-visible outage — directly protecting the trust the product is positioned on._

### Technical Research Methodology

- **Technical Scope**: orchestration architecture, structured-output/schema design, sync vs. async request patterns, integration and API design, resilience/architectural patterns, and implementation/adoption strategy — spanning six research passes (scope confirmation → technology stack → integration patterns → architectural patterns → implementation research → this synthesis).
- **Data Sources**: primary vendor documentation (Google Gemini API docs, Google Cloud/Vertex AI docs, AWS documentation and architecture blog), established pattern references (Azure Architecture Center, Alistair Cockburn's original Hexagonal Architecture paper), and current third-party technical analysis (webhook security practice via Svix, LLM cascade routing research).
- **Analysis Framework**: each technical claim was checked against live, current documentation rather than relying on general model knowledge, with explicit callouts where prior research documents' assumptions were confirmed, updated, or sharpened.
- **Time Period**: current as of August 2026 — several confirmed details are recent changes (e.g., Gemini's January 2026 inline-data limit increase to 100MB, the May 2026 webhook launch, Gemini Pro's removal from the free tier in April 2026).
- **Technical Depth**: implementation-ready — schema field names, header names, rate-limit tiers, and specific AWS/Gemini service boundaries, not high-level conceptual coverage.

### Technical Research Goals and Objectives

**Original Technical Goals:** a concrete, implementation-ready grasp of the end-to-end architecture (orchestrator, service boundaries, data flow); sync photo path vs. async video path implementation patterns; structured-output schema design; outcome-logging for fine-tuning; failure handling and hybrid-routing patterns as volume grows; and AWS-to-Gemini integration mechanics (auth, latency, quota).

**Achieved Technical Objectives:**

- The orchestrator's shape (API Gateway → Lambda → Gemini, "Direct Lambda to LLM" pattern) is confirmed and detailed, including the specific ways it differs from an in-VPC AWS service call (auth, timeout budget, cold-start stacking).
- The sync/async split is now a fully specified pair of request patterns: inline-base64 synchronous calls for photos, File API + signed/idempotent webhook + queue for video.
- The diagnosis schema design is grounded in Gemini's actual `responseSchema` mechanism, including two Gemini-specific pitfalls (`propertyOrdering`, partial JSON Schema support) not obvious from generic LLM knowledge.
- Failure handling and hybrid routing now have named, well-understood patterns (Circuit Breaker, Retry, Queue-Based Load Leveling, cascade/model-routing) rather than being designed from scratch.
- **Additional insight discovered beyond the original goals**: the ports-and-adapters (hexagonal) architectural boundary emerged as the connective pattern that makes several originally-separate goals (vendor flexibility, hybrid routing, testability) cheap to satisfy simultaneously, rather than requiring separate solutions for each.

## 2. AI Analysis Engine Technical Landscape and Architecture Analysis

### Current Technical Architecture Patterns

The dominant, directly-reusable shape for this system is **API Gateway → Lambda → external LLM API**, which AWS's own guidance frames as the "Direct Lambda to LLM Pattern" — the same pattern used for Bedrock, just with the outbound call pointed at Gemini instead. The one architectural wrinkle that matters throughout this research: this is a call to **Google's infrastructure, not an in-VPC AWS service call**, which carries a distinct auth, latency, and quota profile. This shows up in three concrete orchestrator decisions: **auth** (API key for Phase 1 vs. a Google Cloud service account once Vertex AI fine-tuning arrives in Phase 3 — affects Secrets Manager vs. mounted-key secrets design), **timeout budget** (Lambda timeout + Google egress latency must fit inside the 60-second ceiling with retry margin), and **cold-start stacking** (a cold Lambda plus a cold outbound HTTPS connection to a third-party host behaves differently under load than warm in-VPC AWS calls, and should be load-tested specifically against the 60s ceiling).

Layered on top of this base pattern are the three resilience patterns that define the architecture's actual robustness: **Circuit Breaker** around the Gemini call (fails fast once Gemini is detected as degraded, rather than letting every concurrent request individually burn through a timeout), **Retry** for genuinely transient faults (used _through_ the breaker, not instead of it — retry while Closed, respect Open as a stop signal), and **Queue-Based Load Leveling** for the video path (an SQS buffer between the webhook receiver and the processing worker, so bursty submission volume never directly hits downstream S3/DynamoDB writes at a bursty rate).

_Dominant Patterns: Direct Lambda-to-LLM base pattern + Circuit Breaker/Retry/Queue-Leveling resilience layer, not a single monolithic pattern._
_Architectural Trade-offs: an overly aggressive circuit-breaker threshold degrades service during brief blips; too lenient a threshold lets a genuinely down Gemini API continue consuming Lambda concurrency and cost on every request._
_Source: [AWS Compute Blog — Designing serverless integration patterns for LLMs](https://aws.amazon.com/blogs/compute/designing-serverless-integration-patterns-for-large-language-models-llms/)_
_Source: [Azure Architecture Center — Circuit Breaker pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)_

### System Design Principles and Best Practices

**Hexagonal architecture (ports and adapters)**, from Alistair Cockburn's original 2005 formulation, is the design principle that ties the rest of the architecture together: the diagnosis logic (validate input, build the prompt, apply the schema, interpret the result) sits behind a port, with "call Gemini" as just today's adapter — not hard-wired throughout the codebase. This single decision is what makes three otherwise-separate goals cheap simultaneously: the two-vendor posture (a second vision vendor becomes a new adapter), the Phase 4 hybrid-routing cascade (cheap/premium model routing becomes adapter selection logic), and unit testability (business logic tested against a mock adapter with zero network calls or Gemini billing).

_Design Principles: the circuit breaker and retry logic wrap the adapter itself, not the core business logic — keeping the resilience layer and the domain logic cleanly separated along the same boundary._
_Source: [Alistair Cockburn — Hexagonal Architecture (original 2005 article)](https://alistair.cockburn.us/hexagonal-architecture/)_

## 3. Implementation Approaches and Best Practices

### Current Implementation Methodologies

The structured-output mechanism is the single most load-bearing implementation detail: setting both `responseMimeType: "application/json"` **and** `responseSchema` together _guarantees_ the response parses and matches the target structure — no markdown-fence stripping, no regex extraction, no malformed-JSON retry logic. Setting `responseMimeType` alone only nudges the model and is a common implementation mistake. The diagnosis shape (`{ likely_issue, confidence, urgency_tag, narrative }` per trade) becomes a literal `responseSchema` object enforced at generation time. Two Gemini-specific pitfalls to design around from day one: `propertyOrdering` (a Gemini-specific hint that fixes emitted field order and can measurably affect output quality/consistency, unlike standard JSON Schema which ignores key order), and **partial JSON Schema support** (very large or deeply nested schemas can be silently rejected or have keywords dropped — generate a TypeScript/Python type from the schema so backend code and the model contract can't silently drift).

For the **photo path**, inline base64 data (not the File API) is correct — Gemini raised its inline-data limit to 100MB per request in January 2026, comfortably covering any homeowner photo without upload/poll indirection. For the **video path**, the File API + webhook combination is confirmed with concrete shape from Google's May 2026 launch materials: **HTTPS Gateway → Signature Verification → immediate 2xx → Message Queue → Backend Worker**, with the receiver's only job being fast verify-and-acknowledge, never inline result-processing. File API uploads expire after 48 hours, so the durable copy of any submitted media must be persisted to S3 independently, never relying on the File API copy as the record of truth.

_Development Approaches: schema-first diagnosis design, with the schema treated as a strict contract validated by generated types, not a loose prompt hint._
_Code Organization Patterns: ports-and-adapters boundary around the model call, resilience patterns (breaker/retry) wrapping the adapter, not the domain logic._
_Source: [Structured outputs | Gemini API](https://ai.google.dev/gemini-api/docs/structured-output)_
_Source: [Image understanding | Gemini API](https://ai.google.dev/gemini-api/docs/image-understanding)_
_Source: [Reduce friction and latency for long-running jobs with Webhooks in Gemini API — Google Blog](https://blog.google/innovation-and-ai/technology/developers-tools/event-driven-webhooks/)_

### Implementation Framework and Tooling

**AWS SAM** is the recommended IaC entry point given a pure Lambda + API Gateway + DynamoDB + SQS + S3 stack with no legacy migration involved — its CLI (`sam init`, `sam local`, `sam sync`) supports local Lambda/API Gateway testing before every deploy, directly useful for iterating on the orchestrator's timeout/retry/breaker logic before it's live. SAM and CDK aren't mutually exclusive; SAM's local-testing features can layer onto a CDK app if the team later prefers CDK's programmatic constructs for Phase 4's more complex Step Functions orchestration. Three test tiers map onto the ports-and-adapters boundary: unit tests against a mock Gemini adapter (every commit, zero network/billing), local integration tests via `sam local` (Lambda/API Gateway/DynamoDB wiring), and staging smoke tests against the real Gemini adapter (catching schema drift Gemini might silently introduce).

_Build and Deployment Systems: SAM templates generate the CloudFormation needed for staging/production as a pipeline stage, not a manual step; rollback on a bad deploy is a standard stack-rollback operation._
_Source: [AWS — What is the AWS Serverless Application Model (AWS SAM)?](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)_

## 4. Technology Stack Evolution and Current Trends

### Current Technology Stack Landscape

**Orchestration**: Lambda (Node.js or Python — both have first-class official Gemini SDKs: `@google/genai`, `google-genai`) behind API Gateway, calling out to Gemini. **Storage**: S3 for durable originals and the outcome-log dataset (required, since File API's 48-hour expiry rules it out as a durable store); DynamoDB for webhook idempotency-key tracking (TTL ≥ 24 hours) and the structured diagnosis record keyed by request ID; GCS enters only at Phase 3 as Vertex AI's required fine-tuning data source, fed from the S3 pipeline rather than a parallel primary store. **Fine-tuning**: Vertex AI supervised fine-tuning via `sft.train()` against a JSONL image/diagnosis training set (100–500 examples minimum), monitored by job-state polling rather than a webhook — an acceptable pattern here since it's an offline batch job, not a latency-sensitive path.

_Database and Storage Technologies: S3 (durable originals + outcome logs) / DynamoDB (idempotency + diagnosis records) / GCS (Phase-3-only fine-tuning export) — three stores with clearly separated responsibilities, not overlapping choices._
_Source: [Fine-Tuning Gemini on Vertex AI: A Practical End-to-End Guide](https://medium.com/@mdvikranth/fine-tuning-gemini-on-vertex-ai-a-practical-end-to-end-guide-3744bbf5c4db)_
_Source: [Tune Gemini models with supervised fine-tuning | Google Cloud Docs](https://cloud.google.com/vertex-ai/generative-ai/docs/models/gemini-use-supervised-tuning)_

### Technology Adoption Patterns

Three adoption trends confirmed across this research: **webhooks over polling** as the current direction of travel (Gemini's own May 2026 webhook launch, general LLM-infra practice — polling now reserved for genuinely offline jobs like fine-tune status); **guaranteed-schema structured output as standard practice**, not just a Gemini quirk (mirrored by OpenAI and Claude per current comparisons — meaning the schema-first design here is portable to a second vendor); and **model cascading/routing maturing into a named architecture pattern** with published tuning heuristics (a well-calibrated cascade typically escalates 15–35% of queries to the premium tier — above ~50% suggests too-conservative a threshold, below ~5% suggests under-escalation risk on a product where accuracy is the core differentiator).

Quota planning is directionally stable but tiered: free tier ≈ 5–15 RPM (Gemini Pro removed from free tier entirely in April 2026); Tier 1 paid ≈ 150–300 RPM; Tier 2 (after $250 cumulative spend) ≈ 1,000+ RPM; Tier 3 enterprise, 4,000+ RPM — enforced **per Google Cloud project, not per API key**. At Phase 1 volumes (0–5,000 diagnoses/month), even Tier 1 has generous headroom; this becomes a real capacity input only as volume approaches the hybrid-routing threshold (~10,000–20,000/month).

_Adoption Trends: cascading/routing and guaranteed structured output are both now mainstream enough to treat as standard building blocks rather than novel designs to prototype from scratch._
_Source: [Rate limits | Gemini API](https://ai.google.dev/gemini-api/docs/rate-limits)_
_Source: [Dynamic Model Routing and Cascading for Efficient LLM Inference: A Survey](https://arxiv.org/html/2603.04445v2)_

## 5. Integration and Interoperability Patterns

### Current Integration Approaches

The widget's public API needs exactly two operations, not a general REST resource model: a **synchronous endpoint** for the photo path (blocking, returns the structured diagnosis directly within the 60s budget) and an **asynchronous submission endpoint** for the video path (returns a job ID immediately, with the result delivered via the File API + webhook flow and pushed to the contractor dashboard). REST-over-HTTPS is the right shape — no relational query need justifies GraphQL, and the two-operation surface doesn't need gRPC's complexity. Critically, **the widget never receives Gemini's webhook directly** — the backend's own webhook receiver is the sole Gemini-facing endpoint, and the widget instead polls or subscribes to the backend's own job-status endpoint, keeping Gemini's webhook signing secret entirely server-side.

For the Phase 4 hybrid-routing cascade specifically, the multi-call, branching-logic shape is structurally a **prompt-chaining** problem, and AWS's own guidance recommends **Step Functions** over additional Lambda-to-Lambda calls once that shape emerges — declarative escalation branches, built-in retry/backoff/jitter (useful against the RPM ceilings above), and avoiding Lambda's 15-minute timeout ceiling stacking across chained calls. Worth designing for now even though Phase 1 doesn't need it: keep "call model, get result" logic in a unit a Step Functions state can invoke directly, rather than baking branching logic into one monolithic handler.

_API Design Patterns: two-operation REST surface (sync photo, async video-job) instead of a general resource model; Step Functions reserved for Phase 4, not introduced prematurely._
_Service Integration: webhook receiver is the sole Gemini-facing endpoint; the widget only ever talks to the first-party backend._
_Source: [AWS Compute Blog — Designing serverless integration patterns for LLMs](https://aws.amazon.com/blogs/compute/designing-serverless-integration-patterns-for-large-language-models-llms/)_

### Interoperability Standards and Protocols

JSON is the sole data format needed across every boundary in this system — widget↔backend, backend↔Gemini (`responseSchema`-enforced), and backend↔webhook receiver — with no case in current scope for binary serialization (Protobuf/gRPC); the payload shapes and volumes don't reach the tier where that trade-off pays for itself. Authentication/authorization differs sharply by boundary: the **public-facing widget API** (contractor-embedded, different threat model) uses API Gateway's toolkit — API keys + usage plans per embedding contractor (identification, rate-limiting, and cost-abuse defense against a public endpoint that triggers paid Gemini calls), Lambda authorizers for finer-grained tenant checks, AWS WAF against volumetric/exploit abuse, and explicit CORS configuration since the widget loads cross-origin. The **backend↔Gemini boundary** is a separate trust zone entirely, secured via API-key/service-account secrets management, not shared credentials with the public API.

Webhook interoperability follows the now-standard shape confirmed both by Gemini's own headers (`webhook-id`, `webhook-timestamp`) and broader industry practice (Svix-style infrastructure): **HMAC signature verification** proves sender authenticity, and a **timestamp rejected outside a ~5-minute window** defeats replay attacks. Widget embedding (iframe or injected script on contractor sites) relies on `window.postMessage()` for any parent-page↔widget communication, with two non-negotiable rules: always specify an explicit `targetOrigin` (never `*`) on outbound calls, and always validate `event.origin` before trusting `event.data` on the receiving end — both defend the homeowner photo/diagnosis data crossing this specific trust boundary.

_Standards Compliance: HMAC + timestamp webhook verification and explicit-origin `postMessage` are both now standard, well-documented practice, not bespoke security design._
_Integration Challenges: keeping the public-ingress boundary and the third-party-egress boundary using independent controls — conflating them (e.g., shared secrets) collapses a boundary that should stay separate._
_Source: [AWS API Gateway — Control and manage access to REST APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html)_
_Source: [Svix — Why Verify Webhooks](https://docs.svix.com/receiving/verifying-payloads/why)_
_Source: [MDN — Window: postMessage() method](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)_

## 6. Performance and Scalability Analysis

### Performance Characteristics and Optimization

The 60-second sync boundary is the system's hardest performance constraint, and it must absorb Lambda cold starts, network egress to Google (not in-VPC), Gemini's own inference time, and any bounded retry — meaning the retry policy has to be tightly scoped (one or two attempts, small backoff) rather than a generic exponential-backoff policy that could itself exhaust the budget. Two cost-adjacent optimizations double as performance levers: Gemini's **implicit context caching** (automatic on Gemini 2.5+ models, visible via `usage.total_cached_tokens`) reduces both cost and latency when large repeated content (e.g., a fixed trade-specific instruction preamble) sits at the start of the prompt; and routing non-latency-sensitive workloads (fine-tuning dataset backfill, historical reprocessing) through Gemini's **Batch API** or **Flex/Priority inference** tiers keeps the production photo/video paths uncontended by bulk work.

_Performance Benchmarks: no case in this research for binary serialization or gRPC — current volumes and payload shapes don't reach the tier where that complexity pays off._
_Optimization Strategies: prompt structure (large/common content first) to maximize implicit cache hits; tier-routing bulk/offline work away from the real-time path._
_Source: [Gemini API — Context caching](https://ai.google.dev/gemini-api/docs/caching)_

### Scalability Patterns and Approaches

The video path is architecturally a **Queue-Based Load Leveling** problem: an SQS buffer between the webhook receiver and the processing worker means bursty homeowner submission volume never directly drives a bursty call pattern against S3/DynamoDB/dashboard-notification — the worker drains at a controlled rate, the receiver stays maximally simple (verify → check idempotency → enqueue → fast 2xx), and failures are isolated (a slow downstream write can't trigger Gemini webhook-redelivery storms, since the receiver already acknowledged). This requires **idempotent consumers** by design (standard queues deliver at-least-once, reinforcing rather than duplicating the existing webhook idempotency-key table) and a **dead-letter queue** for jobs that fail repeatedly, so they're investigable rather than silently dropped.

Scaling from Phase 1's low volume toward the Phase 4 hybrid-routing threshold (~10,000–20,000/month) requires no architecture change under this design — only the number of worker instances draining the queue needs to scale, and the circuit breaker/retry layer already contains third-party degradation regardless of volume.

_Scalability Patterns: queue-based decoupling supports the full Phase 1→4 volume growth path without an architecture change._
_Capacity Planning: RPM tiering (Tier 1 → Tier 2 → Tier 3) is the concrete input for when hybrid routing or multi-project sharding becomes necessary._
_Source: [Azure Architecture Center — Queue-Based Load Leveling pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling)_

## 7. Security and Compliance Considerations

### Security Best Practices and Frameworks

Two independently-secured trust boundaries run through this entire architecture and must not be conflated: the **public ingress boundary** (homeowner/widget → backend, defended by API Gateway's API keys + usage plans, Lambda authorizers, WAF, and explicit CORS) and the **third-party egress/ingress boundary** (backend ↔ Gemini, defended by API-key/service-account secrets management plus HMAC webhook signature verification and timestamp replay protection). Widget-embedding security (`postMessage` origin-locking on both send and receive sides) is a third, browser-level boundary specific to the embed-on-contractor-sites delivery model.

_Threat Landscape: a public endpoint that triggers paid third-party API calls per request is a direct cost-abuse vector if unprotected, not merely an availability concern — WAF and usage-plan throttling are cost controls as much as security controls here._
_Secure Development Practices: circuit breaker and retry belong strictly on the backend↔Gemini leg, never on the widget↔backend leg, where a fast honest failure is preferable to backend-side retries eating into the 60-second budget._
_Source: [AWS API Gateway — Control and manage access to REST APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html)_

### Compliance and Regulatory Considerations

No industry-specific regulatory framework was identified as directly applicable within this research's scope (general contracting/home-services diagnosis imagery, not health or financial data), but the data-handling chain — homeowner-submitted photos/video flowing through a third-party (Google) API — means data-residency and retention posture should be made explicit: File API copies expire after 48 hours by design (not a durable record), so S3 is the system of record and any data-retention policy should be defined against the S3/outcome-log store, not assumed to be handled by Gemini's transient storage.

_Regulatory Compliance: no confirmed regulatory obligation is analyzed here beyond general data-handling hygiene; flagged as an area for the business/legal side to confirm against jurisdiction-specific requirements._
_Audit and Governance: DynamoDB's structured diagnosis record (keyed by request ID) and the S3 outcome-log are the natural audit trail for any future compliance or contractor-dispute review._

## 8. Strategic Technical Recommendations

### Technical Strategy and Decision Framework

Build the orchestrator behind a **hexagonal (ports-and-adapters) boundary from Phase 1**, even though only one vendor (Gemini) and one model tier exist at launch — this is the single highest-leverage architectural decision in this research, because it's what keeps the two-vendor posture, the Phase 4 hybrid-routing cascade, and unit testability all cheap simultaneously, rather than requiring separate rework for each later. Layer **Circuit Breaker + Retry** around the adapter (not the domain logic) specifically for the Gemini call, tuned tightly against the 60-second sync budget. Treat the video path as a **Queue-Based Load Leveling** problem from day one rather than a direct webhook-to-processing pipeline.

_Architecture Recommendations: ports-and-adapters + circuit breaker/retry + queue-based leveling, together, form the architectural core this entire report converges on._
_Technology Selection: AWS SAM for IaC given the pure-serverless stack; Step Functions deliberately deferred to Phase 4._

### Competitive Technical Advantage

The accuracy-as-differentiator positioning (per the originating brainstorm intent) means the hybrid-routing cascade's escalation-rate tuning (targeting the 15–35% published band) is a direct competitive lever, not just a cost optimization: under-escalating ambiguous cases to save cost is the more dangerous failure mode for a product whose entire value proposition is diagnostic trustworthiness. The outcome-logging pipeline built in Phase 1 (S3 originals + DynamoDB diagnosis records + contractor-confirmed outcomes) is also the foundation for the Phase 3 fine-tuning advantage — a proprietary, outcome-verified dataset that a competitor calling a vendor API generically would not have.

_Technology Differentiation: schema-enforced structured diagnosis + outcome-verified fine-tuning data are the two technical assets that compound over time into a defensible accuracy advantage._
_Innovation Opportunities: the outcome-log dataset is valuable independent of whether Phase 3 fine-tuning ships on schedule — it's useful for prompt/cascade-threshold tuning even before a fine-tuned model exists._

## 9. Implementation Roadmap and Risk Assessment

### Technical Implementation Framework

1. **Phase 0 (build-ready):** SAM project skeleton (API Gateway + orchestrator Lambda + DynamoDB + S3), Gemini call isolated behind a ports-and-adapters boundary, mock adapter in place for local/unit testing from the start.
2. **Phase 1 (this research's scope):** real Gemini adapter with `responseSchema`-enforced structured output; circuit breaker + bounded retry on the sync photo path; queue-based webhook pipeline (signature + timestamp verification → idempotency check → SQS → worker) for the async video path.
3. **Phase 2:** outcome-logging pipeline (S3-durable originals + DynamoDB diagnosis records + contractor-confirmed-outcome capture) feeding the eventual fine-tuning dataset.
4. **Phase 3:** GCS export path for Vertex AI supervised fine-tuning once the outcome-log dataset reaches the 100–500 example minimum.
5. **Phase 4:** hybrid-routing cascade (cheap → premium escalation) as a Step Functions state machine invoking the same adapter interface established in Phase 0, targeting the 15–35% escalation-rate band.

_Implementation Phases: each phase adds a capability without requiring rework of the prior phase's architecture — the ports-and-adapters boundary and the resilience layer are both established in Phase 0/1 and simply reused thereafter._
_Resource Planning: team needs working familiarity with AWS serverless primitives + SAM tooling, Gemini's structured-output/File-API/webhook mechanics, and resilience-pattern literacy (circuit breaker, retry, queue leveling) sufficient to tune rather than merely implement them._

### Technical Risk Management

Three risks dominate: **vendor coupling** (mitigated by the hexagonal boundary — a second vendor or hybrid routing is an adapter addition, not a rewrite); **third-party dependency failure cascading into the homeowner-facing path** (mitigated by circuit breaker + retry + queue-based leveling, containing a Gemini-side outage to a fast, honest failure rather than a pile of timed-out Lambda invocations); and **silent schema drift** (Gemini's schema enforcement is a subset of full JSON Schema and can silently drop unsupported keywords — mitigated by the staging smoke-test tier that exercises the real adapter against generated types before every production deploy).

_Technical Risks: schema drift is the least obvious but most insidious risk — it fails silently rather than loudly, which is exactly why a dedicated smoke-test tier (not just unit tests against a mock) is required._
_Business Impact Risks: an under-escalating hybrid-routing cascade (Phase 4) is a business-critical risk specifically because the product's differentiator is accuracy — cost savings from under-escalation would directly undermine the core value proposition._

## 10. Future Technical Outlook and Innovation Opportunities

### Emerging Technology Trends

**Near-term (1–2 years):** the webhook-over-polling shift and guaranteed-schema structured output are already mainstream and stable enough to build on directly rather than treat as bleeding-edge; expect Gemini's quota tiers and inline-data limits to continue increasing (as already observed in the January 2026 and April 2026 changes tracked in this research), which should be revisited periodically rather than assumed static. **Medium-term (3–5 years):** model cascading/routing is likely to become a built-in vendor feature (rather than custom-built escalation logic) as the pattern matures further — worth watching Gemini's own roadmap for a native routing/cascade offering that could simplify or replace the Phase 4 Step Functions design. **Long-term:** the outcome-logging dataset accumulated from Phase 1 onward becomes a compounding asset regardless of specific vendor/model choices — it's portable to whatever fine-tuning or routing mechanism is current at the time Phase 3/4 actually ships.

_Near-term Technical Evolution: continue tracking Gemini API changelog/release notes given the pace of change already observed within this single research window._
_Medium-term Technology Trends: native vendor-side model routing could reduce the custom cascade logic this report recommends building in Phase 4._

### Innovation and Research Opportunities

The outcome-logging pipeline is the clearest innovation surface: because it's built from Phase 1 (independent of whether Phase 3 fine-tuning ships on schedule), it's available for earlier uses — tuning the Phase 4 cascade's confidence thresholds against real contractor-confirmed outcomes, for instance, well before a fine-tuned model exists. Worth flagging as a research opportunity for a later phase: whether token-level confidence signals (used for cascade escalation) can also feed the homeowner-facing "confidence" field in the diagnosis schema directly, rather than being a purely internal routing signal.

_Research Opportunities: reusing cascade-routing confidence signals as the homeowner-facing confidence field, rather than computing them separately._
_Emerging Technology Adoption: revisit Gemini's own routing/cascade features if/when they become a first-party offering, as a potential simplification of the Phase 4 custom build._

## 11. Technical Research Methodology and Source Verification

### Comprehensive Technical Source Documentation

**Primary Technical Sources:** Google Gemini API documentation (structured output, image understanding, rate limits, context caching), Google Cloud/Vertex AI documentation (supervised fine-tuning), Google's official blog (webhook launch, file-size-limit increase), AWS documentation (API Gateway access control, AWS SAM), and the AWS Compute Blog (serverless LLM integration patterns).

**Secondary Technical Sources:** Azure Architecture Center (Circuit Breaker, Retry, Queue-Based Load Leveling patterns — used as the canonical pattern reference despite the AWS-native implementation, since these are cloud-agnostic architectural patterns), Alistair Cockburn's original Hexagonal Architecture paper, Svix's webhook-verification documentation, MDN's `postMessage()` reference, and a current LLM cascade-routing survey.

**Technical Web Search Queries:** Gemini API authentication and service-account practices; AWS API Gateway authorization mechanisms; webhook signature verification and HMAC/replay protection; SQS/Lambda decoupling and asynchronous microservice communication; cross-origin `postMessage` security; circuit breaker, retry, and queue-based load-leveling cloud design patterns; hexagonal/ports-and-adapters architecture; AWS SAM vs. CDK; Gemini context caching and cost optimization.

### Technical Research Quality Assurance

_Technical Source Verification: every architectural or API-behavior claim in this report was checked against a live, current source rather than relied upon from general training knowledge — several details (100MB inline limit, April 2026 free-tier change, May 2026 webhook launch) are recent enough that stale knowledge would have produced an incorrect design._
_Technical Confidence Levels: high confidence on Gemini API mechanics (directly sourced from ai.google.dev) and on the resilience/architectural patterns (canonical, stable references); moderate confidence on exact quota figures, which are explicitly flagged in Technology Stack Analysis as "subject to change, but directionally stable for capacity planning."_
_Technical Limitations: no regulatory/compliance framework was identified as directly applicable and confirmed — this is flagged for business/legal follow-up rather than asserted; the escalation-rate tuning band (15–35%) is a general heuristic from cascade-routing research, not a figure specific to this product's diagnosis domain, and should be validated against real outcome data once Phase 4 is live._

## 12. Technical Appendices and Reference Materials

### Detailed Technical Data Tables

**Resilience Pattern Placement:**

| Boundary                      | Pattern(s) Applied                                          | Rationale                                                                  |
| ----------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------- |
| Widget ↔ Backend (sync photo) | Fast, honest failure — no backend-side retry                | Preserves the 60s budget; retries belong on the Gemini leg, not this one   |
| Backend ↔ Gemini              | Circuit Breaker + bounded Retry                             | Third-party dependency outside AWS's network; must fail fast once degraded |
| Gemini Webhook → Backend      | Idempotency check + signature/timestamp verification        | Gemini owns webhook retry/redelivery; receiver must be safe to call twice  |
| Webhook Receiver → Worker     | Queue-Based Load Leveling (SQS) + idempotent consumer + DLQ | Decouples bursty intake from downstream write rate; isolates failures      |

**Storage Responsibility Table:**

| Store    | Responsibility                                                                | Phase   |
| -------- | ----------------------------------------------------------------------------- | ------- |
| S3       | Durable original media + outcome-log dataset                                  | Phase 1 |
| DynamoDB | Webhook idempotency keys (TTL) + structured diagnosis records                 | Phase 1 |
| GCS      | Vertex AI fine-tuning training-data source (export target, not primary store) | Phase 3 |

### Technical Resources and References

**Technical Standards:** JSON Schema (as the basis for Gemini's `responseSchema`, noting its partial-support caveat); HMAC-based webhook signing (industry-standard, not Gemini-specific); OWASP-aligned API Gateway access control (API keys, WAF, CORS).

**Technical Communities/References:** Google AI for Developers documentation hub (ai.google.dev); AWS Architecture Center and Compute Blog; Azure Architecture Center's cloud design pattern catalog (used here as a vendor-neutral pattern reference); Svix's webhook-infrastructure documentation.

---

## Technical Research Conclusion

### Summary of Key Technical Findings

The AI Analysis Engine's Phase 1 architecture is not a novel design problem — it's a well-understood "Direct Lambda to LLM" pattern (API Gateway → Lambda → Gemini), made robust by three confirmed, industry-standard resilience patterns (Circuit Breaker, Retry, Queue-Based Load Leveling) and made extensible by one architectural discipline (hexagonal ports-and-adapters). The photo path and video path are genuinely different implementation problems — inline synchronous vs. File API/webhook/queue asynchronous — and the research confirms concrete, current mechanics for both, down to specific header names, size limits, and quota tiers.

### Strategic Technical Impact Assessment

Getting the resilience layer and the hexagonal boundary right in Phase 1 is what makes Phases 2–4 (outcome-logging, fine-tuning, hybrid routing) additive rather than disruptive. Skipping either — building a Gemini-coupled orchestrator without the ports-and-adapters boundary, or shipping without a circuit breaker on the Gemini call — would mean Phase 4's hybrid-routing cascade requires a rework rather than an adapter addition, and any Gemini-side outage would degrade the homeowner-facing experience directly rather than failing gracefully.

### Next Steps Technical Recommendations

Proceed to Phase 0 scaffolding (SAM project skeleton with the ports-and-adapters boundary and mock adapter in place) before writing the real Gemini integration, so the resilience and testing patterns are established structurally from the first commit rather than retrofitted.

---

**Technical Research Completion Date:** 2026-08-16
**Research Period:** Current comprehensive technical analysis (August 2026)
**Source Verification:** All technical facts cited with current sources
**Technical Confidence Level:** High — based on multiple authoritative technical sources, cross-checked against three prior project documents

_This comprehensive technical research document serves as an authoritative technical reference on the AI Analysis Engine Implementation Architecture and provides strategic technical insights for informed decision-making and implementation._
