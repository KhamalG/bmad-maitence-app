# Gemini-Based Vision/Video Analysis: Gap Check & Implementation Roadmap

Follow-up to the vision-model and vendor-architecture research (`compass_artifact` report). This document checks the "build on Gemini" recommendation against the three original research questions and lays out a phased implementation plan.

---

## Checking the Gemini Recommendation Against the Original Questions

### 1. "What frameworks are out there that I can leverage now? Could I train a model to learn what I'm looking for, and what does that training process look like?"

**Partially answered — with an important nuance.**

Gemini isn't itself a "framework you train a model on" the way Rekognition Custom Labels or YOLO are — it's a general-purpose foundation model called via API. Using it out-of-the-box (zero-shot or few-shot prompting) is a legitimate framework to leverage *now*, but that's a different thing from *training a model to learn what you're looking for*.

There are two distinct paths under the Gemini umbrella, and a complete roadmap needs both:

- **Few-shot / prompt-based calibration** — no training at all, just structured prompts with example descriptions of what plumbing/electrical/HVAC issues look like. Fast, zero data requirement, but has a ceiling on accuracy for subtle distinctions (e.g., a rust stain vs. active corrosion).
- **Supervised fine-tuning of Gemini itself**, via Vertex AI — this is real training: you assemble input/output pairs in JSONL format (image + correct diagnosis label/narrative), and Google recommends starting with roughly 100 examples. This adjusts the model's weights rather than just prompting it, and is the actual answer to "train a model to learn what I'm looking for" using the Gemini family specifically.
  - **Caveat:** supervised fine-tuning support tends to lag the newest model generation. As of a few months ago, fine-tuning wasn't yet available for the Gemini 3.x line, only 2.x. Verify current fine-tuning support for whichever Gemini version you build on before planning around it.
- The custom-detector path (YOLO / SageMaker) from the earlier research is **not replaced** by Gemini — just deferred. It's still the answer if you eventually need scored, auditable per-hazard confidence that a fine-tuned Gemini can't cleanly provide.

### 2. "Can we leverage LLM offerings out there from Claude, ChatGPT, etc.?"

**Fully answered.** This is Gemini's strongest fit: it's a multimodal LLM, so vision and narrative-generation happen in one call rather than needing a separate LLM step. Claude or GPT remain viable as a second opinion or for text-only work elsewhere in the app (e.g., contractor-facing summaries where video input isn't involved).

### 3. "How can I host this engine, and how can it handle a heavy flow of requests and traffic?"

**Partially answered — this is the gap most worth closing before building.** The prior answer ("no idle cost, scales instantly") is true but incomplete. It didn't cover what changes in the AWS-native architecture when the vision/LLM call goes to an external Google API instead of Bedrock:

- You're making an outbound call from your Lambda orchestrator to Google's infrastructure, not an in-VPC AWS service call — different latency, auth (API key or service account), and quota characteristics than Bedrock.
- Google now supports **webhooks for both video processing and batch jobs**. Instead of polling an operation repeatedly, your backend registers a webhook endpoint and Google pushes a signed HTTP POST when a video finishes processing — a cleaner fit for the addendum's "upload → job ID → poll or push notification" pattern than blind polling. Payloads carry a status pointer to the result, not the full result, so you fetch the actual output separately.
- For genuinely large video files (over 100MB) or anything referenced more than once, Gemini's **File API** is the mechanism: upload the video, poll (or now, get webhooked) until it reaches `ACTIVE` state, then reference the file in your generation call. Video is sampled at 1 frame/second by default for visual understanding — worth noting as a limit for very fast-motion content, though unlikely to matter for a homeowner filming a leaking pipe.
- For non-urgent bulk work (e.g., reprocessing historical submissions, nightly batch scoring), the **Batch API runs at 50% of standard cost** with a target turnaround well under its 24-hour ceiling — a real lever for the Outcome Log / FR-15-style backend work, not for the live 60-second-response homeowner path.

---

## Implementation Roadmap

### Phase 0 — Prompt and Schema Design
*1–2 weeks, no infrastructure*

Design the diagnostic prompt per trade (plumbing/electrical/HVAC), each asking for a structured JSON response: likely issue, confidence, urgency tag (per the ER-triage framing), and a homeowner-readable narrative. Validate manually against a small set of real or sourced photos before writing any backend code — this is the cheapest place to find out if zero-shot Gemini is even in the right ballpark for your defect types.

### Phase 1 — Live Photo Path, Zero-Shot
*Weeks 2–5*

Wire the synchronous photo flow: Lambda orchestrator → Gemini API call with the Phase 0 prompt → structured JSON parsed and stored. This is the MVP diagnosis engine, no training required.

In parallel, **start saving every submitted photo + its eventual outcome** (from the contractor's accept/schedule/actual-issue-found flow) into S3 / a dataset table. This is the raw material both fine-tuning and any future custom detector will need — much cheaper to start collecting from day one than to backfill later.

### Phase 2 — Async Video Path
*Weeks 4–7, overlapping Phase 1*

Implement the File API upload + webhook flow for video: contractor dashboard or homeowner app uploads video, backend calls Gemini's Files API, registers the webhook, and a Lambda receiver processes the completion event and fetches the result. This directly replaces the Bedrock frame-extraction plan from the addendum with native video handling.

### Phase 3 — Calibration Once You Have Volume
*Roughly Month 2–3, data-dependent, not calendar-dependent*

Once a few hundred labeled examples per trade have accumulated from Phase 1's outcome logging, run supervised fine-tuning on Gemini via Vertex AI. This is the actual "train a model" step from the original question, applied to the model already in use rather than standing up a second system. Evaluate the tuned model against the zero-shot baseline before switching production traffic over.

### Phase 4 — Selective Custom Detectors
*Month 4+, only if Phase 3 has a ceiling*

For any specific defect type where even a fine-tuned Gemini underperforms — likely the subtle visual distinctions (corrosion patterns, specific electrical hazard signatures) — introduce a narrow YOLO/SageMaker detector for that type only, feeding its structured output into the Gemini narrative step rather than replacing it. This keeps the LLM as the single narrative surface while adding scored precision only where it's earned.

---

## Open Item for the PRD

Registering and securing a webhook receiver endpoint, and deciding whether video processing lives fully in the async Gemini flow or whether an S3 copy is still needed for compliance/audit regardless of what Google's Files API does with it — worth resolving before Phase 2 implementation starts.
