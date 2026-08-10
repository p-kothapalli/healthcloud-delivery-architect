# Evaluations — Health Cloud Delivery Architect

> Test scenarios + rubrics for the `healthcloud-delivery-architect` skill.
> Forked from the `lsc-delivery-architect` evaluations harness and re-scoped
> to Health Cloud personas, objects, and integrations.

## Structure

Each evaluation is a `.json` file with:

- `id` — Short identifier
- `mode` — Workflow mode being tested (New Feature / Refactor / Epic / Bug Fix / Legacy → HC Migration / Plan + Prototype)
- `prompt` — The exact user prompt
- `context` — Optional setup (files present, prior stories)
- `rubric` — What a correct output must include (hard blockers first, then quality items)

## Evaluations

| File | Mode | What it validates |
|------|------|-------------------|
| `eval-01-legacy-migration.json` | Legacy → HC Migration | Terminology translation from Salesforce Care Cloud legacy to native HC objects, persona correction (Care Coordinator, not Patient), Pattern A + Pattern E, Legacy Source header |
| `eval-02-patient-call-logging.json` | New Feature | Patient Services intake flow — persona (Care Coordinator, not Patient), Pattern A GWT, Pattern E on Case + ContactEncounter, PHI audit AC (RULE 16), declarative-first build-tech decision (Screen Flow, not OmniScript) |
| `eval-03-um-prior-auth.json` | New Feature | Utilization Management prior-auth triage — persona (Utilization Reviewer), Pattern D determination rules, Pattern E across CareRequest + CareRequestReview + CareRequestItem, PHI audit AC, SLA rules |

## Rubric — universal hard blockers

Every evaluation checks these regardless of scenario:

1. **Persona:** concrete Health Cloud role, not "user"/"business user"/"system". Patient / Caregiver / Provider used as subject, not persona (unless portal story).
2. **AC format:** every AC uses Pattern A/B/C/D/E. Pattern A = three explicit lines, one When, no prose.
3. **Business language:** no Apex class names, IP step numbers, SOQL, picklist API values, or `*__c` API names inside Pattern-A ACs.
4. **Technical Implementation section:** present after the ACs, concise, cross-references ACs.
5. **Pattern E per-field spec:** every "records created/updated" AC carries a Pattern E block enumerating every field on every object touched.
6. **PHI awareness (RULE 16):** any story that displays/logs/exports/sends PHI includes an audit AC + FLS/perm-set spec.
7. **Build-technology decision:** stated in the header (RULE 7a), declarative-first. OmniStudio choice justified with rejected-alternative note.
8. **No hallucinated component names:** proposed names marked `(proposed)` when the Health Cloud package isn't deployed in the workspace.
9. **Estimated Effort:** component-level sizing (S/M/L/XL/XXL) with an epic roll-up.
10. **Definition of Done:** present, checklist-style.

## Red flags — auto-fail

- Persona is "user" or "As a Patient" for an internal feature.
- Pattern A AC contains an Apex name, IP step, or SOQL.
- A Save/Submit AC has no Pattern E block, or the Pattern E abbreviates fields with "etc.".
- Legacy terminology retained in the story body during a migration (STEP 0 required translation).
- Prototype produced without a Build-Technology banner or per-element component labels (§6.7 hard blocker).
- PHI story missing an audit AC or permission-set spec.

## Running

The evaluations are run manually — no automated harness (yet). Paste the prompt
into a fresh Cursor session in a workspace that has the skill installed, then
score the output against the rubric.
