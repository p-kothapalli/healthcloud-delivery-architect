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

**Walkthrough (GitHub Pages):**
[p-kothapalli.github.io/healthcloud-delivery-architect](https://p-kothapalli.github.io/healthcloud-delivery-architect/)
— four tabs: The Challenge · The Transformation · How It Works · The Proof.

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
| **2** | **Grounded HTML prototype** | *"Show me the built feature"* | Single self-contained `.html`, SLDS 2-flavoured, clickable. Build labels (`OOTB` / `Config` / `Flow` / `LWC` / `OS` / `Apex` / `Ext`) follow `prototypeLabels`: **clean** (hidden), **explicit** (on every element), or **both** (clean screen with a switch). The Solution Plan always records the build decision. |
| **3** | **Implementation-ready user story** | *"Promote this plan to stories"* — or start here | Concrete Health Cloud persona, Given/When/Then ACs in business language, **Pattern E** per-field record spec on every write, **RULE 16 PHI/HIPAA audit AC**, Technical Implementation table, Definition of Done, Estimated Effort. |

Any one artifact, any combination, or all three — driven by the STEP 0 workflow
mode (`Plan + Prototype`, `New Feature`, `Refactor`, `Epic Breakdown`,
`Bug Fix`, or `Legacy → HC Migration`).

---

## Grounded prototype hard blockers (§6.7)

A prototype produced by this skill MUST:

1. **Record the Build Technology** — the chosen tech + rationale + rejected alternative (declarative-first per RULE 7a). On the screen only when labels are on.
2. **Follow `prototypeLabels`** — asked before the prototype is drawn. `explicit`: a badge on every interactive element. `both`: the same badges, hidden until "Show build labels" is on. `clean`: no badges on the screen. The component inventory is written in every mode.
3. **Distinguish OOTB vs. custom when labels are on** — coloured badges per component type.
4. **Match the story's ACs and Pattern E field spec** (once stories exist) — every happy-path AC reachable in the click-through; forms show every field in the Pattern E table.
5. **Be a single self-contained `.html` file** — inline CSS, no external fonts/JS/CDN dependencies.
6. **Use SLDS 2-flavoured styling** so the mockup reads as *Salesforce*, not a generic web app.
7. **Include a Build-Technology legend when labels are on** — component inventory with effort sizes mirroring the Solution Plan / story. Omitted on a clean screen.
8. **Never invent component or field names** — custom components verified against the codebase, standard Health Cloud objects/fields against the official documentation; unverifiable names marked `(proposed)`. *(No MCP server required — see [Grounding](#grounding-there-is-nothing-you-need-to-install).)*

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
- **Grounded components** — custom verified against the codebase, standard
  Health Cloud against the official documentation; proposals flagged when the
  Health Cloud package isn't deployed in the target org. No MCP server required.

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

Invoke the skill, then describe the job. Do not paste a preamble — persona, grounding, and fictional data are already in the skill.

```
/healthcloud-delivery-architect

Plan and prototype <the job, in one sentence> — no stories yet.
```

Build labels are a question the skill asks. Add a line only if you already know: `Build labels: clean`, `explicit`, or `both`.

When the walkthrough is right: `Promote this plan to stories.`

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

## Grounding: there is nothing you need to install

**The skill has no MCP prerequisites.** Install the `.cursor/` folder, reload
the window, and it works.

This section used to tell you to configure `code-review-graph` and
`salesforce-docs` in your `.cursor/mcp.json`. That was wrong, and it sent people
looking for packages that do not exist. **Both are internal to the author's
environment and are not publicly distributed** — there is no npm package, no
repository, and nothing to request. If an assistant tells you to ask the author
for the server definitions, it has misread the skill.

Nor are they load-bearing. Neither has been available while this skill was
built: `code-review-graph` has never been registered in the authoring
workspace, and `salesforce-docs` has been in an error state since 2026-08-06.

What RULE 3 actually requires is a **capability**, not a server:

| Capability | Default — no setup | Optional upgrade |
|---|---|---|
| Verify a **custom** component before naming it | Grep / Glob / Read over your repo. This is the normal path, not a degraded one. | `code-review-graph`, if your organisation happens to run one, adds caller / dependent / test context. |
| Verify a **standard** Health Cloud object, field, or feature | The official [Health Cloud Object Reference](https://developer.salesforce.com/docs/atlas.en-us.health_cloud_object_reference.meta/health_cloud_object_reference/sforce_api_objects.htm) and [help.salesforce.com](https://help.salesforce.com), cited by URL. | The **Salesforce DX MCP Server** — see below. |

Either way the rule that matters is unchanged: anything unverifiable is marked
`(proposed)`, and **a tool being unavailable is never evidence that a platform
feature does not exist.** An out-of-the-box-versus-custom call that cannot be
verified is a blocking question, not a licence to build custom.

### The one worth configuring: Salesforce DX MCP

If you want stronger grounding than documentation, use the **official, public**
[Salesforce DX MCP Server](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_mcp_server.htm):

```json
{
  "mcpServers": {
    "salesforce-dx": {
      "command": "npx",
      "args": ["-y", "@salesforce/mcp@latest",
               "--orgs", "DEFAULT_TARGET_ORG",
               "--toolsets", "orgs,metadata,data,users,testing"]
    }
  }
}
```

Pointed at an org where Health Cloud is deployed, `run_soql_query` and
`retrieve_metadata` answer "does this object or field exist **here**" better
than any document can. Still optional.

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
index.html                  # GitHub Pages walkthrough (Challenge → Proof)
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

Current: **v1.1.2** — Build labels on a prototype are a question (`clean` / `explicit` / `both`), not something the prompt has to specify. v1.1.1 stopped treating a missing verification tool, or absence from the object catalog, as proof a Health Cloud feature does not exist.
v1.1.0 adds `RULE 17` (org-brand skinning) and the first
supported brand overlay, [Insulet / OmniPod](.cursor/skills/healthcloud-delivery-architect/references/insulet-omnipod-brand.md).
v1.0.1 (2026-08-10) scrubbed LSC template residue surfaced by the end-to-end
sanity test. v1.0 (2026-08-10) was the initial Health Cloud fork of
`lsc-delivery-architect` v1.8. See the Version History table in
[`SKILL.md`](.cursor/skills/healthcloud-delivery-architect/SKILL.md) for the
full change log.
