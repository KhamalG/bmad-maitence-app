---
name: AI-Powered Lead Intake Platform for Contractors
status: final
created: 2026-07-14
updated: 2026-07-14
sources:
  - ../../prds/prd-maitence-app-2026-07-14/prd.md
  - ../../prds/prd-maitence-app-2026-07-14/addendum.md
  - ../../../../docs/Property_Trades_Taxonomy.md
---

# AI-Powered Lead Intake Platform for Contractors — Experience Spine

## Foundation

Two surfaces, one product:

- **Homeowner app** — native mobile, iOS (Swift) + Android (Kotlin), platform parity. No UI system named; inherits platform navigation, gesture, and dynamic-type conventions. `[NOTE FOR UX]` A cross-platform framework (React Native, Flutter) was never discussed in the PRD — native-per-platform is the assumed build target per the PRD's tech stack, not a UX decision.
- **Contractor/Admin dashboard** — responsive web, Angular. `[ASSUMPTION]` No component library (Angular Material or otherwise) is named in the PRD; this spine specifies behavior independent of implementation library — Architecture picks the library, this spine's behavioral rules hold regardless.

`DESIGN.md` is the shared visual identity reference for both surfaces; this spine is the experience layer for each. Single-tenant per Contractor account; a Contractor may have multiple Admin/Dashboard User seats at Tier 3 (PRD §4.8), all seeing the same Job Queue.

## Information Architecture

**Homeowner app (mobile)**

| Surface | Reached from | Purpose |
|---|---|---|
| Home | App open | Choose Guided Assessment Flow or Skip-AI Path; entry point for both Repair/Emergency and Improvement/Inspection intake |
| Guided Assessment | Home → "Diagnose my problem" | Photo/video capture, description, budget input (PRD FR-1) |
| AI Diagnosis Result | End of Guided Assessment | AI Diagnosis + Misdiagnosis Disclaimer, inline (FR-1) |
| Condition Assessment Result | End of Guided Assessment (Inspection-mapped) | Condition Assessment, same card pattern as AI Diagnosis (FR-3) |
| Visualization Result | End of Guided Assessment (Improvement-mapped, optional) | Space Visualization, additive not gating (FR-4) |
| Quick Submit | Home → "I know what I need" | Skip-AI Path — job description + time window (FR-2) |
| Contractor Match | After Diagnosis/Quick Submit | Single matched Contractor profile, why-matched context |
| No Match Yet | After Diagnosis/Quick Submit, when the Lead is Cold or the Service-mapping needs fallback categorization | Honest, actionable state — never a fake "finding your contractor" spinner (FR-5, FR-8) |
| Booking Confirmation | Contractor Match → Book | Confirms visit; Contact-Reveal Event fires here (FR-13) |
| Budget Realism Note | Inline during Guided Assessment / Quick Submit, before submit | Light realism note if stated budget looks off (FR-9) |

**Contractor/Admin dashboard (web)**

| Surface | Reached from | Purpose |
|---|---|---|
| Job Queue | App open (default) | Card-based incoming Leads, Emergency-priority sorted first (FR-12, FR-14) |
| Lead Card (expanded) | Tap/click a queue card | Full detail: photos, video, AI Diagnosis/Assessment, budget, Lead-Quality Signal |
| Cold Leads | Job Queue → filter toggle | Deprioritized view of filtered-out Leads, not deleted (FR-8) |
| Active Jobs | Nav | Accepted Jobs in progress |
| Job Completion | Active Jobs → mark complete | Captures actual final cost + AI Diagnosis match/no-match into the Outcome Log (FR-15) |
| Trade/Service Selection | First login, step 1 of Verification | Search-first, Trade-scoped picker — see Component Patterns (FR-11) |
| Verification / Onboarding | First login, step 2, after Trade/Service Selection | Licensing/legitimacy verification gate; account is unusable until this clears (FR-11) |
| Trade/Service Settings | Account nav, post-onboarding | Same picker component, reachable any time to add/remove Trades and Services |
| Account / Tier | Nav | Tier details, Admin seat management (FR-17) |
| Performance Insights | Nav (post-MVP, FR-19) | Conversion, response time, volume trends — Should-Have, not in MVP build |

→ Composition reference: `mockups/diagnosis-result.html` (AI Diagnosis Result), `mockups/job-queue.html` (Job Queue, collapsed + expanded Lead Card), `mockups/trade-service-picker.html` (Trade/Service Selection + Settings, both steps). Remaining surfaces build from the tables in this file alone. Spine wins on conflict with any mock.

## Voice and Tone

Microcopy. Brand voice and aesthetic posture live in `DESIGN.md.Brand & Style`.

| Do | Don't |
|---|---|
| "Here's what we think is going on." | "AI Diagnosis Complete: Confidence 87%" |
| "This is our best guess — your contractor will confirm on-site." | "Disclaimer: This diagnosis may be inaccurate and is not a substitute for professional inspection." |
| "3 leads waiting." | "You have 3 unread notifications." |
| "This one's urgent — a pipe's actively leaking." (Emergency) | "⚠️ EMERGENCY LEAD ⚠️" |
| "Nothing here yet — leads will show up as they come in." | "No data available." |
| Same warm, direct voice to Homeowner and Contractor. | Corporate/legal register for disclaimers, casual/gimmicky register everywhere else. |

## Component Patterns

Behavioral. Visual specs live in `DESIGN.md.Components`.

| Component | Use | Behavioral rules |
|---|---|---|
| AI Diagnosis Card | Homeowner: Diagnosis Result | Disclaimer renders inline, same card, never a separate screen or dismissible modal. Low-confidence media triggers a re-prompt for one more photo/angle before the card renders (FR-1). → `mockups/diagnosis-result.html` |
| Lead Card | Contractor: Job Queue | Collapsed shows quality signal, trade/service label, and urgency at a glance — zero taps. Expands in place (accordion), never navigates to a new screen. Emergency Badge always visible even collapsed; other Work Types (Repair, Maintenance, Improvement, Inspection) are conveyed through the trade/service text label only, not a separate badge — badges stay reserved for Emergency and Verified (FR-12, FR-14). → `mockups/job-queue.html` |
| No Match Yet panel | Homeowner: after Diagnosis/Quick Submit, when Cold or fallback-categorized | Names the likely reason in plain language and gives one concrete next action (edit budget, add a photo, add detail) — never a generic "please wait" with no way forward (FR-5, FR-8). |
| Accept action | Contractor: Lead Card (expanded) | One tap/click from expanded card = accept. Max two taps/clicks total from queue view (FR-13). |
| Budget Realism Note | Homeowner: pre-submit, Guided Assessment + Quick Submit | Appears inline below the budget field when the stated amount looks off; never blocks submission, self-dismisses on edit (FR-9). |
| Contact-Reveal moment | Contractor: Booking confirmed / Lead accepted | Contact info reveals with a visible confirmation state (not silent); Contact-Reveal Event fires the instant it's visible, logged regardless of what the Contractor does next (FR-13). |
| Cold Lead filter toggle | Contractor: Job Queue header | Switches queue view between default (qualified only) and Cold Leads; Cold Leads never auto-appear in default view (FR-8). |
| Trade/Service Picker | Contractor: onboarding step 1, and later via Trade/Service Settings | MVP scope is the Phase-1 Trade subset only — provisionally Plumbing, Electrical, HVAC (PRD §8 OQ1) — shown as a short checklist, not the full ~20-Trade taxonomy. Within a selected Trade, Services are search-first (search input + checklist), since even one Trade's Service list can run long. The full taxonomy is the long-term data model (PRD §6.2); this picker's UI should not be built against it until multi-trade expansion is scoped (FR-11). → `mockups/trade-service-picker.html` |
| Verification gate | Contractor: first login | Hard gate — no Job Queue is reachable until verification (including the Trade/Service Picker's output) passes (FR-11). |
| Tier Badge | Contractor: Account/Tier surface only | Renders as a neutral, text-labeled account attribute — never implies rank or quality. Matching Priority, the ranking mechanic Tier confers (FR-18), has no visual representation anywhere in the product; it is a backend-only mechanic, deliberately not exposed as a sellable feature, to keep the within-quality-band trust boundary intact. |

## State Patterns

| State | Surface | Treatment |
|---|---|---|
| Low-confidence media | Homeowner: Guided Assessment | Re-prompt for one more photo/angle rather than rendering a low-confidence AI Diagnosis silently (FR-1). |
| AI Diagnosis pending | Homeowner: Guided Assessment → Result | Synchronous, sub-few-second — a lightweight in-card loading state, not a full-screen spinner or progress bar (PRD: latency treated as core value prop, not a background task). |
| Video still processing | Homeowner: Diagnosis Result | AI Diagnosis renders immediately from photo/description; video processes async and attaches to the Lead without blocking or re-rendering the already-shown Diagnosis (FR-1). |
| Empty Job Queue | Contractor: Job Queue | "Nothing here yet — leads will show up as they come in." No illustration needed; keep it low-key. |
| Cold Lead filtered | Contractor: Job Queue | Lead silently routes to the Cold Leads view — no rejection notification to the Homeowner, no error state (FR-8). |
| Unverified Contractor | Contractor: any post-login surface | Job Queue and all downstream surfaces are unreachable; single Verification surface only, with clear status (pending / needs info / rejected). |
| Emergency Lead arrival | Contractor: Job Queue (already open) | Card inserts at top of list with Emergency Badge; expedited alert (push notification) fires simultaneously — the in-app state and the alert are not sequential, both fire together (FR-14). |
| Skip-AI submission | Homeowner: Quick Submit | Still passes Service-mapping and Lead-Quality checks silently before reaching a Contractor — no visible "checking..." state to the Homeowner; this happens between submit and match, not as a blocking step they watch. |
| Low-confidence Service mapping | Homeowner: after Diagnosis/Quick Submit | Routes to No Match Yet with an honest, specific message ("we need a bit more detail to match you") and a concrete next action — never a spinner that never resolves (FR-5). |
| Cold Lead (fails Lead-Quality) | Homeowner: after Diagnosis/Quick Submit | Routes to No Match Yet with a message tailored to the likely cause (e.g. budget looks off → points back to the Budget Realism Note; thin description → prompts for more detail/photos) — actionable, not a dead end (FR-8). |
| Camera/photo permission denied | Homeowner: Guided Assessment | Explains why the permission is needed in plain language, links to system settings; does not dead-end — Quick Submit remains reachable as a fallback path for a Homeowner who can't or won't grant camera access. |
| AI Diagnosis service error | Homeowner: Guided Assessment → Result | Distinct from low-confidence media: "We couldn't process that just now — try again?" with a retry action; submitted photos/description are preserved, not lost on retry. |
| Offline (mid-capture) | Homeowner: Guided Assessment | Photo/video capture and description entry continue to work locally; submission queues and sends on reconnect with a plain-language "we'll send this as soon as you're back online," not a silent failure — relevant since Maria (UJ-1) is framed as mid-crisis and may have unreliable connectivity. |
| Matched Contractor unavailable | Homeowner: Contractor Match → Booking Confirmation | If the matched Contractor can't take the booking (e.g. time window fills between match and confirm), Homeowner sees a brief explanation and is re-matched automatically rather than dropped back to Home with no context. |

## Interaction Primitives

**Homeowner (mobile):** Tap to act. Native camera/video capture for photo/video steps — no custom capture UI. Swipe between Guided Assessment steps where linear. Pull-to-refresh not used (there's nothing to refresh mid-flow).

**Contractor (web):** Click/tap to act — the two-tap-max discipline (FR-13) is the primary interaction constraint, not a keyboard-first posture (unlike a power-user tool; Contractors are checking a queue between jobs, often one-handed on a phone browser). Hover reveals nothing critical — every action must also work via tap, since FR-14's NFR requires full phone-browser usability.

**Banned everywhere:** infinite scroll on the Job Queue (a Contractor needs to know when they've seen everything — paginate or "load more"), carousels, celebratory/gamified animations on Lead acceptance (this is a professional tool, not a habit app), auto-playing video.

## Accessibility Floor

Behavioral. Visual contrast lives in `DESIGN.md` (every semantic color pairing is now stated with its computed WCAG ratio there — see `DESIGN.md.Colors`).

- WCAG 2.1 AA across both surfaces. `[ASSUMPTION: not explicitly stated in the PRD — standard floor for a consumer-facing product, revisit if a different bar is required.]`
- VoiceOver / TalkBack (mobile): every interactive element labeled with role + state; the AI Diagnosis Card and its Disclaimer Banner are announced as one unit, disclaimer included, never skippable.
- **Maria's flow under assistive tech, not just under stress.** UJ-1 frames Maria as mid-crisis; the Guided Assessment Flow must not assume "stressed but sighted, typical motor function" is the only accessibility scenario in play. Camera capture during Guided Assessment is VoiceOver-operable (standard OS camera accessibility, not a custom capture UI — reinforces the "no custom capture UI" rule in Interaction Primitives). For a Homeowner who cannot practically self-direct a photo (no sighted assistance, low vision), the description field is a genuine alternative path to a Lead, not just a supplement — the Guided Assessment Flow must accept a submission on description text alone if photo capture repeatedly fails, routing to Low-Confidence Media's re-prompt at most once before falling back, not looping indefinitely.
- **FR-13's two-tap flow is reachable by every input method — mouse, keyboard, switch access, voice control.** The Lead Card's expand and Accept controls are real focusable button elements (see `DESIGN.md.Components`, Lead Card), not divs with click handlers — this is what makes them reachable via iOS/Android switch access and voice control on the phone browser, and via Tab + Enter/Space on desktop. One underlying requirement expressed per input method, not a desktop-only concession.
- **Focus visible (WCAG 2.4.7).** `{colors.focus-ring}` renders on every interactive element on every focus/switch/voice-control selection event, including pill-shaped buttons and badges — never suppressed for the sake of the soft, rounded aesthetic.
- **Verification gate status is programmatically announced.** The pending/needs-info/rejected status on the Verification surface is exposed via `aria-live`/`role=alert` (or platform equivalent) so a screen-reader user isn't left guessing why their account — and income — is blocked.
- Tap targets ≥ 44pt (iOS) / 48dp (Android); web click targets ≥ 44px, since the dashboard must work on a phone browser (FR-14 NFR).
- Dynamic type / OS text scaling and browser zoom (to 200%) honored without truncating the Lead Card's quality-signal/urgency summary — that summary is the whole point of a zero-tap scan.
- **Reduced motion.** Card expand/collapse, sheet/modal entrances, and any step-to-step transition in Guided Assessment respect `prefers-reduced-motion` — cut to instant state changes, no vestibular-triggering motion, on a stress-prone persona by design.
- **Cognitive accessibility is an accessibility-floor commitment, not just a brand-voice choice.** Plain language over jargon (see Voice and Tone) and one primary action per screen (see `DESIGN.md` Do's and Don'ts) are cross-referenced here explicitly so a future brand refresh can't quietly regress an accessibility guarantee while "just" updating tone.
- Non-color-alone: Emergency Badge and Verified Badge always render their label text, never fill color alone (see `DESIGN.md.Components`) — relevant given Coral and Emergency Red sit close in hue for protanopia/deuteranopia.

## Responsive & Platform

| Breakpoint (Contractor dashboard) | Behavior |
|---|---|
| `≥ lg` (1024px+) | Job Queue as a wide single column, max 1200px content width — never a multi-column table (deliberate departure from Jobber/ServiceTitan-style dense grids, see `DESIGN.md`). |
| `md` (768–1023px) | Same single-column queue, tighter margins. |
| `< md` (`sm`, phone browser) | Full-width cards, nav collapses to a bottom bar. This is a first-class target, not a fallback — Devon-style Admin/Dashboard Users check the queue on a phone browser between jobs (PRD UJ-3). |

Homeowner app is native mobile only — no responsive web equivalent in MVP scope.

## Key Flows

### UJ-1 — Maria gets a same-day diagnosis for a leaking water heater

1. Maria opens the app (unauthenticated), lands on Home.
2. Taps "Diagnose my problem" → Guided Assessment.
3. Captures photos + a short video of the leak, answers clarifying questions, enters a rough budget.
4. AI Diagnosis Card renders inline, low seconds: "likely a failed pressure relief valve or tank corrosion," Misdiagnosis Disclaimer immediately beneath it, Teal confidence line.
5. Contractor Match surface shows a single verified Contractor with a one-line "why this contractor."
6. **Climax:** She taps Book. Booking Confirmation shows the visit time; Contact-Reveal Event fires; she closes the app knowing exactly what's wrong and who's coming, without a single competing choice along the way.

Edge case: low-detail photos → re-prompt for one more angle before the AI Diagnosis Card renders, not after (see State Patterns). If the matched Contractor becomes unavailable between Contractor Match and Booking Confirmation, Maria sees a brief explanation and is automatically re-matched rather than dropped back to Home (see State Patterns: Matched Contractor unavailable).

### UJ-2 — James skips straight to booking an electrician he's used before

1. James opens the app (authenticated, returning), lands on Home.
2. Taps "I know what I need" → Quick Submit.
3. Types a one-line description ("outlet replacement"), picks a time window.
4. **Climax:** Contractor Match appears immediately — no diagnosis screen, no disclaimer, no detour. He books in three taps total from app open.

### UJ-3 — Devon (contractor/admin) clears the morning queue in under five minutes

1. Devon opens the dashboard on his phone browser between jobs, lands on Job Queue.
2. Scans collapsed Lead Cards — quality signal, trade, urgency visible with zero taps; an Emergency Badge card sits at the top, inserted ahead of older Leads.
3. Taps a card to expand; reviews photos, AI Diagnosis, budget in place.
4. Taps Accept.
5. **Climax:** The card moves to Active Jobs, Contact-Reveal fires, and Devon is back to the next card — two taps, no navigation away from the queue at any point.

Edge case: Devon on Tier 3 with a dedicated Admin seat — the Admin User sees and works the identical queue, same mechanism, different login.

### UJ-4 — Priya envisions her kitchen remodel before contacting anyone

1. Priya opens the app, lands on Home, enters the Improvement/Renovation path (routed via her described want, not a diagnosed problem).
2. Submits photos of her current kitchen and a short description ("open up the island, lighter cabinets").
3. **Climax:** Visualization Result renders — an AI-generated concept of the change in her actual kitchen, clearly labeled as a concept, not a binding design or quote. She attaches it to her Lead.

Edge case: she skips waiting for the visualization — Lead still submits with just photos and description; visualization is additive, never a gate.
