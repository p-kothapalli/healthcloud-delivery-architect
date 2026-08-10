# Health Cloud — build-technology decision guide + component conventions

> Loaded on demand by `SKILL.md` (RULE 7a). Health Cloud is a **standard
> Salesforce platform product**; OmniStudio is available and useful but
> **additive**. This guide picks the right tech for each requirement.

## Contents

- Declarative-first decision guide (RULE 7a)
- Build-technology decision matrix (by requirement shape)
- Component types on the standard platform
- Component types in OmniStudio
- OmniStudio runtime — Standard vs managed package
- Health Cloud naming conventions
- PHI / HIPAA considerations for each component type

---

## Declarative-first decision guide (RULE 7a)

**Order of preference — always try higher up the list first:**

1. **OOTB Lightning** — a standard Health Cloud object (`CarePlan`, `CareRequest`, `Case`, `HealthcareProvider`), a standard Lightning record page, a standard related list, a standard Path. **Cost: none.**
2. **Config** — page layout, compact layout, field creation, Dynamic Actions with visibility rules, custom permission sets, custom list views, custom report types. **Cost: hours to days.**
3. **Screen Flow** — a guided multi-screen interaction (patient intake, prior-auth submission wizard, referral triage). Runs on Lightning Experience and Salesforce Mobile (limited iPad offline). **Cost: days.**
4. **Record-Triggered Flow / Subflow** — automation on create/update (auto-assign a care coordinator by territory, spawn a `Case` when a `CareObservation` is out of range, close an authorization when the coverage terminates). **Cost: days.**
5. **Action Launcher / Quick Action** — one-tap entry point for a Screen Flow or LWC. **Cost: hours.**
6. **LWC** — custom UI where a Lightning record page + Dynamic Actions cannot achieve the pattern (multi-column care-team collaboration board, live vitals dashboard, drag-drop referral triage). **Cost: days to weeks.**
7. **Apex** — custom logic that can't be expressed declaratively (complex clinical rules, high-volume triggers, transaction-boundary control, callouts to non-FHIR/non-MuleSoft endpoints). **Cost: days to weeks.**
8. **OmniStudio (OmniScript, FlexCard, IP, Data Mapper)** — a **specific-purpose** tool for guided, branching, sometimes-offline user flows (patient intake, complex UM workflows, member enrollment) or reusable server orchestration (Integration Procedure). **Not the default.** **Cost: weeks.**

**Rule of thumb:** if two candidate technologies both meet the requirement,
pick the higher one on the list. Justify the OmniScript decision in Technical
Implementation — "chosen because [branching + offline / reusable across N
callers / …]; rejected [Screen Flow] because [reason]."

---

## Build-technology decision matrix (by requirement shape)

| Requirement shape | Recommended technology | Why |
|-------------------|------------------------|-----|
| **Configure a standard HC object page** (`CarePlan`, `CareRequest`, `Case`) | OOTB + Config | Standard record page, Path, compact layout, related lists — no code |
| **Add a Dynamic Action to a record page** with stage-gated visibility | Config (Dynamic Actions) | Declarative, evaluated at runtime |
| **Auto-assign a `CareRequest` to a UM reviewer** by product / provider / plan | Record-Triggered Flow | Reads related objects, sets `OwnerId`, no code |
| **Bulk auto-close** expired prior authorizations | Scheduled Flow or Batch Apex | Flow first; Batch Apex if volume > flow-safe limits |
| **Multi-screen patient intake wizard** (guided, some fields conditional) | Screen Flow | First choice — runs on Lightning Experience + Salesforce Mobile |
| **In-field home-visit capture with iPad offline** | OmniScript (Standard runtime) | Offline persistence + branching; Screen Flow fallback if offline is optional |
| **Care team collaboration board** with drag/drop columns per stage | LWC | Beyond page-layout capabilities |
| **Real-time vitals dashboard** with WebSocket / streaming updates | LWC + Platform Events | Reactive UI, event-driven |
| **Reusable server orchestration** across multiple consumers | Integration Procedure + Data Mapper (Load/Extract) | Server-side, cacheable, versionable |
| **Compact 360° tile** on a Patient page (open Cases, open Care Plans, next appt, latest A1C) | FlexCard | Read-many, config-only display component |
| **Complex clinical rule** (e.g., prior-auth medical-necessity determination) | Apex + `ClinicalGuideline`/`ClinicalRule` | Standard HC objects for rules; Apex for evaluation logic |
| **FHIR R4 API integration** to an EHR (Epic, Cerner) | Native FHIR R4 endpoints + MuleSoft (or DX) | Health Cloud ships FHIR R4 endpoints |
| **HL7 v2 legacy interface** (ADT, ORU, MDM) | MuleSoft with HL7 v2 connector | Not native; MuleSoft is the Salesforce-standard middleware |
| **Eligibility 270/271** check | MuleSoft + custom Apex service or Data Cloud | Payer-side integration |
| **Consent-gated outbound email** to a patient | OOTB `AuthorizationFormConsent` + Approved Email + Flow | Native consent model |
| **Provider roster monthly ingestion** | MuleSoft + Bulk API + Data Mapper (Load) | Batch data with error handling |
| **Referral routing** across the enterprise | Screen Flow (intake) + Record-Triggered Flow (assign) on `CareRequest` | Declarative end-to-end |
| **Adherence prediction / risk stratification** | Data Cloud + Einstein Discovery model + FlexCard | Beyond declarative logic |

---

## Component types — Standard Salesforce platform

| Type | Naming pattern | Where it lives | Notes |
|------|---------------|----------------|-------|
| **Lightning Record Page** | Standard object page | Object Manager → Lightning Record Pages | Use Dynamic Forms + Dynamic Actions for HC records |
| **Lightning App** | `Health_Cloud_Care_Mgmt`, `Patient_Services`, `Utilization_Mgmt` | Setup → Lightning Apps | One per major sub-domain |
| **Page Layout / Compact Layout** | `Patient_360_Care_Mgmt`, `CareRequest_UM_Reviewer` | Object Manager | Prefer Dynamic Forms over page layouts for new work |
| **Field Set** | `PatientOverview_HighlightsPanel` | Object Manager | Used by LWCs and Flows |
| **Screen Flow** | `HC_PatientIntake_v1`, `HC_ReferralTriage_v1` | Flow Builder | Business-facing wizard |
| **Record-Triggered Flow** | `CareRequest_AutoAssign`, `CareObservation_Alert` | Flow Builder | Automation on create/update |
| **Subflow** | `HC_ConsentCheck_Subflow` | Flow Builder | Reusable step |
| **Path** | Stage progression on `CareRequest`, `CarePlan`, `Case` | Setup → Path Settings | Declarative stage guidance |
| **Dynamic Action** | Action + visibility rule on a record page | Lightning App Builder → Actions | Show/hide by stage, record type, field |
| **Quick Action** | `Log_Patient_Call`, `Submit_Prior_Auth` | Object Manager | Entry point for a Flow or LWC |
| **Action Launcher** | Launcher component on record page | Lightning App Builder | Groups multiple actions |
| **Permission Set** | `HC_CareCoordinator`, `HC_UM_Reviewer`, `HC_PatientServicesRep` | Setup → Permission Sets | One per persona role |
| **Permission Set Group** | `HC_ClinicalRoles`, `HC_ContactCenter` | Setup → Permission Set Groups | Compose persona bundles |
| **Sharing Rule** | Criteria-based on `Account` (Patient/Member) | Sharing Settings | Complements FLS for PHI |
| **Apex Class** | `PatientEligibilityService`, `CareRequestReviewController` | force-app/main/default/classes | Include an `@AuraEnabled` where an LWC consumes it |
| **LWC** | `hcPatientTimeline`, `hcCareTeamBoard`, `hcVitalsDashboard` | force-app/main/default/lwc | Kebab-case dir; `hc` prefix for Health Cloud LWCs in this workspace |
| **Custom Object** | Avoid if a standard HC object fits | `Custom_Item__c` | Only when standard object cannot be extended |

---

## Component types — OmniStudio

| Type | Naming pattern | Notes |
|------|---------------|-------|
| **OmniScript** | `HC_PatientIntake`, `HC_UM_ReviewerWorkflow`, `HC_InHomeVisit` | Prefix with sub-domain if multi-domain |
| **FlexCard** | `FC_Patient_360`, `FC_CarePlan_Summary`, `FC_UM_QueueTile` | `FC_` prefix |
| **Integration Procedure** | `IP_PatientEligibility`, `IP_SubmitPriorAuth`, `IP_FhirPatientSync` | `IP_` prefix; reusable server orchestration |
| **Omnistudio Data Mapper** | `DM_Load_CareRequest_UM`, `DM_Extract_PatientTimeline`, `DM_Transform_FhirBundle`, `DM_TurboExtract_Roster` | `DM_<Type>_<Purpose>_<Persona>` |

**Data Mapper types:**
- **Turbo Extract** — fast read of a single sObject (best default for reads).
- **Extract** — flexible read across related objects, supports formulas.
- **Load** — atomic write of one or more related records.
- **Transform** — transformation-only; no read/write.

**Legacy note:** "DataRaptor" is the old name. In new stories, always write
"Omnistudio Data Mapper (Extract / Load / Transform / Turbo Extract)" — keep
"DataRaptor" only in a mapping note when migrating an existing story.

---

## OmniStudio runtime — Standard vs managed package

Health Cloud can run OmniStudio in two runtimes:

| Runtime | Description | When to choose |
|---------|-------------|----------------|
| **Standard runtime** (Recommended for new work) | OmniStudio components deploy as **native Salesforce metadata** — no `omnistudio__` namespace prefix. SF CLI-friendly, source-controlled naturally, **Agentforce-ready**. | Any new build. |
| **Managed-package runtime** (Legacy / Vlocity) | The pre-native, `vlocity_ins__` / `omnistudio__` managed-package runtime. Maintenance mode; not receiving major new features. | Only when the target org already runs the managed package and migration is out of scope for this story. |

Always confirm the runtime in Phase 3 (Q13 in `SKILL.md`). Note the choice in
Technical Implementation. Default to **Standard runtime** unless there's a
reason not to.

---

## Health Cloud naming conventions

- **Custom fields:** `HC_CareCoordinatorRegion__c` on `Account` (Patient); prefix with `HC_` for Health-Cloud-specific fields. Standard-object custom fields on `Case` follow the org's global convention.
- **Record types:** `Patient`, `Member`, `Caregiver`, `HealthcareProvider`, `HealthcareFacility`, `Payer` on `Account`. `PatientServices`, `AppealsIntake`, `PriorAuthIntake`, `RefillRequest` on `Case`.
- **Screen Flows:** `HC_<Verb><Subject>` — `HC_LogPatientCall`, `HC_SubmitPriorAuth`, `HC_IntakeNewPatient`.
- **Record-Triggered Flows:** `<Object>_<Behavior>` — `CareRequest_AutoAssign`, `CareObservation_AlertOnOutOfRange`.
- **Permission sets:** `HC_<Role>` — `HC_CareCoordinator`, `HC_UM_Reviewer`, `HC_NetworkManager`, `HC_PatientServicesRep`.
- **Apex classes:** `<Domain><Purpose>Service` / `Controller` — `PatientEligibilityService`, `CareRequestReviewController`.
- **LWCs:** kebab-case dir, `hc` prefix — `hc-patient-timeline`, `hc-care-team-board`.
- **OmniStudio:** see the OmniStudio table above.

---

## PHI / HIPAA considerations by component type

| Component | PHI concern | Mitigation |
|-----------|-------------|------------|
| **Standard record page** | Field visibility | Use FLS + permission sets; sensitive fields (SSN, DOB masking) gated |
| **Report / Report Type** | Bulk PHI export | Use "Only users assigned to selected report folders can view reports" + audit report exports |
| **List View** | Enumerating patients | Owner-based sharing + list-view filters; consider row-limit |
| **Screen Flow** | Displays PHI in mid-flow screens | Screen-level FLS respected; capture user in the audit log |
| **LWC** | PHI in browser | No PHI in `console.log`; sanitize error messages sent to Sentry/etc. |
| **Apex** | PHI in debug logs | Never `System.debug()` PHI at Info+; use Fine when unavoidable |
| **OmniScript** | Offline cache on iPad | Encrypt at rest (Mobile App Encryption); wipe on logout |
| **Integration Procedure / Data Mapper** | PHI in cache | Disable IP cache for PHI-carrying results; use per-user cache scope |
| **MuleSoft / Ext callout** | PHI over the wire | TLS 1.2+; mutual auth to EHR/FHIR endpoints; log correlation IDs, never PHI |
| **Chatter** | PHI in feeds | Turn off Chatter on Patient records unless clinically justified; educate users |
| **Reports & Analytics** | Bulk analytics | Field-level encryption for at-rest; Shield Platform Encryption for high-risk fields |
| **Data Cloud** | Unified profile | Configure DMO field-level access; align consent DMOs |

**Always required (RULE 16):** a Pattern-A audit AC (Given/When/Then that
verifies a `SetupAuditTrail` or custom audit-log entry is written) AND a
Pattern B/C spec for the field-level security / permission set that gates
access, on any story that displays, logs, exports, or sends PHI.

---

## References

- [Salesforce Health Cloud](https://help.salesforce.com/s/articleView?id=sf.health_cloud.htm)
- [Health Cloud Object Reference](https://developer.salesforce.com/docs/atlas.en-us.health_cloud_object_reference.meta/health_cloud_object_reference/sforce_api_objects.htm)
- [Health Cloud Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.health_cloud_dev_guide.meta/health_cloud_dev_guide/)
- [OmniStudio for Life Sciences / Health](https://help.salesforce.com/s/articleView?id=xcloud.os_omnistudio_standard.htm&type=5)
- [Utilization Management](https://help.salesforce.com/s/articleView?id=sf.hc_um_overview.htm)
- [Care Plan](https://help.salesforce.com/s/articleView?id=sf.hc_admin_careplan.htm)
- [FHIR R4 for Health Cloud](https://developer.salesforce.com/docs/atlas.en-us.health_cloud_fhir_r4.meta/health_cloud_fhir_r4/)
- [Consent Management](https://help.salesforce.com/s/articleView?id=sf.hc_admin_consent.htm)
- [Shield Platform Encryption](https://help.salesforce.com/s/articleView?id=sf.security_pe_overview.htm)
