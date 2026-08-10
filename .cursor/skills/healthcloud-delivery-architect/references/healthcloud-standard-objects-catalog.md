# Health Cloud standard-object catalog

> Curated top-list of Health Cloud standard objects, grouped by functional
> domain. Sourced from the [Health Cloud Object Reference](https://developer.salesforce.com/docs/atlas.en-us.health_cloud_object_reference.meta/health_cloud_object_reference/sforce_api_objects.htm)
> (Summer '26, API v67.0). Verify field-level detail with the `salesforce-docs`
> MCP before use (RULE 3).

## How to use

- **Business-term first** — look up the object by what the business calls it (Care Plan, Prior Auth, Referral, Roster).
- **API name second** — that's what goes into the Technical Implementation section, never into a Pattern-A Given/When/Then line (RULE 13).
- If you can't find an object here, it may be a **custom** object in the target org — check `code-review-graph` before assuming it exists.
- **Since** column reflects the API version the object was **introduced or last significantly updated** in.

---

## Patient & Person identity

| Object | Purpose | Since |
|---|---|---|
| **Account** (Patient / Member / Provider / Facility / Payer record types) | Universal party record; foundation for all HC identity | Standard |
| **Individual** | Person-level identity (Patient, Contact, Lead) — unified model | 40.0 |
| **Contact** | Related person: Caregiver, next-of-kin, PCP contact | Standard |
| **ContactContactRelation** | Person-to-person relationships (Caregiver, POA, family) | 45.0 |
| **AccountContactRelation** | Person-to-account relationships | Standard |
| **Household** | Family/household grouping | 51.0 |

## Care Management

| Object | Purpose | Since |
|---|---|---|
| **CareProgram** | Named care management program (e.g. "Diabetes Care") | 44.0 |
| **CareProgramEnrollee** | Patient enrollment in a Care Program | 45.0 |
| **CareProgramProduct** | Products/services covered under a Care Program | 46.0 |
| **CareProgramGoal** | Goal at the program level | 45.0 |
| **CarePlan** | A patient's active care plan (goals + activities) | 41.0 |
| **CarePlanTemplate** | Reusable template | 45.0 |
| **CarePlanTemplateGoal** | Template goal (blueprint) | 45.0 |
| **CarePlanGoal** | A specific goal for the patient | 45.0 |
| **CarePlanActivity** | Intervention/activity in a plan | 46.0 |
| **CarePlanProblem** | Clinical problem being addressed | 46.0 |
| **CareTask** | Discrete task on a Care Plan | 46.0 |
| **CareBarrier** | Obstacle to progress (transport, housing) | 45.0 |
| **CareBarrierType** | Configured barrier categories | 46.0 |
| **CareMetricTarget** | Measurable target on a goal | 46.0 |
| **CareObservation** | Observed value (A1C, BP, weight) | 46.0 |
| **CareDiagnosis** | Diagnosis (ICD-10) tied to a patient | 45.0 |
| **CareInterventionType** | Configured intervention catalog | 46.0 |
| **PatientCareTeam** | The care team for a patient | 41.0 |
| **PatientCareTeamMember** | Member of a care team | 41.0 |
| **HealthcareProviderCareTeam** | Provider-owned care team | 41.0 |

## Assessments & Screening

| Object | Purpose | Since |
|---|---|---|
| **AssessmentTaskDefinition** | Master assessment definition (PHQ-9, PRAPARE, HRSN) | 47.0 |
| **AssessmentQuestion** | Question within an assessment | 47.0 |
| **AssessmentQuestionVersion** | Versioned question text | 47.0 |
| **AssessmentQuestionResponse** | Patient's answer | 47.0 |
| **AssessmentTaskContentDocument** | Attached content | 47.0 |
| **AssessmentIndicatorDefinition** | Score threshold definitions | 47.0 |

## Utilization Management (Prior Auth, Referrals, Appeals)

| Object | Purpose | Since |
|---|---|---|
| **CareRequest** | Prior-auth request / referral / service review | 46.0 |
| **CareRequestItem** | Service line (CPT/HCPCS) on a request | 46.0 |
| **CareRequestItemDetail** | Extended attributes on a service line | 47.0 |
| **CareRequestDrug** | Drug line on a pharmacy PA | 47.0 |
| **CareRequestReview** | Reviewer determination (Approve/Deny/Pend) | 46.0 |
| **CareRequestExtension** | Extension to an existing authorization | 46.0 |
| **ClinicalGuideline** | Medical policy / criteria | 47.0 |
| **ClinicalRule** | Individual rule within a guideline | 47.0 |
| **CareDeterminant** | Factor influencing determination | 47.0 |

## Payer / Member 360

| Object | Purpose | Since |
|---|---|---|
| **HealthcareIndividualEnrollment** | Member's enrollment record | 46.0 |
| **PurchaserPlan** | The health plan (employer/group + plan design) | 46.0 |
| **PurchaserPlanAssn** | Association of plans to purchasers | 46.0 |
| **MemberPlan** | Patient's active membership | 46.0 |
| **CoverageBenefit** | Benefit line (medical/dental/Rx) | 46.0 |
| **CoverageBenefitItem** | Detail on a benefit item | 46.0 |
| **BenefitItem** | Configured benefit definition | 46.0 |
| **Claim** | Medical claim | 46.0 |
| **ClaimItem** | Service line on a claim | 46.0 |
| **Encounter** | Clinical encounter record | 46.0 |
| **EncounterDiagnosis** | Diagnosis on an encounter | 47.0 |
| **EncounterFacility** | Facility on an encounter | 47.0 |

## Provider Network (roster, network, licenses)

| Object | Purpose | Since |
|---|---|---|
| **HealthcareProvider** | Individual clinician (NPI, taxonomy) | 44.0 |
| **HealthcareProviderSpecialty** | Specialty taxonomy | 45.0 |
| **HealthcareProviderLicense** | State license | 45.0 |
| **HealthcareProviderNpi** | Federal NPI record (also on `HealthcareProvider`) | 45.0 |
| **HealthcareFacility** | Hospital / clinic / imaging center | 44.0 |
| **HealthcareFacilityNetwork** | Facility's network affiliation | 45.0 |
| **HealthcareProviderFacilityNetwork** | Provider's in-network status at a facility | 45.0 |
| **HealthcarePractitionerFacility** | Junction — where provider practices | 45.0 |

## Devices & RPM

| Object | Purpose | Since |
|---|---|---|
| **CareRegisteredDevice** | Device linked to a patient | 47.0 |
| **CareObservation** | Reading captured from the device | 46.0 |
| **HealthcareVisit** | Scheduled visit (can be in-home for RPM/home health) | 51.0 |

## Consent & Communication

| Object | Purpose | Since |
|---|---|---|
| **AuthorizationForm** | The consent form definition | 48.0 |
| **AuthorizationFormConsent** | The patient's consent record for a specific form | 48.0 |
| **EngagementChannelType** | The channel (email, SMS, phone, mail) | 48.0 |
| **AuthorizationFormDataUse** | What the consent covers | 48.0 |
| **IndividualEmailResult** | Outbound email delivery result | 51.0 |
| **PartyConsent** | Legacy consent object (still supported) | 45.0 |
| **DataUsePurpose** | Purpose category for consent | 48.0 |
| **DataUseLegalBasis** | Legal basis (GDPR-flavoured) | 48.0 |

## Case / Contact Center

| Object | Purpose | Since |
|---|---|---|
| **Case** (Patient Services / Member Services / Appeals record types) | Inbound issue tracking | Standard |
| **ContactEncounter** | Contact event (call, chat, email) | 51.0 |
| **ContactEncounterParticipant** | Who participated in the encounter | 51.0 |
| **VoiceCall** | Service Cloud Voice call record | 48.0 |
| **CaseContactRole** | Who is involved in a case | Standard |

## Care Barriers & SDOH

| Object | Purpose | Since |
|---|---|---|
| **CareBarrier** | Obstacle to progress | 45.0 |
| **CareBarrierType** | Configured barrier categories | 46.0 |
| **CareBarrierAction** | Action taken to address a barrier | 46.0 |

## Clinical / EHR interop

| Object | Purpose | Since |
|---|---|---|
| **HealthCloudGA__EhrPatient__c** *(managed pkg legacy)* | EHR patient record | Legacy |
| **CareObservation** | FHIR Observation equivalent | 46.0 |
| **CareDiagnosis** | FHIR Condition equivalent | 45.0 |
| **CareTask** | FHIR ServiceRequest equivalent | 46.0 |
| **Encounter** | FHIR Encounter equivalent | 46.0 |
| **CareRegisteredDevice** | FHIR Device equivalent | 47.0 |

## Platform / Cross-domain

| Object | Purpose | Since |
|---|---|---|
| **User** | Salesforce user (care team member as login) | Standard |
| **Territory2 / UserTerritory2Association** | Assignment routing | Standard |
| **Address** | Standard address | 51.0 |
| **Task** | Follow-up task | Standard |
| **Event** | Appointment | Standard |
| **SetupAuditTrail** | Config-change audit | Standard |
| **LoginHistory** | Login audit trail | Standard |
| **RecordType** | Record-type routing (e.g. Patient vs Member on Account) | Standard |

---

## Version discovery

To confirm an object exists in the target org and see its fields, use:

```bash
sf sobject describe --sobject CarePlan --target-org <alias> | jq '.fields[].name'
```

Or via `salesforce-docs` MCP:

```
salesforce_docs_search  query="CarePlan object"
```

---

## Notes

- Health Cloud has evolved through multiple runtimes: the **managed-package** era (namespace `HealthCloudGA__`), the **Industries Common Layer** era, and the **native standard objects** era (post-Summer '20). New work should target the native standard objects wherever possible.
- Some organisations still run legacy managed-package objects (`HealthCloudGA__EhrPatient__c` etc.). When migrating a legacy Health Cloud instance, translate to native equivalents in the story body and keep the managed-pkg name only as a mapping note.
- For fields not obvious from the object name (e.g., `CareRequest.SubmissionDate` vs `CareRequest.EffectiveStartDate`), always verify via `salesforce-docs` before adding to Pattern E.
