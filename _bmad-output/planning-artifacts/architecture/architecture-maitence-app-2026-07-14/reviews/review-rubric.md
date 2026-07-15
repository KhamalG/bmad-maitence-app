# Review: ARCHITECTURE-SPINE.md (maitence-app, 2026-07-14)

**Verdict:** The spine correctly fixes domain boundaries, data ownership, auth, and contact-info security for every FR-1–FR-19 capability, but it is silent or internally inconsistent on three load-bearing areas — event-delivery semantics (idempotency/ordering) for its own default integration mechanism, the operational envelope (alerting, DR/backup, on-call), and verification of the precise tool versions it locks — any of which could let independently-built domains diverge exactly the way this document exists to prevent.

---

## Critical Findings

### C1. No idempotency/ordering convention for the default integration mechanism (AD-4)
AD-4 makes pub/sub over EventBridge the *default* cross-domain integration, and the Event catalog shows real fan-out (e.g. `IssueMappedToService` → Trust & Quality *and* Job Queue & Lifecycle; `LeadDraftCreated` and `IssueMappedToService` both feeding Trust & Quality for the same Lead). EventBridge gives at-least-once delivery with no ordering guarantee. The spine's Consistency Conventions table fixes the event *envelope* shape (including a `correlationId` and `version`) but never states how a consumer must handle a duplicate delivery or an out-of-order arrival. With ten independently-built domains each writing their own consumer, some will dedupe on `eventId`, some won't; some will assume `LeadDraftCreated` arrives before `IssueMappedToService`, some won't. This is not in the Deferred section either — it's an outright omission on the very mechanism the spine designates as default. This should be a fixed convention (e.g., "consumers are idempotent, keyed on `eventId`; no consumer may assume delivery order across event types").

### C2. Operational envelope is effectively silent
The checklist's known failure mode — a spine that's domain-boundary/security-focused but silent on the operational envelope — applies here. What *is* decided: structured logging + correlationId + X-Ray (Consistency Conventions), and an edge/account security baseline (AD-12: WAF, CloudTrail, GuardDuty, Security Hub). What is **not** decided, deferred, or even flagged as an open question anywhere in the document:
- Alerting/paging on the CloudWatch/X-Ray data being collected (no SLOs, error budgets, on-call model, or runbook expectation).
- Backup/DR strategy (RPO/RTO) for the DynamoDB tables and Aurora Serverless clusters that hold Lead, Job, and contact-info data.
- Any deployment/promotion model for environments (dev/staging/prod) — the "AWS account topology" Deferred item addresses account *isolation*, not environment promotion, config-per-environment, or how staging exercises real vs. synthetic PII given how strong AD-8's contact-info isolation is.
Given this is the initiative-altitude document, these are exactly the structural dimensions it should own, defer explicitly, or flag as an open question — not leave unaddressed.

### C3. Named tech versions asserted with unverified precision
The Stack table pins **AWS CDK 2.261.x**, **Aurora PostgreSQL 17.7 (platform version 4)**, and **nodejs22.x (LTS)** as flat facts, with no citation of a release-notes check or verification step. The document is dated 2026-07-14; patch-level version numbers this specific for a fast-moving toolchain (CDK ships weekly) read as plausible-sounding rather than confirmed-current. Before any builder pins these in `package.json`/CDK bootstrap, the actual current versions should be verified against AWS's own release notes rather than carried forward as asserted.

---

## High Findings

### H1. Deferred "AWS account topology" doesn't fix the one thing that prevents divergence today
The Deferred section says naming/tagging conventions "should anticipate the split from day one even if the split itself is deferred" — but doesn't actually fix any naming/tagging convention (stack-naming scheme, environment suffixes, tag keys for cost allocation). This is precisely the kind of low-level convention the Consistency Conventions table exists to fix for entity IDs and event envelopes; leaving it as unfixed prose invites two teams standing up dev/staging today to diverge on exactly this.

### H2. Capability map cites the wrong AD for Notifications
`Capability → Architecture Map`: "Push alerts | Notifications | **AD-12**." AD-12 is the edge/account security baseline (WAF, CloudTrail, GuardDuty, Security Hub) — it has nothing to do with push-notification delivery. This looks like a copy/reference error; Notifications' actual governance is the event catalog (`EmergencyLeadRouted`, `ContactRevealed` → Notifications) under AD-4, not AD-12.

### H3. AD-3 Binds vs. Rule text disagree on Outcome & Pricing
AD-3's **Binds** line is "all client-facing FRs (FR-1–FR-14, FR-17–FR-19)" — it skips FR-15/FR-16 (Outcome & Pricing). But AD-3's own **Rule** text lists `/outcomes` as one of the API-Gateway route groups fronted by the single gateway. The document is internally inconsistent about whether Outcome & Pricing sits behind the shared, Cognito-authorized gateway or not — a future builder could reasonably read either.

### H4. Consistency Conventions have no enforcement mechanism named
The error envelope, event envelope, and entity-id conventions are stated as prose rules with no shared-library, Lambda-layer, schema-registry, or contract-test mechanism named to enforce them. For ten independently-built Lambda services, this is exactly the class of thing that quietly drifts (one domain's error shape omits `requestId`, another's `eventType` isn't strictly PascalCase-past-tense). AD-1/AD-9/AD-10 get real enforcement (IAM policy, separate Cognito pools); the Conventions table doesn't get an equivalent teeth-mechanism.

---

## Medium / Low Findings

- No AWS region(s) named anywhere in the document.
- No cost-guardrail/budget-alarm mention for a Bedrock/Rekognition-heavy, pay-per-call AI pipeline — a real operational risk given FR-1's synchronous-latency promise likely pushes toward provisioned concurrency.
- AD-2's promise that "splitting a stack later must never change a route or event shape" has no contract-testing or versioning gate named to actually catch a violation — currently just a stated intention.
- AD-13 (CDK-only IaC) names no CI/repo-level check preventing a raw CloudFormation or Terraform stack from creeping in — relies on convention alone.
- The Identity context has no dedicated AD beyond AD-7/AD-9 and doesn't appear as its own node in the Structural Seed diagram (Cognito stands in for it) — cosmetic, not load-bearing, but worth a one-line clarification of where token-exchange/session-promotion Lambdas actually live relative to the two Cognito pools.

---

## What the spine gets right (for balance)

- FR-1 through FR-19 all resolve cleanly in the Capability → Architecture Map; no capability gap.
- AD-8 (contact-info isolation) and AD-7 (anonymous session scoping) are sharp, concrete, and each directly enforceable at the code/IAM level — genuinely prevent the divergence they name.
- AD-5's DynamoDB/Aurora split is well-reasoned against actual access patterns (Taxonomy's join-heavy relational shape vs. high-write single-entity domains), not a default-everything-to-one-database call.
- Deferred items for the AI-mapping algorithm, numeric latency/accuracy targets, and Space Visualization's generative mechanism are legitimately unfixed at this altitude and correctly traced back to their PRD Open Questions rather than silently dropped.
