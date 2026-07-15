---
name: AI-Powered Lead Intake Platform for Contractors
description: Fun, welcoming, confident home-services matching platform. AI-guided diagnosis for homeowners, low-click job queue for contractors — one identity across native mobile and web.
status: final
created: 2026-07-14
updated: 2026-07-14
sources:
  - ../../prds/prd-maitence-app-2026-07-14/prd.md
  - ../../prds/prd-maitence-app-2026-07-14/addendum.md
  - ../../../../docs/Property_Trades_Taxonomy.md
colors:
  surface-base: '#FFF9F2'
  surface-raised: '#FFFFFF'
  ink-primary: '#23252B'
  ink-secondary: '#6B6558'
  ink-disabled: '#B0A99C'
  primary: '#C2410C'
  primary-foreground: '#FFFFFF'
  trust: '#0B7A70'
  trust-foreground: '#FFFFFF'
  alert-emergency: '#C43A2E'
  alert-emergency-foreground: '#FFFFFF'
  border-hairline: '#ECE6DA'
  surface-sunken: '#F5EFE3'
  focus-ring: '#0B7A70'
typography:
  display:
    fontFamily: 'Plus Jakarta Sans'
    fontSize: 28px
    fontWeight: '700'
    lineHeight: '1.2'
    letterSpacing: -0.01em
  title:
    note: 'Plus Jakarta Sans on both surfaces — iOS/Android load as a bundled custom font, matching web weight/size at Title 1 / Headline Small equivalents.'
  body:
    fontFamily: 'Inter'
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.5'
  meta:
    fontFamily: 'Inter'
    fontSize: 13px
    fontWeight: '500'
    lineHeight: '1.4'
rounded:
  sm: 8px
  md: 14px
  lg: 20px
  full: 9999px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '5': 24px
  '6': 32px
  '7': 48px
components:
  button-primary:
    background: '{colors.primary}'
    foreground: '{colors.primary-foreground}'
    radius: '{rounded.full}'
  button-secondary:
    background: 'transparent'
    foreground: '{colors.primary}'
    border: '{colors.primary}'
    radius: '{rounded.full}'
  ai-diagnosis-card:
    background: '{colors.surface-raised}'
    radius: '{rounded.lg}'
    accent: '{colors.trust}'
  disclaimer-banner:
    background: '{colors.surface-sunken}'
    foreground: '{colors.ink-secondary}'
    radius: '{rounded.md}'
  lead-card:
    background: '{colors.surface-raised}'
    radius: '{rounded.md}'
    border: '{colors.border-hairline}'
  emergency-badge:
    background: '{colors.alert-emergency}'
    foreground: '{colors.alert-emergency-foreground}'
    radius: '{rounded.full}'
  verified-badge:
    background: '{colors.trust}'
    foreground: '{colors.trust-foreground}'
    radius: '{rounded.full}'
  tier-badge:
    background: '{colors.surface-sunken}'
    foreground: '{colors.ink-primary}'
    radius: '{rounded.full}'
  trade-service-picker:
    background: '{colors.surface-raised}'
    radius: '{rounded.md}'
    chip-selected: '{colors.trust}'
    chip-selected-foreground: '{colors.trust-foreground}'
  no-match-panel:
    background: '{colors.surface-sunken}'
    foreground: '{colors.ink-secondary}'
    radius: '{rounded.md}'
  budget-realism-note:
    background: '{colors.surface-sunken}'
    foreground: '{colors.ink-secondary}'
    radius: '{rounded.sm}'
  cold-lead-toggle:
    foreground-active: '{colors.ink-primary}'
    foreground-inactive: '{colors.ink-secondary}'
---

## Brand & Style

This platform exists to replace two bad feelings — a homeowner's anxiety about not knowing what's wrong or who to trust, and a contractor's overwhelm from running a business solo — with confidence and ease. The brand follows directly: **fun, welcoming, confident**, and above all *easy* — an interface that never makes either side feel like they're filling out a form or reading a disclaimer novel, even when the content (a misdiagnosis disclaimer, a budget-realism flag, a verification gate) is inherently serious.

The visual language leans warm and rounded rather than clinical — this is not a sterile SaaS dashboard or an insurance-form aesthetic, both of which the incumbent lead-gen marketplaces lean toward. A confident coral carries every primary action (the "yes, let's go" color); a grounded teal is reserved exclusively for trust and verification signals, so it reads as *earned*, not decorative. Generous rounding (soft cards, pill buttons) signals approachability without tipping into whimsy — this is still a product people use when their pipe is leaking.

One identity serves both surfaces. The homeowner's native mobile app and the contractor's web dashboard share every token in this file; the difference between "welcoming intake" and "efficient queue" is expressed through layout density and copy (see `EXPERIENCE.md`), not through a different color story.

## Colors

Every foreground/background pairing below is stated with its computed WCAG contrast ratio (relative-luminance formula, not eyeballed) against the AA floor `EXPERIENCE.md` commits to: 4.5:1 for normal text, 3:1 for large text (≥18pt/24px or ≥14pt/19px bold) and non-text UI component boundaries. This table is the source of truth — if a component's spec elsewhere implies a pairing not listed here, treat it as unverified and check before shipping. `primary`, `trust`, and `alert-emergency` are all darker than their first-draft values for the same reason: each original computed under 4.5:1 white-on-fill and failed AA outright on a load-bearing signal (the primary CTA, the Verified Badge, the Emergency Badge). Darkening was the fix in every case, not a copy or icon change — treat "looks distinct enough" as insufficient for any future color decision here.

- **Ivory (`#FFF9F2`)** — `surface-base`. Warm, not stark-white — the app should feel like a well-lit room, not a hospital form.
- **White (`#FFFFFF`)** — `surface-raised`. Cards, sheets, and the composer sit here.
- **Charcoal-Navy (`#23252B`)** — `ink-primary`. Primary text. ~16.7:1 on `surface-base`. Deliberately not pure black — softer, warmer, consistent with the "welcoming" brief.
- **Warm Gray (`#6B6558`)** — `ink-secondary`. ~5.5:1 on `surface-base`, ~5.05:1 on `surface-sunken`. Metadata, timestamps, disclaimers, helper text — passes AA normal-text at both.
- **Coral (`#C2410C`)** — `primary`. The single "go" color. ~5.18:1 in both directions it's used (white-on-fill for button-primary; as text/border on `surface-base`/`surface-raised` for button-secondary). Never used for anything but the primary CTA — not icons, not decoration, not state.
- **Teal (`#0B7A70`)** — `trust`. ~5.20:1 white-on-fill. Reserved exclusively for verification and trust signals: the Verified Contractor badge, the AI Diagnosis confidence indicator, the Contact-Reveal confirmation. If it isn't a trust claim, it isn't teal.
- **Emergency Red (`#C43A2E`)** — `alert-emergency`. ~5.26:1 white-on-fill. Reserved exclusively for Emergency Work Type Leads (PRD FR-14). Always paired with the word "Emergency," not the fill alone — Coral and Emergency Red sit close enough in hue that protanopia/deuteranopia can't reliably tell them apart by color.
- **Hairline (`#ECE6DA`)** — `border-hairline`. ~1.1:1 against `surface-base`/`surface-raised` — a decorative accent only (list-row dividers), never the sole signal for a meaningful UI boundary. See Elevation & Depth for how card boundaries are actually made legible.
- **Sunken Ivory (`#F5EFE3`)** — `surface-sunken`. Disclaimer banners, budget-realism notes, and tier badges — a shade recessed from the base, never used for actionable surfaces.
- **Focus Ring (`#0B7A70`, same value as `trust`)** — `focus-ring`. Visible focus indicator on every interactive element on the web dashboard, satisfying WCAG 2.4.7 explicitly — never suppressed for aesthetic reasons, even on pill-shaped buttons/badges.

Avoid: gradients (flattens the confident-but-simple brief into something that reads as a pitch deck), more than these three chromatic colors, Teal or Emergency Red used decoratively, and any new color pairing introduced downstream without computing its contrast ratio first — the three colors above were darkened specifically because "looks distinct enough" was not a reliable substitute for the actual formula.

## Typography

Two-role system across both surfaces: **Plus Jakarta Sans** (Display/Title — rounded terminals, confident without being playful-childish, carries the brand's "fun and confident" personality at headline moments) and **Inter** (Body/Meta — highly legible workhorse for everything else, on native mobile and web alike).

`display` (28px/700) is reserved for rare, high-value moments: the AI Diagnosis headline result, the "Welcome back" greeting, empty-state hero text. `title` covers section and card headers. Everything else — descriptions, form labels, lead-card content, dashboard tables — is `body` or `meta`. No all-caps labels; the brand doesn't shout.

Dynamic type / OS text scaling honored on mobile; the web dashboard respects browser zoom without breaking layout at 200%.

## Layout & Spacing

Scale: 4 / 8 / 12 / 16 / 24 / 32 / 48px. Mobile intake flows use generous spacing (`{spacing.5}`–`{spacing.6}` between major steps) — this is where "doesn't overwhelm" is most load-bearing, since a homeowner is often mid-crisis. The Contractor Job Queue is denser by necessity (PRD FR-12/13's low-click, scan-fast requirement) but never drops below `{spacing.3}` between card elements — density comes from card count on screen, not from cramming a single card.

Mobile margins follow platform convention (iOS 16pt / Android 16dp). Web dashboard max content width 1200px on `lg+`; the queue is a single scrollable column of cards, never a dense data table — this is a deliberate departure from incumbent contractor-CRM aesthetics (Jobber, ServiceTitan), which the PRD's own addendum flags as competing for the same budget/attention.

## Elevation & Depth

Restrained, but not tone-only. The `surface-raised`/`surface-base` tone shift and the `{colors.border-hairline}` border both compute to near-zero contrast (~1.1:1) — real enough for a sighted, non-impaired reader in good light, not reliable as the sole boundary signal for a low-vision user, and not defensible against WCAG 1.4.11 for a boundary that's functionally load-bearing (a Contractor scanning a Job Queue needs to see where one Lead Card ends and the next begins). Every card (Lead Card, AI Diagnosis Card, Disclaimer Banner) therefore carries a subtle default shadow — low-opacity, short-blur, present but not heavy — in addition to the tone shift and hairline. A second, slightly stronger shadow level is reserved for the active/expanded Lead Card and any modal or bottom sheet, signaling "this is temporarily on top." Two levels total; nothing heavier.

## Shapes

`{rounded.sm}` (8px) for inputs and small controls. `{rounded.md}` (14px) for cards — Lead Card, AI Diagnosis Card, Disclaimer Banner. `{rounded.lg}` (20px) for sheets, modals, and the primary AI Diagnosis result card, which should feel like the single most important surface in the homeowner flow. `{rounded.full}` for every button and every badge (Emergency, Verified, Tier) — pills read as approachable and tappable, and give badges a consistent "chip" language across both surfaces.

## Components

- **Button (primary)** — `{colors.primary}` fill, `{rounded.full}`, visible `{colors.focus-ring}` on keyboard/switch focus (never suppressed). One per screen wherever possible; this is the "go" affordance and it should never compete with itself.
- **Button (secondary)** — Coral outline, transparent fill, `{rounded.full}`, same focus-ring rule. Used for "Skip AI" / "not now" / secondary paths — never a second filled button next to a primary one.
- **AI Diagnosis Card** — `{rounded.lg}`, `surface-raised`, default shadow (see Elevation & Depth). Houses the AI Diagnosis or Condition Assessment result plus the Misdiagnosis Disclaimer inline beneath it (never a separate screen the Homeowner must navigate to). Teal accent marks the confidence/verification line only.
- **Disclaimer Banner** — `surface-sunken`, `{rounded.md}`, `ink-secondary` text, default shadow. Always visible with its parent result, never collapsible to zero height, never dismissible permanently (PRD FR-1).
- **Lead Card** — `{rounded.md}`, hairline border, default shadow (the shadow, not the hairline alone, is what makes the boundary legible — see Elevation & Depth). Collapsed state shows lead-quality signal, trade/service label, and urgency (Emergency Badge if applicable) with no tap required; expands in place, never navigates away (PRD FR-12). Expand/Accept controls are real focusable elements (button semantics, not a div with a click handler) so switch-access and voice-control users on a phone browser reach them the same way keyboard users do.
- **Emergency Badge** — `{colors.alert-emergency}` pill. Always renders the word "Emergency," not fill color alone — this is the non-color-alone signal the Coral/Emergency-Red hue proximity requires. Appears only on Emergency Work Type Leads, top-left of the Lead Card, always visible even collapsed.
- **Verified Badge** — `{colors.trust}` pill. Always renders the word "Verified," not fill color alone. Contractor profile and Job Queue header only — never on an unverified account (PRD FR-11 gate).
- **Tier Badge** — `surface-sunken` pill, neutral ink, renders the Tier name as text. Deliberately unobtrusive relative to Verified/Emergency — Tier is an account attribute, not a trust or urgency claim, and should never visually imply either. This is the *only* place Tier appears in the product; Matching Priority (the ranking mechanic Tier confers, PRD FR-18) has no visual representation anywhere — see `EXPERIENCE.md` for why.
- **Trade/Service Picker** — `{rounded.md}` card, `surface-raised`. Step 1: Trade selection as a short checklist (Phase-1 subset only — see `EXPERIENCE.md` Foundation for MVP scope). Step 2, per selected Trade: a search input plus checklist of that Trade's Services; selected Services render as `{colors.trust}`-filled chips (reusing the Verified Badge's trust color to signal "confirmed," a deliberate visual echo — this is the one place `trust` marks a selection rather than a claim about the Contractor).
- **No Match Yet panel** — `surface-sunken`, `{rounded.md}`, `ink-secondary` text, same visual family as Disclaimer Banner — both are "here's an honest status update" surfaces.
- **Budget Realism Note** — `surface-sunken`, `{rounded.sm}` (smaller than the panel-level components — this is an inline field-level hint, not a standalone card).
- **Cold Lead filter toggle** — Text-only two-state control (active/inactive ink-primary vs. ink-secondary), no fill, no icon — deliberately the lowest-emphasis control on the Job Queue header, consistent with Cold Leads being deprioritized, not hidden.

## Do's and Don'ts

| Do | Don't |
|---|---|
| One filled primary button per screen, always Coral | Multiple competing filled CTAs |
| Teal exclusively for trust/verification claims | Teal as a generic "success" or decorative color |
| Disclaimer always inline with its AI result, same card | Disclaimer as a separate screen, tooltip, or dismissible modal |
| Emergency Red reserved for Emergency Work Type only, always paired with the word "Emergency" | Emergency Red for generic warnings or errors, or color alone with no text |
| Generous spacing in the homeowner intake flow | Dense, form-like layouts during diagnosis |
| Tier Badge visually neutral, never trust-coded; Matching Priority stays invisible everywhere | Tier Badge in Coral or Teal, or any surface stating "upgrade to rank higher" |
| Pill shapes for every button and badge | Sharp-cornered buttons or square badges |
| Compute and state the contrast ratio before introducing any new color pairing | Assume a color pairing is "distinct enough" without running the formula |
| Visible focus ring on every interactive element, including pills | Suppressed focus outlines for aesthetic reasons |
