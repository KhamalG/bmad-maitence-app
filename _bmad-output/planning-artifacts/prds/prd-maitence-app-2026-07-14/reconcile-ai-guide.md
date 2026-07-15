# Reconciliation: AI_Photo_Video_Analysis_Guide.md vs. prd.md / addendum.md

Source input: `docs/AI_Photo_Video_Analysis_Guide.md`
Checked against: `prd.md` and `addendum.md` (prd-maitence-app-2026-07-14)

## Summary judgment

The addendum's "AI Vision-Analysis Spike — Technical Direction" section (lines 21-38) is a faithful and largely complete distillation of the guide. It correctly captures: backend-centric architecture and rationale; Rekognition for structured detection + Bedrock/multimodal LLM for reasoning; the sync-photo/async-video split with the correct underlying reason (Bedrock's frame-extraction-only video support); the AWS example pipeline (API Gateway → Lambda/ECS/EKS → S3 → Rekognition/Bedrock → DB); and the cost/scale levers (resize/compress, cache, route to lower-cost models, queue/retry video). PRD §4.1 FR-1's Feature-specific NFRs correctly reflect the sync-image / async-video split and correctly avoid over-specifying implementation, deferring to the addendum as intended.

Two gaps found, both minor-to-moderate; no critical omissions.

## Gap 1 (moderate): SageMaker / Custom Vision option entirely absent

The guide's "Combining Multiple AI Services" table (lines 173-182) lists six task→service mappings. The addendum's service mapping (lines 27-30) covers five of six (Rekognition→object detection/OCR, Bedrock→reasoning, Textract/Transcribe→OCR/speech) but never mentions **SageMaker for Custom Vision** (guide lines 38-44: industry-specific custom models, custom object detection) — despite the guide listing it as a first-class AWS AI service alongside Rekognition and Bedrock (comparison table, lines 9-17).

Why this matters beyond "missing implementation detail": the PRD explicitly commits to making the AI Diagnosis "best-in-class" as the core competitive differentiator (§1 Vision, §8 Open Question 8) rather than merely offering AI diagnosis (competitors already do). A custom-trained vision model (e.g., detecting specific patterns — water damage staining, corrosion, panel-specific electrical hazards — that generic Rekognition object detection wouldn't distinguish) is a plausible lever for hitting that bar, and it's the one AWS-native option the guide offers for it. It isn't surfaced anywhere in the PRD or addendum as even a candidate/open item.

**Suggested handling:** not a PRD capability requirement (too solution-specific for a capabilities doc), but worth a one-line addition to the addendum's spike section noting SageMaker/custom-labels as a candidate lever for the "best-in-class" bar, cross-referenced from PRD §8 Open Question 8.

## Gap 2 (minor): OCR-for-equipment-identification not connected to a concrete capability

The addendum mentions OCR only generically: "Amazon Textract / Transcribe — available if OCR or spoken description transcription becomes relevant (e.g. a Homeowner narrates the issue on video)." The narration/speech-to-text example is well covered. But the OCR half is left abstract — it never connects to the concrete capability the guide's data implies: Rekognition's own listed capabilities include "Text detection (OCR)" (guide line 24), and the Combining-Services table lists OCR→Textract independently. A natural use: reading model/serial numbers or nameplate labels off equipment in a homeowner's photo (e.g., a water heater's brand/model plate, a breaker panel label) to sharpen the AI Diagnosis ("likely a failed pressure relief valve on a [Brand] [Model] unit") or to enrich Outcome Log data (FR-12/FR-13) with equipment specifics.

This isn't reflected as a capability idea anywhere — not in PRD §4.1 FR-1 (which only discusses photo/video/description as generic inputs), not in the Open Questions, not in the addendum beyond the vague "if OCR becomes relevant" phrasing.

**Suggested handling:** minor — could be folded into PRD §8 Open Question 8 ("what makes the analysis best-in-class") as one candidate capability, or left as-is since the guide itself doesn't make this connection explicitly either (it's an inference from combining two parts of the guide, not a guide recommendation). Flagging for completeness per the task's explicit prompt to check this angle.

## Explicitly checked and NOT gaps

- **Sync images / async video split**: Guide (lines 186-198) recommends synchronous images (1-5s) and asynchronous video (upload → job ID → poll/push). Addendum (lines 32-34) and PRD §4.1 FR-1 Feature-specific NFRs (lines 115-117) both correctly reflect this, including the specific Bedrock-frame-extraction rationale for why video can't be sync. No gap.
- **Backend-centric architecture**: Fully and accurately captured in addendum lines 25, including the four-part rationale (lightweight app, server-side API keys, swappable providers, queue/retry for video) — a superset/paraphrase of the guide's stated benefits (security, flexibility, scalability, cost). No gap.
- **Cost/scale levers**: resize/compress, cache, route to lower-cost models, async+retry/queue for video — all four from the guide's "Cost Optimization" and "Scalability" bullets (lines 143-151) are captured verbatim in addendum line 36. No gap.
- **Speech-to-text (Transcribe)**: Explicitly captured with the homeowner-narrates-video example. No gap.
- **Rekognition vs. Bedrock division of labor**: structured detection vs. higher-level reasoning — correctly and specifically mapped in addendum lines 28-29, matching guide lines 19-36. No gap.
- **Open-source models (LLaVA, YOLOv11, SAM 2, etc.)**: Guide lists these but doesn't recommend them for the production path; correctly absent from both PRD and addendum since the guide's own "Overall Recommendation" doesn't select them. No gap.
- **Enterprise API example / JSON response shape**: Pure implementation detail (guide lines 201-216); correctly not reflected in PRD (capability-focused) or restated in addendum (already covered at the architecture-pattern level via "unified JSON response to the app," addendum line 25). No gap.
- **Amazon Comprehend / Amazon Nova**: Listed in the guide's AWS services table but not part of its own "Overall Recommendation" section — reasonable to omit from the addendum's distillation. No gap (noting only as a completeness footnote, not an issue).
