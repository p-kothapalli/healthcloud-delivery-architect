# Persona & AC Authoring Contract + AC Pattern Library (Health Cloud)

The two non-negotiables for every generated story: a concrete Health Cloud
business-role persona, and acceptance criteria written in business language
using one of five patterns. This file is the authoritative detail for both;
`SKILL.md` summarizes and links here.

## Contents

- Persona rule (concrete HC business role) + Health Cloud Persona Cheatsheet + the Patient/Provider trap
- Given / When / Then contract (Pattern A rules + right/wrong examples)
- AC Pattern Library: Pattern A (behavioural), B (field/object/metadata),
  C (permission set / FLS), D (field update rules), E (record & field specification)
- "When in doubt" pattern-selection guide

---

## A) Persona must be a real, concrete Health Cloud business role

The persona is the **Salesforce login user** who performs the action — a
care-team, contact-center, utilization, or network-ops role — NOT the patient,
caregiver, or provider the work is *about*.

| Don't write | Write instead |
|---|---|
| "As a business user" | "As a **Care Coordinator**" |
| "As a user" | "As a **Utilization Reviewer**" / "**Patient Services Representative**" / etc. |
| "As a Patient" (for an internal feature) | "As a **Patient Services Representative** supporting a Patient" (the Patient is the subject) |
| "As a Provider" (for an internal feature) | "As a **Network Manager** managing a HealthcareProvider" |
| "As a developer / business analyst" | Keep the business persona on the header; put developer-facing rules into Technical Implementation. |

### The Patient / Caregiver / Provider trap (Health-Cloud-specific)

- **Patient**, **Caregiver**, **HealthcareProvider** (individual clinician), and **HealthcareFacility** are the **subjects/targets** of the work — the person or entity a care team member serves, contacts, or manages.
- They are the persona **only** when the story is genuinely for a Patient-or-Provider-facing experience (an **Experience Cloud** patient/member portal, a provider self-service portal, a Caregiver mobile app). In that case name it: "As a **Patient portal user**" or "As a **HealthcareProvider portal user**".
- For everything internal, the persona is the care/contact-center/UM/network role.

### Health Cloud Persona Cheatsheet

Detected on first run by scanning `requirements/` and `permissionsets/` for role
labels. Canonical starter list (Health Cloud):

| Sub-domain / Context | Persona to use |
|---|---|
| Care planning, care coordination, longitudinal care | **Care Coordinator** |
| Complex-case management, high-risk patients, care programs | **Care Manager** (or **Nurse Case Manager** for clinical) |
| Behavioural health / social work | **Behavioural Health Specialist** or **Social Worker** |
| Inbound patient/member calls, adherence coaching, device help | **Patient Services Representative** (or **Adherence Coach**) |
| Contact-center team lead / supervisor | **Patient Services Team Lead** |
| Prior authorization / service review / medical necessity | **Utilization Reviewer** (or **UM Nurse**) |
| Physician-level UM decision, denials, peer-to-peer | **Medical Director** |
| Appeals & grievances | **Appeals Analyst** |
| Provider roster ingestion, network adequacy, directory data | **Network Manager** (or **Provider Data Steward**) |
| Payer member enrollment / benefits inquiry | **Member Services Representative** |
| Home-health scheduling, in-home visits | **Home Health Coordinator** |
| Remote patient monitoring (device readings, alerts) | **RPM Nurse** |
| Telehealth / virtual care scheduling | **Telehealth Coordinator** |
| Care-team collaboration (multi-disciplinary rounds) | **Care Team Lead** |
| Privacy / HIPAA compliance oversight | **Privacy Officer** (or **Compliance Analyst**) |
| Referral triage & routing | **Referral Coordinator** |
| Health Cloud administration / configuration | **Health Cloud Administrator** |

If the user names a persona that isn't in the cheatsheet, accept it verbatim and
add it to the story header. Prefer the workspace's established term over a casual
term when they conflict.

---

## B) Acceptance Criteria — Given / When / Then is mandatory

Every behavioural AC is **three explicit lines**, not a prose paragraph:

```
**AC-N — Short title**

**Given** [precondition / state before the action — be specific about objects, fields, picklist values, RecordTypes, patient state]
**When** [the single trigger / action — usually one user action]
**Then** [the observable, verifiable outcome — record changes, field values, error messages, UI behavior]
**And** [optional additional outcomes, one per line]
```

Hard rules:

- One **When** per AC. If you have two actions, split into two ACs.
- The **Then** must be observable and checkable (QA can query, click, or read a
  log to confirm it). "It works" is not a valid Then.
- Use `**And**` lines for compound outcomes, not commas inside Then.
- Keep verbs in business language; implementation details belong in the
  Technical Implementation section.
- Edge cases get their own AC (Given = the edge state; Then = the
  rejection / fallback / log entry).
- If you write more than ~6 lines under a single Then, split the AC.
- **PHI ACs are mandatory (RULE 16).** If a story displays, logs, exports, or sends PHI, include an audit AC: "**Then** an audit log entry is written with the user, timestamp, and record accessed."

**Example — wrong (prose with technical jargon):**

> AC-1.1 When Submit fires, `HC_PriorAuth_Procedure_4` calls
> `CareRequestReviewService.determine(...)` before inserting the
> `CareRequestReview` record.

**Example — right (GWT in pure business language):**

```
**AC-1.1 — Prior authorization is routed to a reviewer**

**Given** a Utilization Reviewer has an incoming prior-authorization request for a service
that requires medical-necessity review,
**When** they submit the request for triage,
**Then** the request is routed to the Utilization Reviewer covering the requested service
line and the patient's health plan,
**And** an SLA clock starts at 72 hours for standard reviews (24 hours for urgent),
**And** the patient's active coverage is verified before routing (denied requests never enter the queue).
```

The implementation details (service class name, IP step numbers, SOQL) move to
the story's Technical Implementation (high-level) section.

---

## AC Pattern Library

Every AC uses ONE of five patterns. Pick the pattern that matches what the AC is
asserting; don't force a GWT shape on a metadata-spec AC.

### Pattern A — Behavioural AC (Given / When / Then) — DEFAULT

Use for: anything that asserts **system behaviour** the persona can observe — a
record gets created, a field gets updated, an error is shown, a case is routed,
a determination is made.

```
**AC-N — Short business-language title**

**Given** [persona + business state, no API names, no class names]
**When** [single business action]
**Then** [observable business outcome — what the persona sees, what business records change in business terms]
**And** [optional additional outcomes, one per line]
```

Keep the verbs in business language. Say "the care plan is closed", not
"`CarePlan.Status__c` is set to `Closed`". Say "the review is set to
Approved", not "`CareRequestReview.Determination` is `Approved`".

### Pattern B — Field / Object / Metadata Creation

Use for: new custom fields, new custom objects, new Custom Metadata Types, new
field history tracking, new picklist values. These are inherently technical-spec
ACs — bullet structure is required, GWT is not.

```
**AC-N — Create following fields on [Object Label]**

- **API Name:** HC_XxxYyy__c
- **Object:** [Salesforce object label / API name — e.g. Case, CarePlan]
- **Type:** Checkbox / Text(80) / Lookup(Target) / Picklist / Master-Detail / etc.
- **Label:** [User-facing label]
- **Default:** [optional]
- **Help text:** [optional, customer-facing]
- **Description:** [optional, admin-facing]
- **Track History:** true / false
- **Required:** true / false (where applicable)
- **PHI classification:** Standard / Sensitive / High-Sensitivity (drives FLS + Shield encryption decision)
```

For new objects, add: Plural label, Sharing model, Auto-Number / Name format,
and a sub-list of fields using the same bullet style. **Prefer extending a
standard Health Cloud object over creating a custom object.**

### Pattern C — Permission Set / FLS

Use for: permission set updates, profile FLS, OWD changes, sharing rules. Health
Cloud stories touching PHI **must** include this pattern.

```
**AC-N — Field Access & Permission Sets**

- **HC_CareCoordinator:**
  - Object level: Read, Create, Edit on Case, CarePlan, CareTask
  - Field level: Read and Edit on all available non-sensitive fields
- **HC_UM_Reviewer:**
  - Object level: Read, Create, Edit, View All Records on CareRequest, CareRequestReview
  - Field level: Read and Edit on all UM fields
- **HC_PatientServicesRep:**
  - Object level: Read, Create, Edit on Case with Patient Services record type
  - Field level: Read on Patient demographic fields; no access to SSN, financial info
- **HC_PrivacyOfficer:**
  - Object level: Read on Setup Audit Trail, Login History
  - Field level: Read on all audit fields
```

If only one permission set is affected and only one field is changing, you may
collapse into a single bullet, but keep the Object-level / Field-level split.

### Pattern D — Field Update Rules (business calculation tables)

Use for: ACs that describe **how** a value is computed when many conditional
branches feed it. Often appears inside the Then of a Pattern A AC, or as a
standalone "update rules" supporting block.

```
**AC-N — Update following records on [Object]**

**Given** [precondition],
**When** [action],
**Then** the following fields are set per the rules below:

- **CareRequest Determination =**
  - When ClinicalRule matches AND patient coverage is active -> Approved
  - When ClinicalRule fails AND no clinical override -> Denied
  - When additional information is required -> Pended (send letter to provider)
  - Else -> Route to Medical Director
- **SLA Due Date =** SubmissionDate + 72 business hours (urgent = SubmissionDate + 24 hours)
- **Case Priority =**
  - When patient is in an active care program AND age >= 65 -> High
  - When patient has 3+ open cases -> High
  - Else -> Normal
```

This pattern is preferred over cramming six "And" lines into a single Then.

### Pattern E — Record & Field Specification (post-action record recipe) — MANDATORY when records are created/updated

Use for: any story where a Save, Submit, batch run, flow completion, or trigger
**creates or updates records** and the business team needs the *exact recipe* —
every object, every field, and the exact value (or formula) each is set to. One
block **per object**, listing **every field**, so a developer can build the write
without a follow-up meeting and QA can assert every field.

**Why mandatory:** a Pattern-A AC says *"the prior auth is submitted"* in
business language, but it does **not** tell the developer that
`CareRequest.SubmissionDate = TODAY`, `CareRequest.PriorityLevel = "Standard"`,
etc. Whenever an AC's outcome is *"records are created/updated"*, pair it with a
Pattern E block that enumerates the fields.

**Structure — one sub-block per object, in creation/update order (parents before children):**

```
**AC-N — Records created/updated on <trigger> (outcome = <branch>)**

**Given** [precondition / which branch this recipe applies to],
**When** [the single trigger — Save / Submit / batch run / button click],
**Then** the following records are created/updated exactly as specified:

**CareRequest — Create**

| Field | Value | Notes |
|---|---|---|
| Record Type | Prior Authorization | new |
| Status | Submitted | |
| Priority Level | Standard | Urgent if requester flagged |
| Submission Date | {TODAY} | |
| Requesting Provider | {Selected Provider} | HealthcareProvider lookup |
| Patient (Account) | {Patient} | lookup, Person Account |
| Health Plan | {Patient MemberPlan.PurchaserPlan} | roll-up from active MemberPlan |
| SLA Due Date | {TODAY} + 72 business hrs | Pattern D above |
| Care Program Enrollee | {Patient's active enrollee} | lookup if patient is in a program |
| ... every remaining field ... | ... | ... |

**CareRequestItem — Create (1 per service line)**

| Field | Value | Notes |
|---|---|---|
| CareRequest | {parent CareRequest} | master-detail |
| Service Code | {CPT / HCPCS} | picklist |
| Units | {Requested Units} | |
| ... | ... | ... |
```

Hard rules for Pattern E:

- **List EVERY field** the record write touches — no "etc." / "and other fields".
- **One table per object.** Order objects the way they're written (parents
  before children) so the developer can follow the DML order.
- **Value column** holds the literal value, a `{merge/source}` placeholder in
  curly braces (e.g. `{Patient}`, `{TODAY}`, `{Care Plan}`), or a short
  formula. For multi-branch computed fields, nest the Pattern D rules in the
  Value cell or link to the Pattern D block.
- Use **business field labels**, not API names — the API-name mapping lives in
  the Technical Implementation table.
- If the same trigger produces **different recipes per branch**, give **each
  branch its own Pattern E AC** so QA can test each outcome independently.
- **PHI note:** if any field in the record is Sensitive or High-Sensitivity PHI, mark it in the Notes column ("PHI — Shield-encrypted").
- A "Create a Note / Care Task" instruction is a record write — spec it as its
  own object block.

### When in doubt

- A field, object, metadata type, or perm set is being **created or changed** (its *definition*) → Pattern B or C (structured bullets).
- The persona observes **behaviour** of the system → Pattern A (GWT).
- A computation has **many conditional branches** feeding a single outcome → Pattern D (rules block).
- A Save / Submit / batch / trigger **creates or updates record data** and you need the exact per-field recipe → Pattern E. **Always pair Pattern E with the Pattern A AC that describes the same action in business language.**
- A story touches **PHI** (displays, logs, exports, sends) → add a Pattern A audit AC + Pattern C permission-set/FLS spec (RULE 16 in `SKILL.md`).

See `references/story-examples.md` for worked exemplars (Patterns A–E).
