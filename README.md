# Health Cloud Delivery Architect

A reusable **Cursor skill** that acts as a Salesforce **Delivery / Solution
Architect** for **Health Cloud** — and, uniquely, works **prototype-first**:
it can hand a Product Owner a *clickable, Salesforce-grounded HTML mockup* + a
lightweight *Solution Plan* **before** any user story is written, then promote
the plan into a full epic when the PO signs off.

It is the **Health Cloud sibling** of the
[`lsc-delivery-architect`](https://github.com/p-kothapalli/lsc-delivery-architect)
skill (Life Sciences Cloud vertical). Same STEP 0–6 workflow, same three-artifact
model, same hard blockers; the vertical, persona cheatsheet, object model, and
integration mode (HL7 v2 / FHIR R4 / EHR instead of SAP Concur / Veeva CRM)
are Health-Cloud-specific.

---

## Prototype-first, story-second

Traditional user-story generators force this order:

> _clarify → write story → argue about ACs → maybe build a mockup → build_

For most Health Cloud engagements that's the wrong order. Product Owners consistently
ask two questions **before** they're ready to sign off on Given/When/Then ACs:

1. **"What will this actually look like on-screen?"**
2. **"Can we build it with OOTB Lightning, or do we need LWC / Flow /
   OmniStudio?"**

The Health Cloud Delivery Architect answers both **first**, and only writes
stories once the PO has walked through the mockup and agreed to the shape.

### The three artifacts

| # | Artifact | When | Contract |
|---|----------|------|----------|
| **1** | **Solution Plan** | *"Plan this feature — no stories yet"* | Build-technology decision (declarative-first per RULE 7a), Component Inventory with badges + effort sizing, open questions. **No** ACs, **no** Pattern E — deliberately lightweight. |
| **2** | **Grounded HTML prototype** | *"Show me the built feature"* | Single self-contained `.html`, SLDS 2-flavoured, clickable state toggler, **every element badged** with the Salesforce component that will implement it (`OOTB` / `Config` / `Flow` / `LWC` / `OS` / `FC` / `IP` / `AL` / `Apex` / `Ext`), Build-Technology banner at top, component legend at bottom. |
| **3** | **Implementation-ready user story** | *"Promote this plan to stories"* — or start here | Concrete Health Cloud persona, Given/When/Then ACs in business language, **Pattern E** per-field record spec on every write, **RULE 16 PHI/HIPAA audit AC**, Technical Implementation table, Definition of Done, Estimated Effort. |

Any one artifact, any combination, or all three — driven by the STEP 0 workflow
mode (`Plan + Prototype`, `New Feature`, `Refactor`, `Epic Breakdown`,
`Bug Fix`, or `Legacy → HC Migration`).

---

## Grounded prototype hard blockers (§6.7)

A prototype produced by this skill MUST:

1. **State the Build Technology at the top** — the chosen tech + rationale + rejected alternative (declarative-first per RULE 7a).
2. **Label every interactive element** with the Salesforce component that will implement it — Lightning record page + Dynamic Actions, Screen Flow screen, LWC, OmniScript step, FlexCard, Action Launcher, Quick Action, related list, Path, etc.
3. **Distinguish OOTB vs. custom visually** — coloured badges per component type with hover-tips explaining the "why".
4. **Match the story's ACs and Pattern E field spec** (once stories exist) — every happy-path AC reachable in the click-through; forms show every field in the Pattern E table.
5. **Be a single self-contained `.html` file** — inline CSS, no external fonts/JS/CDN dependencies.
6. **Use SLDS 2-flavoured styling** so the mockup reads as *Salesforce*, not a generic web app.
7. **Include a Build-Technology legend footer** — component inventory with effort sizes mirroring the Solution Plan / story.
8. **Never invent component or field names** — custom components verified via `code-review-graph`, standard Health Cloud objects/fields via `salesforce-docs`; unverifiable names marked `(proposed)`.

A prototype without the grounding is worse than no prototype — it misleads the
PO on cost shape. That's the whole point.

---

## Customer-brand skinning (RULE 17) — new in v1.1.0

When the skill runs inside a customer workspace it recognises, it **auto-applies
that customer's brand skin** to the §6.7 prototype: colour tokens, typography,
product terminology, and the customer's mandatory legal footer are overlaid on
top of the SLDS 2 baseline (never in place of it). The PO sees a mockup that
looks like it came from *their* brand system on day one.

### Supported brands (v1.1.0)

| Brand | Auto-detected from | Reference |
|-------|--------------------|-----------|
| **Insulet / OmniPod** | Workspace path or repo name contains `insulet` / `omnipod` / `podder`; `sfdx-project.json` mentions Insulet; `.cursor/rules/insulet-*` file; prompt mentions `OmniPod`, `Podder`, `PDM`, `SmartAdjust`, `SmartBolus`, `PodderCentral`; or explicit *"apply the Insulet skin"* | [`insulet-omnipod-brand.md`](.cursor/skills/healthcloud-delivery-architect/references/insulet-omnipod-brand.md) |

The Insulet reference carries the OmniPod colour palette extracted from the
live omnipod.com CSS (grape `#743DBC`, sunlight `#FFA700`, coral `#F75E4C`,
info-teal `#1AD1DB`, warm neutrals), typography (IBM Plex Sans + Open Sans),
an OmniPod → Health Cloud terminology cross-walk (Podder → Person Account,
Pod → Asset, PDM → Asset, Pump alarm → Case, Re-supply → Order), a
ready-to-paste CSS overlay, and the mandatory legal footer (Safety Info + HIPAA
+ Customer Support 1-800-591-3455).

### Adding a new brand

Drop a `references/<org>-brand.md` file following the same structure as the
Insulet reference, and add a row to the supported-brands table in the trigger
rule. The skill treats every brand file identically: CSS overlay + terminology
+ mandatory footer, on top of SLDS 2.

---

## What it produces (story mode)

Every generated story follows one contract:

- **Concrete Health Cloud persona** — a real HC role (Care Coordinator, Care
  Manager, Nurse Case Manager, Patient Services Representative, Utilization
  Reviewer, Medical Director, Network Manager, RPM Nurse), never *"the user"*.
  Patients / Caregivers / Providers are the *subjects* of the work, not the
  login persona (unless it's genuinely an Experience Cloud portal story).
- **Given / When / Then** ACs in business language — no Apex class names, IP
  step numbers, SOQL, or `*__c` API names inside a GWT line.
- **Pattern E per-field record spec** on every "records created/updated"
  outcome — every field enumerated, no "etc.".
- **RULE 16 — PHI / HIPAA audit AC** on every story that displays, logs,
  exports, or sends PHI, plus a permission-set / FLS spec (Pattern C).
- **`## Technical Implementation (high-level)`** section after the ACs —
  a concise table naming components, change type, and a one-line note.
- **Definition of Done** and a **Clarification Questions** table for unknowns.
- **Estimated Effort** — component-level sizing (S / M / L / XL / XXL).
- **Grounded components** — custom verified via `code-review-graph`, standard
  Health Cloud via `salesforce-docs`; proposals flagged when the Health Cloud
  package isn't deployed in the target workspace.

---

## What it knows (Health Cloud knowledge pack)

- **Health Cloud sub-domains** — Care Management (provider + payer), Patient
  Services / Contact Center, Utilization Management, Provider Network Ops,
  Member 360, Home Health / Remote Patient Monitoring.
- **Health Cloud object model** — `Account` (Patient Person Account, Caregiver
  Contact, HealthcareProvider, HealthcareFacility, Payer), `Individual`,
  `CarePlan` / `CarePlanTemplate` / `CarePlanGoal` / `CarePlanActivity`,
  `CareRequest` / `CareRequestReview` / `CareRequestItem` for Utilization
  Management, `Case` with Patient Services / Appeals / Refill record types,
  `ContactEncounter` / `VoiceCall` for the contact center, `CareObservation` /
  `CareRegisteredDevice` for RPM, `HealthcareIndividualEnrollment` +
  `MemberPlan` + `PurchaserPlan` for Member 360, `AuthorizationFormConsent` for
  consent-driven outbound. Full catalog in
  [`healthcloud-standard-objects-catalog.md`](.cursor/skills/healthcloud-delivery-architect/references/healthcloud-standard-objects-catalog.md).
- **Integration knowledge** — HL7 v2 (ADT/ORU/MDM via MuleSoft), FHIR R4 native
  endpoints, eligibility (270/271), claims (837/835), Service Cloud Voice for
  telephony, MuleSoft for provider-roster ingestion.
- **HIPAA / PHI hard blocker (RULE 16)** — every story that touches PHI carries
  an audit AC + FLS/perm-set spec. Not optional.
- **Declarative-first build-technology decision guide (RULE 7a)** — prefer OOTB
  Lightning + Dynamic Actions / Screen Flow before OmniScript; OmniStudio is
  one option, not the default. Standard vs. managed-package runtime call-out.
- **SLDS 2 primer** — token-driven design system reference for prototypes and
  LWC work, with an HC-specific worked example (Log Patient Call).
- **Legacy → Health Cloud migration mode** — Salesforce Care Cloud legacy,
  custom Force.com patient-services, third-party CRM → native HC.

---

## Install

The skill lives under `.cursor/`, which Cursor auto-loads.

### Project-level (per repository) — recommended

```bash
# From the root of THIS repo, copy the .cursor payload into your target repo:
cp -R .cursor /path/to/your-project/

# or clone and copy:
git clone https://github.com/p-kothapalli/healthcloud-delivery-architect.git
cp -R healthcloud-delivery-architect/.cursor /path/to/your-project/
```

Reload the Cursor window. It activates automatically.

### User-level (available in every workspace)

```bash
cp -R .cursor/skills/healthcloud-delivery-architect ~/.cursor/skills/
cp    .cursor/rules/use-healthcloud-delivery-architect.mdc ~/.cursor/rules/
```

(Confirm the exact user-skills path in **Cursor → Settings → Rules &
Skills**.)

---

## Use

In a repo with the skill installed, just ask in natural language. The skill
routes to the right workflow mode via STEP 0.

### Prototype-first prompts (the new way — no stories yet)

- *"Plan and prototype `<capability>`."*
- *"Solution plan for `<capability>` — no stories yet."*
- *"Prototype-only for `<capability>` — we'll do stories after PO sign-off."*
- *"Just show me what the built feature would look like and what it costs."*

**Examples for a med-device patient-services team (Insulet OmniPod pattern):**

- *"Plan and prototype: Care Coordinators need a better way to log and route patient calls."*
- *"Plan and prototype a utilization-management workflow for prior-authorization triage."*
- *"Solution plan for referral routing from PCP to endocrinologist — no stories yet."*

The skill produces:

1. `requirements/<Capability>_SolutionPlan.md`
2. `requirements/<Capability>_Prototype.html`

…then ends with **"Promote to Stories?"** — say the word and it switches to
Epic Breakdown mode and expands the plan into full user stories with ACs,
Pattern E, RULE 16 PHI audit, Technical Implementation, DoD, and QTA test
prompts.

### Story-first prompts

- *"Write a Health Cloud user story for `<capability>`."*
- *"Break this Health Cloud scope doc into stories."*
- *"Create a story to migrate `<legacy Care Cloud feature>` to Health Cloud."*
- *"Fix HC Story: defect INQ-123 in `HC_LogPatientCall` — current: …, expected: …"*

### Post-generation offers (STEP 6)

Every generated artifact ends with a set of offers, gated by MCP availability:

- **6.1** QTA test-bridge prompts (once ACs exist)
- **6.2** Story / epic dependency diagram (udd-whiteboard / figma / Mermaid)
- **6.3** GUS work item creation
- **6.4** Salesforce Docs verification
- **6.5** Health Cloud Librarian notebook lookup (NotebookLM)
- **6.6** Story dependency check across `requirements/`
- **6.7** Grounded HTML prototype — first-class in Plan + Prototype mode,
  optional after a story

---

## Verify it loaded

Type either of these and confirm the skill activates:

- *"Plan and prototype a test capability — no stories yet"* → should route to
  Plan + Prototype mode and ask Phase 1 + Phase 2 clarifying questions only.
- *"Write a Health Cloud user story for a test field"* → should ask 5–16 HC
  clarifying questions via a structured multiple-choice picker.

---

## Optional: component grounding via MCP

For full structural grounding (caller/dependent/test context), configure the
`code-review-graph` MCP server in your `.cursor/mcp.json`. For standard Health
Cloud object/field/feature verification with citations, configure the
`salesforce-docs` MCP. Without either, the skill still works — it falls back
to file search + curated references and clearly marks unverifiable names as
`(proposed)`.

---

## Repository layout

```
.cursor/
  skills/healthcloud-delivery-architect/
    SKILL.md                    # the skill (navigational overview, STEP 0–6)
    references/
      ac-pattern-library.md              # Persona contract + AC Patterns A–E (with HC personas)
      output-template.md                 # Full story template + effort sizing
      plan-prototype-mode.md             # Plan + Prototype (no stories) contract
      post-generation-offers.md          # STEP 6 detail — including §6.7 prototype
      healthcloud-object-model.md        # Curated HC data model + acronyms
      healthcloud-standard-objects-catalog.md  # Complete HC standard objects catalog
      healthcloud-components.md          # Build-tech decision guide + component conventions
      slds2-healthcloud-primer.md        # SLDS 2 tokens + HC worked example
      insulet-omnipod-brand.md           # Insulet / OmniPod brand skin (v1.1.0)
      story-examples.md                  # Worked exemplars (Patterns A–E)
    evaluations/                         # Eval scenarios + rubric
  rules/
    use-healthcloud-delivery-architect.mdc   # Trigger rule
README.md
```

---

## Sibling / lineage

- **Sibling for pharma / medtech field:** [`lsc-delivery-architect`](https://github.com/p-kothapalli/lsc-delivery-architect) — Salesforce Life Sciences Cloud (Visits, Sample Management, Managed Events, SAP Concur expenses, Veeva CRM migration, MSL Medical Inquiry).
- **Sibling for PNM / credentialing:** [`user-story-architect`](https://github.com/p-kothapalli/lsc-user-story-architect) predecessor lineage.

If your company touches both — e.g., a device manufacturer that runs LSC for
field-sales commercial and Health Cloud for patient services — install **both**
skills. The trigger rules will route each request to the right skill by
vocabulary (patient/care team → this skill; HCP/MSL/sample → LSC skill).

---

## Version

Current: **v1.1.0** — Adds `RULE 17` (org-brand skinning) and the first
supported brand overlay, [Insulet / OmniPod](.cursor/skills/healthcloud-delivery-architect/references/insulet-omnipod-brand.md).
v1.0.1 (2026-08-10) scrubbed LSC template residue surfaced by the end-to-end
sanity test. v1.0 (2026-08-10) was the initial Health Cloud fork of
`lsc-delivery-architect` v1.8. See the Version History table in
[`SKILL.md`](.cursor/skills/healthcloud-delivery-architect/SKILL.md) for the
full change log.
