# Health Cloud data model — object cheat sheet by sub-domain

> Loaded on demand by `SKILL.md`. Use this reference when writing Health Cloud
> stories to name the **right** standard object, avoid inventing custom
> objects, and reason about parent/child relationships. Field-level facts must
> still be verified via the `salesforce-docs` MCP (RULE 3).

## Contents

- Acronym glossary
- Compliance & regulatory drivers
- Patient / Caregiver / Provider identity model
- Care Management (Provider + Payer) object model
- Patient Services / Contact Center object model
- Utilization Management (Payer) object model
- Provider Network Operations object model
- Member 360 (Payer) object model
- Home Health / RPM object model
- Cross-domain / platform objects

---

## Acronym glossary

| Acronym | Full form |
|---------|-----------|
| **HC** | Health Cloud |
| **PHI** | Protected Health Information |
| **HIPAA** | Health Insurance Portability and Accountability Act |
| **SDOH** | Social Determinants of Health |
| **UM** | Utilization Management |
| **PA** | Prior Authorization |
| **RPM** | Remote Patient Monitoring |
| **EHR / EMR** | Electronic Health / Medical Record |
| **HL7 v2** | Legacy HL7 messaging (ADT, ORU, MDM…) |
| **FHIR R4** | Fast Healthcare Interoperability Resources, release 4 |
| **NPI** | National Provider Identifier |
| **TIN** | Tax Identification Number (provider billing) |
| **NCQA** | National Committee for Quality Assurance (accreditation body) |
| **URAC** | Utilization Review Accreditation Commission |
| **270/271** | Eligibility inquiry / response X12 transactions |
| **278** | Services review / prior-auth request X12 transaction |
| **837/835** | Claim / remittance X12 transactions |
| **HEDIS** | Healthcare Effectiveness Data and Information Set |
| **PCP / SCP** | Primary Care Physician / Specialty Care Provider |
| **OOTB / OOB** | Out Of the Box |

## Compliance & regulatory drivers (non-exhaustive)

- **HIPAA Privacy & Security** — PHI handling, minimum necessary, audit-log, access controls.
- **HITECH** — breach notification, meaningful-use tie-ins.
- **TCPA / CAN-SPAM** — patient outbound communications; consent required.
- **State care-management regulations** — vary by state; typically require member notification and appeal rights.
- **NCQA / URAC accreditation** — care-management program design and documentation standards.
- **21st Century Cures Act — Information Blocking** — patient right to access data via FHIR APIs.
- **CMS Interoperability & Patient Access Rule** — payer FHIR APIs for claims / clinical data.
- **State telehealth regulations** — recording, consent, licensure boundaries.

---

## Patient / Caregiver / Provider identity model

Health Cloud primarily uses **Person Account** or the newer **`Individual`**
object as the "person" record for a Patient. Providers and Facilities have
their own dedicated objects.

| Object (business term) | API name | Purpose | Notes |
|---|---|---|---|
| **Patient (Person Account)** | `Account` (Person Account with the Patient record type) | The patient record | Legacy pattern; still most common in Health Cloud implementations |
| **Individual** | `Individual` | Person-level identity (Patient, Contact, Lead) | Newer unified model; enables Contact-only implementations without Person Accounts |
| **Patient (Contact)** | `Contact` | Person representation | Used when Contact is preferred over Person Account |
| **Caregiver** | `Contact` (with Caregiver relationship to Patient Account) or `ContactContactRelation` | Family member, guardian, POA | Consent implications for what a Caregiver can see/do |
| **Healthcare Provider (individual clinician)** | `HealthcareProvider` | An individual clinician (MD, RN, NP, etc.) | Standard HC object; NPI, taxonomy, licenses |
| **Healthcare Facility** | `HealthcareFacility` | Hospital, clinic, imaging center | Standard HC object; addresses, hours, network status |
| **Healthcare Practitioner Facility** | `HealthcarePractitionerFacility` | Junction: which providers practice at which facilities | For rostering |
| **Care Program** | `CareProgram` | A defined care management program (e.g., "Diabetes Management") | Groups Care Plans |
| **Care Program Enrollee** | `CareProgramEnrollee` | Patient enrollment in a Care Program | Junction Patient↔CareProgram |

---

## Care Management (Provider + Payer) object model

Care management is the core Health Cloud domain: planning, coordinating, and
tracking a patient's care over time across a multidisciplinary team.

| Object (business term) | API name | Purpose | Notes |
|---|---|---|---|
| **Care Plan** | `CarePlan` | A patient's active plan of care | Root object for goals, interventions, tasks |
| **Care Plan Template** | `CarePlanTemplate` | Reusable template (e.g., Diabetes Type 2) | Instantiated per patient |
| **Care Plan Template Goal** | `CarePlanTemplateGoal` | Template goal (blueprint) | Copied into `CarePlanGoal` on instantiation |
| **Care Plan Goal** | `CarePlanGoal` | A specific goal for the patient (e.g., "A1C < 7.0") | Owned by Care Manager |
| **Care Plan Problem** | `CarePlanProblem` | A clinical problem being addressed | e.g., "Uncontrolled diabetes" |
| **Care Plan Activity** | `CarePlanActivity` | An intervention/task in the plan | Assigned to a Care Team member |
| **Care Task** | `CareTask` | A discrete task | e.g., "Schedule podiatry follow-up" |
| **Care Barrier** | `CareBarrier` | An obstacle to progress | e.g., "No transportation to appointments" |
| **Care Metric Target** | `CareMetricTarget` | Measurable target on a Care Plan Goal | Numeric / range |
| **Care Observation** | `CareObservation` | Observed value | e.g., A1C reading, BP |
| **Care Diagnosis** | `CareDiagnosis` | Diagnosis code (ICD-10) tied to a patient | For care & risk stratification |
| **Care Team** | `PatientCareTeam` (patient-owned) or `HealthcareProviderCareTeam` (provider-owned) | The care team for a patient | Members via `PatientCareTeamMember` |
| **Care Team Member** | `PatientCareTeamMember` | A person on a patient's care team | Provider, care coordinator, family |
| **Assessment / Assessment Question** | `AssessmentQuestion`, `AssessmentQuestionVersion`, `AssessmentQuestionResponse` | Structured screening (PHQ-9, PRAPARE, HRSN) | Uses the platform Assessments framework |
| **SDOH (Social Determinant of Health)** | Handled via `CareBarrier` + Assessment questions; often surfaced on the Patient 360 | | Non-clinical factors (food, housing, transport) |

---

## Patient Services / Contact Center object model

Inbound patient contact center; adherence coaching; device help. Highly
relevant for med-device patient services (Insulet OmniPod pattern).

| Object (business term) | API name | Purpose | Notes |
|---|---|---|---|
| **Patient (Person Account)** | `Account` w/ Patient record type | The patient calling in | Same identity model as Care Mgmt |
| **Case** | `Case` on the Patient Account | The inbound contact / inquiry / issue | Standard object; use Health Cloud-specific record types |
| **Case Related Contact** | `CaseContactRole` / `CaseRelatedIssue` | Who else is involved in the case | |
| **Contact Encounter** | `ContactEncounter` | A discrete contact event (call, email, chat) | Includes channel, duration, disposition |
| **Contact Encounter Participant** | `ContactEncounterParticipant` | Who participated in the encounter | Patient, caregiver, coordinator |
| **Voice Call** | `VoiceCall` (Service Cloud Voice) | Telephony call record | For CTI / Amazon Connect / Genesys integration |
| **Task** | `Task` | Follow-up task | Standard object; often linked to Case |
| **Adherence Event** | `AdherenceEvent` *(commonly a custom object depending on vertical solution)* | Missed dose, therapy interruption | Verify existence in target org |
| **Refill Request** | `Case` w/ record type "Refill" or a custom `RefillRequest__c` | Prescription refill support | Verify in target org |
| **Device Support Ticket** | `Case` w/ record type "Device Support" | Device malfunction, replacement | E.g. OmniPod device issue |
| **Consent** | `AuthorizationFormConsent`, `AuthorizationForm`, `IndividualEmailResult` | Communication consent | Consent-driven visibility for outbound comms |

---

## Utilization Management (Payer) object model

Prior authorization, service reviews, appeals, medical-necessity
determinations. See [UM overview](https://help.salesforce.com/s/articleView?id=sf.hc_um_overview.htm).

| Object (business term) | API name | Purpose | Notes |
|---|---|---|---|
| **Care Request** | `CareRequest` | The prior-auth request / referral / service review | Root UM object |
| **Care Request Item** | `CareRequestItem` | A single service line (CPT/HCPCS code) | Multiple per request |
| **Care Request Drug** | `CareRequestDrug` | Drug-specific request line | For pharmacy PA |
| **Care Request Review** | `CareRequestReview` | Reviewer's determination | Approve/Deny/Pend |
| **Care Request Extension** | `CareRequestExtension` | Extension to an existing authorization | Adds units/days |
| **Care Request Item Detail** | `CareRequestItemDetail` | Extended attributes on a request line | Clinical rationale, dates |
| **Coverage Benefit** | `CoverageBenefit` | The patient's plan benefit context | Consumed by UM |
| **Purchaser Plan** | `PurchaserPlan` / `PurchaserPlanAssn` | The insurance plan / employer group | Payer model |
| **Member Plan** | `MemberPlan` | Patient's active membership | Effective dates, coverage |
| **Clinical Guideline** | `ClinicalGuideline` / `ClinicalRule` | Medical policy / criteria | Referenced during review |
| **Care Determinant** | `CareDeterminant` | Factors influencing determination | |
| **Business License Application** (appeal workflow) | Standard `Case` w/ Appeal record type | Appeals track | Not a dedicated HC object |

**Related X12 flow:**
- Inbound `278` (PA request) → creates/updates `CareRequest`
- Outbound `278R` (PA response) → after `CareRequestReview` decision
- `270/271` eligibility check → confirms `MemberPlan.IsActive` before UM

---

## Provider Network Operations object model

*(For deep credentialing lifecycle — application → primary source verification →
committee → recredentialing — use the sibling PNM skill.)*

| Object (business term) | API name | Purpose | Notes |
|---|---|---|---|
| **Healthcare Provider** | `HealthcareProvider` | Individual clinician record | NPI, taxonomy, licenses |
| **Healthcare Facility** | `HealthcareFacility` | Facility/entity record | Hospital, clinic, imaging |
| **Practitioner Facility** | `HealthcarePractitionerFacility` | Junction — where provider practices | For rostering |
| **Provider Specialty** | `HealthcareProviderSpecialty` | Specialty taxonomy | Multiple per provider |
| **Provider Facility Network** | `HealthcareProviderFacilityNetwork` | In-network / out-of-network status | Payer network mgmt |
| **Provider License** | `HealthcareProviderLicense` | State licenses | With effective/expiration |
| **Provider NPI** | Field on `HealthcareProvider` (`NationalProviderId`) | Federal NPI | Unique per clinician |
| **Roster File** | Custom or `FileUpload` / DataMapper Load | Bulk provider ingestion | Typically monthly from delegates |
| **Network Adequacy** | Reports on `HealthcareProviderFacilityNetwork` + `Account` (Member) coverage geography | Access standards | State/CMS reporting |

---

## Member 360 (Payer) object model

Payer-side member experience: enrollments, benefits, claims summary.

| Object (business term) | API name | Purpose | Notes |
|---|---|---|---|
| **Member (Person Account)** | `Account` w/ Member record type | The insured member | |
| **Individual Enrollment** | `HealthcareIndividualEnrollment` | Member's enrollment record | Effective dates, tier |
| **Purchaser Plan** | `PurchaserPlan` | The plan the member is on | Employer/group + plan |
| **Coverage Benefit** | `CoverageBenefit` | Benefit line (medical, dental, Rx) | Deductibles, copays |
| **Coverage Benefit Item** | `CoverageBenefitItem` | Detail on a benefit | e.g., in-network vs OON |
| **Benefit Item** | `BenefitItem` | Configured benefit definition | |
| **Claim** | `Claim` | Medical claim | Summarized in Member 360 |
| **Claim Item** | `ClaimItem` | Service line on a claim | CPT/DRG |
| **Encounter** | `Encounter` | Clinical encounter record | e.g., inpatient stay |
| **Case** (Member Services) | `Case` on the Member Account | Inquiries, benefit questions | |

---

## Home Health / Remote Patient Monitoring (RPM) object model

For device-connected patient monitoring (insulin pumps, CGMs, BP cuffs).

| Object (business term) | API name | Purpose | Notes |
|---|---|---|---|
| **Care Observation** | `CareObservation` | Individual device reading | Value, unit, timestamp |
| **Care Registered Device** | `CareRegisteredDevice` | The physical device linked to a patient | Serial, model, activation |
| **Device Manufacturer** | `Account` w/ Manufacturer record type | Device OEM | |
| **In-Home Visit** | `HealthcareVisit` (yes, LSC shares this object) or `Case` w/ Home Visit record type | Scheduled in-home clinical visit | RPM Nurse coordination |
| **Care Team Member** | `PatientCareTeamMember` | Home-health nurse / clinician | Same as Care Mgmt |
| **Alert / Escalation** | `Case` w/ Alert record type; or `Task` | Out-of-range reading triggers | Fired by Flow on `CareObservation` insert |

---

## Cross-domain / platform objects

Objects used across all Health Cloud sub-domains.

| Object (business term) | API name | Purpose |
|---|---|---|
| **Account** | `Account` (Patient / Provider / Facility / Payer record types) | The universal party record |
| **Contact** | `Contact` (Caregiver, PCP contact person) | Related parties |
| **Individual** | `Individual` | Unified person identity |
| **Address / Location** | `Address` | Standard address object |
| **Consent** | `AuthorizationFormConsent`, `IndividualEmailResult`, `EngagementChannelType` | Consent model for outbound comms |
| **Assessment** | `AssessmentQuestion`, `AssessmentQuestionResponse`, `AssessmentTaskDefinition` | Structured screening |
| **Task** | `Task` | Follow-up tasks (universal) |
| **Case** | `Case` | Cross-domain issue tracking |
| **Event** | `Event` | Appointment scheduling |
| **User** | `User` | Care team member as a Salesforce user |
| **Territory / UserTerritory** | `Territory2`, `UserTerritory2Association` | Assignment routing |
| **Setup Audit Trail** | `SetupAuditTrail` | Config-change audit (PHI safety) |
| **Login History** | `LoginHistory` | Who accessed the org, from where |
| **Data Cloud (Customer Data Platform)** | Data Cloud DMOs (Individual, Engagement, Vital, etc.) | Enterprise identity + unified profile |

---

## Cross-references

- **Objects catalog:** `references/healthcloud-standard-objects-catalog.md`
- **Build-tech / component conventions:** `references/healthcloud-components.md`
- **AC patterns + Health Cloud persona cheatsheet:** `references/ac-pattern-library.md`
- **PHI compliance:** RULE 16 in `SKILL.md`
