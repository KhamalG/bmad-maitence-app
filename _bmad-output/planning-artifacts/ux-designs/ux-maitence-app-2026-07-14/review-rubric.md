# Spine Pair Review — AI-Powered Lead Intake Platform for Contractors

## Overall verdict

The pair is a well-disciplined, source-anchored spine — sources resolve, UJ names are verbatim, section shape matches canonical order exactly, and prose stays lean (tables over paragraphs, editorial voice confined to DESIGN.md as intended). It is not yet a clean contract, however: one committed color pairing fails the WCAG AA floor the spine itself asserts, four components have behavioral specs with no visual counterpart (one of them a gating onboarding component), and the Trade/Service Picker's stated scope directly contradicts the PRD's explicit MVP boundary. A downstream consumer can extract most of this cleanly, but would silently build a wrong-scoped picker and an inaccessible primary button without independent verification.

## 1. Flow coverage — adequate
Checked: all 4 PRD UJs (UJ-1–UJ-4, §2.3) against EXPERIENCE.md § Key Flows for named protagonist, numbered steps, climax, and failure/edge path.
### Findings
- **medium** UJ-1's Key Flow (EXPERIENCE.md lines 132–141) has an edge case only for low-detail photos, not for the climax step itself — no failure path for "Booking Confirmation" if the matched Contractor becomes unavailable between match and booking. State Patterns table also has no row for this. *Fix:* add a Booking Confirmation failure state and a one-line edge case in UJ-1.
- **low** UJ-2's Key Flow (EXPERIENCE.md lines 143–148) has no failure/edge path at all. This mirrors the PRD (UJ-2, §2.3, also has none), so it's an inherited gap, not an EXPERIENCE.md-introduced one — flagging for awareness only.

## 2. Token completeness — thin
Checked: every color/typography/rounded/spacing/component token in DESIGN.md frontmatter (lines 11–90) against every `{path.to.token}` reference in DESIGN.md prose. All tokens are defined; all color tokens carry hex; `title` stays semantic per platform-convention allowance. No dangling references found.
### Findings
- **critical** `primary` (`#FF6B4A`) / `primary-foreground` (`#FFFFFF`) — the button-primary pairing (DESIGN.md lines 58–61, 139) — computes to ~2.8:1 contrast, well under the 4.5:1 WCAG 2.1 AA text threshold that EXPERIENCE.md's Accessibility Floor commits to for "both surfaces" (EXPERIENCE.md line 114). This is the single most-used interactive element under the spine's own "one filled primary button per screen" rule (DESIGN.md line 139, Do's/Don'ts line 152) — every Get Diagnosis / Book Now / Accept Lead CTA inherits the failure. *Fix:* darken `primary` or specify a darker text/fill pairing for button-primary specifically; re-verify against AA before this ships to Architecture.
- **medium** No contrast ratios are stated anywhere in DESIGN.md despite EXPERIENCE.md explicitly deferring "Visual contrast lives in DESIGN.md" (lines 112, 84 in EXPERIENCE.md). Spot-checking two more load-bearing pairs: `trust`/`trust-foreground` (~3.4:1) and `alert-emergency`/`alert-emergency-foreground` (~4.1:1) both sit under 4.5:1 for text-sized use (Verified Badge, Emergency Badge labels). *Fix:* add an explicit contrast line per component-token color pairing, not just hex values.

## 3. Component coverage — thin
Checked: every component name in DESIGN.md § Components (8 rows) against every component name in EXPERIENCE.md § Component Patterns (10 rows).
### Findings
- **high** Trade/Service Picker (EXPERIENCE.md line 83) — a first-run, verification-gating onboarding component with real structural complexity (search-first, Trade-scoped, hierarchical) — has no DESIGN.md row: no radius, no color, no card treatment specified. Downstream has to invent its look from scratch. *Fix:* add a Trade/Service Picker row to DESIGN.md § Components.
- **medium** No Match Yet panel, Budget Realism Note, and Cold Lead filter toggle (EXPERIENCE.md lines 78, 80, 82) each have behavioral rules but no DESIGN.md visual counterpart (no rounding, color, or elevation assigned). *Fix:* add rows for these three, even if minimal (e.g., inherits `disclaimer-banner` treatment).
- **low** DESIGN.md's "Tier Badge" (line 87) is referenced in EXPERIENCE.md under a differently-named row, "Tier / Matching Priority" (line 85), which conflates the visual badge with the backend ranking mechanic. Same underlying component, two names across the pair. *Fix:* rename the EXPERIENCE.md row to "Tier Badge" and note Matching Priority as a related-but-separate backend concept inline, or split into two rows.

## 4. State coverage — thin
Checked: every Homeowner and Contractor/Admin IA surface (EXPERIENCE.md § Information Architecture) against expected states (empty, cold-load, focus, error, offline, permission-denied) in § State Patterns.
### Findings
- **medium** No offline state anywhere in the document, for either surface — notable given the Homeowner persona is explicitly "often mid-crisis" (DESIGN.md line 125) and Guided Assessment requires photo/video capture, a connectivity-sensitive action. *Fix:* add an offline row (e.g., allow local capture, queue submission, or surface an honest "reconnect to submit" state).
- **medium** No camera/photo permission-denied state, despite FR-1's hard requirement of "at least one photo before AI Diagnosis is attempted" (PRD line 109) making this a directly reachable dead end. *Fix:* add a permission-denied row to State Patterns.
- **medium** No generic AI-service-error state (distinct from the already-covered "low-confidence media" case) — what renders if the AI Diagnosis call fails outright rather than returning low confidence. *Fix:* add an error row alongside the existing "Low-confidence media" and "AI Diagnosis pending" rows.
- **low** No search-empty state for the Trade/Service Picker (contrast with the reference examples, which cover "Search empty" / "Command palette no matches" explicitly). *Fix:* add a row once the Picker's visual spec (Finding #3) exists.
- **low** Active Jobs and Account/Tier surfaces have no state rows at all (e.g., empty Active Jobs list, distinct from "Empty Job Queue"). Lower stakes than the above since these are steady-state screens, not gating ones.

## 5. Visual reference coverage — n/a (pre-mock)
`mockups/`, `wireframes/` do not exist; `imports/` exists and is empty. EXPERIENCE.md correctly self-reports this at both IA sections ("Composition reference: none yet — see Finalize for key-screen mocks. Spine wins on conflict.", lines 55, and implicitly for the Contractor IA). This is expected pre-mock state, not a defect.

## 6. Bloat & overspecification — strong
DESIGN.md's editorial paragraphs (Brand & Style, Colors "Avoid" notes) are each tied to a specific decision or FR citation, not decorative narrative — consistent with the rubric's allowance for DESIGN.md prose. EXPERIENCE.md stays table-first throughout with no PRD/persona restatement; its `[NOTE FOR UX]` / `[ASSUMPTION]` tags carry real information (e.g., clarifying that native-per-platform is a tech-stack fact, not a UX decision) rather than padding. No pixel values appear where a token would do. No findings.

## 7. Inheritance discipline — thin
Checked: sources frontmatter resolution, UJ-name verbatim match, glossary-term consistency, component-name consistency, and EXPERIENCE.md-to-DESIGN.md reference resolution.
### Findings
- **high** EXPERIENCE.md's Trade/Service Picker description states "the full ~20-Trade catalog is never shown flat" (line 83), implying the picker's data scope is the entire Property Services Taxonomy. This contradicts PRD §6.2's explicit MVP boundary: "Trade/Service catalog beyond the Phase 1 subset ... only the Phase 1 subset is active for matching (FR-6) and verification (FR-11) in MVP." Since the Picker gates FR-11 verification, this could lead Architecture/Dev to build a full ~20-Trade catalog picker when the PRD scopes MVP to 3 Trades (Plumbing/Electrical/HVAC, provisional per §8 OQ1). *Fix:* rephrase to scope the Picker's MVP catalog to the Phase-1 subset, noting the full taxonomy as the long-term (not MVP) data model — matching the PRD's own framing.
- Sources frontmatter (both files, identical 3-item list) resolves correctly: `prd.md`, `addendum.md`, and `docs/Property_Trades_Taxonomy.md` all exist at the referenced relative paths. Good.
- UJ-1 through UJ-4 names are verbatim matches to PRD §2.3. Good.
- Neither spine defines a standalone Glossary section; PRD-Glossary terms (Lead, Job, Cold Lead, Contact-Reveal Event, Tier, Matching Priority, Work Type, etc.) are used consistently with PRD §3 throughout both files, with the exception of the Tier Badge naming noted in Finding #3 (low).
- EXPERIENCE.md contains no `{path.to.token}` syntax (it references DESIGN.md only by section name, e.g. "DESIGN.md.Components"), consistent with both reference examples — nothing to resolve, nothing broken.

## 8. Shape fit — strong
DESIGN.md section order matches canonical exactly: Brand & Style → Colors → Typography → Layout & Spacing → Elevation & Depth → Shapes → Components → Do's and Don'ts. EXPERIENCE.md carries all required defaults (Foundation, IA, Voice and Tone, Component Patterns, State Patterns, Interaction Primitives, Accessibility Floor, Key Flows) in the same order used by both reference examples, with Key Flows correctly last. Responsive & Platform is present and correctly triggered by the multi-surface product. "Inspiration & Anti-patterns" (present in both reference examples) is dropped — defensible, since it's optional rather than a required default, though the addendum's rich competitive material (Jobber/ServiceTitan density, Thumbtack's AI Helper, anti-gamification stance already implicit in "Banned everywhere," line 108) would have supported one well. No findings beyond this soft note.

## Mechanical notes
- All three `sources` entries resolve identically in both files; no broken cross-refs found.
- Component-name consistency is good except: "Tier Badge" (DESIGN.md) vs. "Tier / Matching Priority" (EXPERIENCE.md) — see Finding #3.
- No `{token}` references in either file point to an undefined token; all frontmatter tokens are used or intentionally available (e.g., `ink-disabled` is defined but never referenced in prose — unused, not missing, low-stakes).
- Frontmatter is complete in both files (name/status/created/updated/sources); DESIGN.md additionally carries `description`, which EXPERIENCE.md omits — consistent with both reference examples (EXPERIENCE.md frontmatter is intentionally leaner).
