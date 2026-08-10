# Health Cloud — Worked story exemplars (Patterns A–E)

> Loaded on demand by `SKILL.md`. These are canonical exemplars showing what a
> Health-Cloud-native user story looks like when the persona contract, AC
> patterns A–E, and RULE 16 (PHI awareness) are all applied correctly.

## Contents

- Exemplar 1 — **Care Coordinator: log & route patient calls** (Pattern A + C + E)
- Exemplar 2 — **UM Reviewer: prior authorization triage** (Pattern A + D + E)
- Exemplar 3 — **New custom field on CarePlan** (Pattern B + C)

---

## Exemplar 1 — Care Coordinator: log & route patient calls (Patient Services)

### Header

- **Story ID:** HC-PS-001
- **Persona:** Care Coordinator (Patient Services)
- **Sub-domain:** Patient Services
- **Priority:** P0
- **Build Technology:** OOTB `Case` record page + Dynamic Actions + Path, one Screen Flow (`HC_LogPatientCall`), one Record-Triggered Flow (`Case_AutoAssign_PatientServices`), one FlexCard (`FC_Patient_Inquiries` — proposed). Native Approved Email for outbound.
- **Effort (AI-estimated — validate with team):** L

### Story

**As a** Care Coordinator supporting patients in the OmniPod adherence program,
**I want** a one-tap way to log an inbound patient call and route it to the right care-team member,
**So that** urgent clinical questions reach a nurse within SLA and I don't lose context when the patient calls back.

### Acceptance Criteria

#### AC-1 — Log a patient call from the Patient 360 record (Pattern A)

**Given** a Care Coordinator is on Patient Sarah Chen's Patient 360 record and has an active call in progress,
**When** they click *Log Patient Call* on the record page,
**Then** a guided intake opens capturing call category (Clinical Question, Refill, Device Issue, Adherence Support, Billing) and captures verbatim call notes,
**And** an audit log entry is written with the coordinator's User ID, timestamp, and Patient Account ID.

#### AC-2 — Auto-route by category (Pattern A)

**Given** the Care Coordinator selects category = *Clinical Question* and submits,
**When** the record saves,
**Then** the Case is assigned to the covering Nurse Case Manager on the Patient's active Care Program,
**And** the SLA clock starts at 4 business hours,
**And** if no Nurse Case Manager owns the Patient the Case is routed to the *Patient Services Escalation* queue.

#### AC-3 — Adverse-event / product-complaint fan-out (Pattern A, edge case)

**Given** the Care Coordinator flags the call as an adverse event or product complaint,
**When** the record saves,
**Then** a parallel intake Case is created in the *Pharmacovigilance / Product Complaint* queue,
**And** the original Case is locked from the Care Coordinator's edit profile per compliance,
**And** the SLA clock on the parallel Case is set to 24 hours.

#### AC-4 — Records created/updated on submit (Pattern E, happy path)

**Given** the Care Coordinator submits an intake with category = *Clinical Question* and no AE flag,
**When** the Screen Flow's final step fires,
**Then** the following records are created/updated:

**Case — Create**

| Field | Value | Notes |
|---|---|---|
| Record Type | Patient Services | new |
| Status | Routed | Path advances via Flow |
| Origin | Phone | picklist |
| Priority | Normal | Pattern D: High if patient has 3+ open cases |
| Category | Clinical Question | picklist |
| Subject | {first 80 chars of call notes} | text |
| Description | {full call notes} | textarea |
| Account (Patient) | {Patient} | lookup, Person Account |
| Care Program Enrollee | {Patient's active enrollee} | lookup |
| Owner | {covering Nurse Case Manager or Escalation queue} | Pattern D |
| SLA Due Date | {TODAY + 4 business hrs} | |
| Is Adverse Event | false | |
| Source Contact Encounter | {ContactEncounter Id} | lookup |
| PHI Classification | Sensitive | audit tag |

**ContactEncounter — Create**

| Field | Value | Notes |
|---|---|---|
| Encounter Type | Inbound Phone | picklist |
| Start Time | {call start timestamp} | |
| End Time | {call end timestamp} | |
| Duration | {computed} | |
| Patient | {Patient} | lookup |
| Handled By | {running Care Coordinator} | user lookup |

**Task — Create (only if follow-up flagged)**

| Field | Value | Notes |
|---|---|---|
| Subject | Follow-up on {Case Subject} | |
| Related To | {parent Case} | |
| Owner | {parent Case Owner} | |
| Due Date | {SLA Due Date} | |

#### AC-5 — Field access & permission sets (Pattern C)

- **HC_CareCoordinator:**
  - Object level: Read, Create, Edit on `Case`, `ContactEncounter`, `Task`
  - Field level: Read, Edit on Case fields listed in AC-4; Read on Patient demographics (no SSN, no DOB masking removed)
- **HC_NurseCaseManager:**
  - Object level: Read, Create, Edit, Delete on `Case` with Patient Services record type
  - Field level: Full Read/Edit on clinical fields; Read on billing fields
- **HC_PrivacyOfficer:**
  - Object level: Read on `SetupAuditTrail`, `LoginHistory`; View All on `Case`
  - Field level: Read on all audit fields

#### AC-6 — Audit log for PHI display (Pattern A — mandatory under RULE 16)

**Given** any Care Coordinator opens Patient Sarah Chen's Patient 360 record,
**When** the record renders,
**Then** an audit-log entry is written with the User ID, Patient Account ID, IP address, and timestamp,
**And** the entry is retained for 6 years (HIPAA minimum).

### Technical Implementation (high-level)

| AC | Component | Type | Change | Notes |
|---|---|---|---|---|
| 1 | `HC_LogPatientCall` Screen Flow | Flow | New | Multi-screen intake, launched by Quick Action on `Account` (Patient) |
| 2 | `Case_AutoAssign_PatientServices` | Record-Triggered Flow | New | Reads CareProgramEnrollee → sets Owner; fallback to queue |
| 3 | `Case_AE_Fanout` | Record-Triggered Flow | New | Fires when `IsAdverseEvent` = true; creates parallel Case |
| 4 | Standard `Case` + `ContactEncounter` + `Task` | OOTB | Config (record type, page layout) | No custom object |
| 5 | Permission sets `HC_CareCoordinator`, `HC_NurseCaseManager`, `HC_PrivacyOfficer` | Config | New | See Pattern C |
| 6 | `HC_PatientAccess_Audit` | Apex trigger + `EventLog` | Apex | Field-level audit; alternatively use Field Audit Trail on `Account` |

### Definition of Done

- [ ] All ACs pass in staging
- [ ] PHI audit trail confirmed writing entries
- [ ] Permission sets deployed and validated
- [ ] Screen Flow tested on Salesforce Mobile
- [ ] SLA countdown displayed on Case highlights panel
- [ ] Runbook updated for Patient Services team
- [ ] Prototype reviewed with PO (link to `HC_PatientCallLogging_Prototype.html`)

### Estimated Effort

| Component | Size |
|---|---|
| Screen Flow `HC_LogPatientCall` | M |
| Record-Triggered Flows (2) | M |
| Dynamic Actions + Page Layout | S |
| Permission Sets (3) | S |
| Audit trigger | S |
| FlexCard `FC_Patient_Inquiries` (proposed) | S |
| **Total (AI-estimated)** | **L** |

---

## Exemplar 2 — UM Reviewer: prior authorization triage (Utilization Management)

### Header (abbreviated)

- **Story ID:** HC-UM-004
- **Persona:** Utilization Reviewer
- **Sub-domain:** Utilization Management
- **Priority:** P0
- **Build Technology:** OOTB `CareRequest` record page + Path, Record-Triggered Flow for routing, Apex service for clinical-rule evaluation against `ClinicalGuideline`, FlexCard for reviewer queue tile.
- **Effort (AI-estimated — validate with team):** L

### Selected ACs

#### AC-1 — Route inbound PA to a reviewer (Pattern A)

**Given** a Utilization Reviewer has an incoming prior-authorization request for a service that requires medical-necessity review,
**When** the CareRequest saves,
**Then** the request is routed to the Utilization Reviewer covering the requested service line and the patient's health plan,
**And** the SLA clock starts at 72 hours (24 hours if flagged urgent),
**And** if no matching reviewer is available, the CareRequest goes to the *UM Triage* queue.

#### AC-2 — Determination outcomes (Pattern D)

**Given** a Utilization Reviewer completes their review of a CareRequest,
**When** they submit the review,
**Then** the CareRequest Determination is set per these rules:

- **Approved** — Clinical rule passes AND patient coverage is active
- **Denied — Not Medically Necessary** — Clinical rule fails AND no clinical override applied
- **Pended — Additional Information Requested** — Reviewer marks the request as needing more info; a letter is generated to the requesting provider
- **Escalated to Medical Director** — Reviewer requests physician-level review
- **Cancelled** — Requesting provider retracts before determination

**And** the SLA clock is paused when Pended, resumed on info receipt.

#### AC-3 — Records created on Approved determination (Pattern E, single branch)

**Given** the Utilization Reviewer submits an Approved determination,
**When** the CareRequestReview saves,
**Then** the following records are created/updated:

**CareRequestReview — Create**

| Field | Value | Notes |
|---|---|---|
| CareRequest | {parent} | master-detail |
| Determination | Approved | |
| Determination Date | {NOW} | |
| Reviewer | {running user} | |
| Clinical Rationale | {reviewer notes} | textarea |
| Guideline Referenced | {selected ClinicalGuideline} | lookup |

**CareRequest — Update**

| Field | Value | Notes |
|---|---|---|
| Status | Approved | Path advances |
| Effective Start Date | {TODAY} | |
| Effective End Date | {TODAY + Authorization Duration} | Pattern D on Duration |
| Authorization Number | {auto-generated} | |
| Determination | Approved | denormalized for reporting |

**CareRequestItem — Update (all lines)**

| Field | Value | Notes |
|---|---|---|
| Status | Approved | |
| Approved Units | {Requested Units unless capped by clinical rule} | |

**Task — Create (for provider notification)**

| Field | Value | Notes |
|---|---|---|
| Subject | Notify requesting provider — Approved | |
| Owner | {UM Notification queue} | |
| Due Date | {TODAY} | |
| Related To | {parent CareRequest} | |

#### AC-4 — Audit log for PA determinations (RULE 16)

**Given** any determination is submitted,
**When** the CareRequestReview saves,
**Then** an audit log entry captures User ID, CareRequest ID, Determination value, and timestamp,
**And** the entry is retained for 10 years (state UM regulations).

---

## Exemplar 3 — New custom field on CarePlan (Pattern B + C only)

### Header

- **Story ID:** HC-CM-012
- **Persona:** Care Manager
- **Sub-domain:** Care Management
- **Priority:** P1
- **Effort:** S

### ACs

#### AC-1 — Create field `HC_HighRiskFlag__c` on CarePlan (Pattern B)

- **API Name:** HC_HighRiskFlag__c
- **Object:** CarePlan
- **Type:** Checkbox
- **Label:** High-Risk Patient
- **Default:** false
- **Help text:** Set when the patient meets high-risk criteria (age ≥ 75 + 3+ chronic conditions).
- **Description:** Flag used by care escalation reports and dashboards. Do NOT edit manually — set by `CarePlan_RiskEvaluation` Flow.
- **Track History:** true
- **Required:** false
- **PHI classification:** Standard (clinical flag, no direct patient identifiers)

#### AC-2 — Field access & permission sets (Pattern C)

- **HC_CareCoordinator:** Read
- **HC_CareManager:** Read, Edit
- **HC_NurseCaseManager:** Read, Edit
- **HC_MedicalDirector:** Read
- **HC_ReportingReadOnly:** Read (for HEDIS + accreditation reports)

---

## Notes on style

- **Persona** is always a real Health Cloud login role, never "user" or "system" (RULE 12).
- **Patient / Caregiver / Provider** are subjects, not personas, in every ACs (unless it's an Experience Cloud portal story).
- **Pattern E is mandatory** on every Save/Submit/batch AC (RULE 15). Every field enumerated, no "etc."
- **RULE 16 audit AC** appears on every story that displays, logs, exports, or sends PHI.
- **Business language only in Pattern A GWT** — no Apex names, IP steps, SOQL, or `*__c` API names.
- **Build technology** stated in the header (RULE 7a) — declarative-first, OmniStudio only when justified.
