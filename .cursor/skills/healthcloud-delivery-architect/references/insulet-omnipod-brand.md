# Insulet / OmniPod — Brand Skin for Health Cloud Prototypes

**Purpose.** When the Health Cloud Delivery Architect skill is invoked inside an
Insulet workspace or on an Insulet / OmniPod capability, prototypes produced under
§6.7 must adopt the OmniPod visual identity **on top of** the SLDS 2 baseline
established in `slds2-healthcloud-primer.md` — not replace it. omnipod.com itself
is built on SLDS 2, so the two systems overlay cleanly.

This file is the **canonical source** for those tokens, terminology, and legal
boilerplate. Copy the CSS block at the bottom into every Insulet prototype.

---

## When to apply this skin (auto-detection)

Turn on Insulet brand skinning when **any** of the following are true. Announce
which signal fired in the prototype's introduction line.

| Signal | Detection |
|--------|-----------|
| Workspace path contains `insulet` / `omnipod` / `podder` | Case-insensitive match on the workspace root path or `.git/config` remote URL |
| `sfdx-project.json` mentions Insulet | Grep `sfdx-project.json` for `"insulet"` or `"omnipod"` |
| A `.cursor/rules/insulet-brand.mdc` (or similar) file is present in the workspace | Any file named `insulet*` or `omnipod*` under `.cursor/rules/` |
| The user's prompt mentions OmniPod / Insulet product terms | Case-insensitive `omnipod`, `insulet`, `podder`, `PDM`, `pod (device)`, `SmartAdjust`, `SmartBolus`, `PodderCentral` |
| The story's sub-domain is Patient Services **and** the therapeutic area is insulin-pump therapy / diabetes device support | Human judgement from Phase 2 answers |
| The user explicitly asks: "apply the Insulet skin" / "brand this as OmniPod" | Explicit instruction wins over all others |

When **no** signal fires, use the SLDS 2 defaults from `slds2-healthcloud-primer.md`
unchanged — do NOT apply this skin to non-Insulet Health Cloud work (Aetna,
UnitedHealth, generic care management, etc.).

---

## Brand identity (voice & tone)

| Attribute | Guidance |
|-----------|----------|
| **Brand promise** | *"Insulin Pump Therapy, Simplified."* / *"Simplify Life."* — reduce cognitive load, celebrate the everyday. |
| **Voice** | Warm, plainspoken, encouraging. Never clinical-cold. Never salesy. |
| **Prototype tone** | Skip diabetes-industry jargon in visible UI text where possible; when medical terminology is required (basal, bolus, IOB, A1c, CGM, TIR), use it precisely — Podders read documentation and know these terms. |
| **Do say** | *"Podder"* (customer/patient), *"Pod"* (the disposable), *"PDM"* / *"Controller"* (the handheld/app), *"CGM sensor"*, *"in Range" / "Time in Range"* |
| **Do not say** | *"Insulin pump user"* (use *Podder*), *"cartridge"* (use *Pod*), *"remote"* (use *PDM* or *Controller*) |

---

## Terminology cross-walk — OmniPod → Health Cloud

Prototypes should surface OmniPod-native product names on the UI while the
underlying Salesforce object stays Health Cloud-native. Never invent a custom
object where a standard HC object fits.

| OmniPod term (surface to Podder / PSR) | Health Cloud object / field |
|----------------------------------------|-----------------------------|
| **Podder** (a person using an OmniPod) | `Account` with **Person Account** record type (or `Individual`) — HC standard `Patient` model |
| **Podder ID** | `Account.OmnipodPodderID__c` (custom formula) or `Account.MedicalRecordNumber__c` |
| **Pod** (the wearable Pod) | `Asset` (SKU tied to the Podder) — one active Pod at a time; historical Pods stay on the record |
| **PDM / Controller / Omnipod 5 App** | `Asset` (parent to Pods) OR `AssetRelationship` |
| **Pump alarm / Pod alarm** | `Case` with record type = **Device Alarm** + `Case.Call_Reason__c = 'Pump Alarm'` |
| **Re-supply order** | `Order` (linked to Podder Account) — kept separate from `Case` |
| **PodderCentral account** | External / Experience Cloud `User` linked to the Podder `Account` |
| **CGM sensor reading** | `HealthConditionDetail` or a custom `CGMReading__c` (choose based on retention window) |
| **Bolus event / basal event** | `PatientMedicationDosage` (native HC) or a custom `InsulinDelivery__c` |
| **Prescriber (endocrinologist)** | `HealthcareProvider` (standard HC) |
| **Pharmacy / DME channel** | `Account` with record type = **Pharmacy** or **DME Supplier** |
| **Insurance / benefits** | `HealthInsurance` + `MemberPlan` + `CoverageBenefit` (standard HC payer model) |
| **Care team (nurse educator + endo)** | `PatientCareTeam` + `PatientCareTeamMember` (standard HC) |

**Rule:** never introduce a custom object named `Podder__c` or `Pod__c` when
`Account` + record type or `Asset` covers the semantics. Use custom fields on
standard objects to carry OmniPod-specific attributes.

---

## Color tokens (extracted from omnipod.com CSS, 2026-08-11)

omnipod.com uses SLDS 2 as its base and overlays a small set of Insulet-branded
accents. These are the tokens verified from the live stylesheet:

### Primary — leverages SLDS 2 blue (no override needed)

| Token | Hex | Usage |
|-------|-----|-------|
| `--slds-g-color-brand-base-50` | `#0176d3` | Primary CTA, links, focus rings (SLDS baseline) |
| `--slds-g-color-brand-base-40` | `#014486` | Hover, active states |
| `--slds-g-color-brand-base-70` | `#0b5cab` | Deep-brand backgrounds |

### Secondary — Insulet grape (distinctive brand accent)

| Token | Hex | Usage |
|-------|-----|-------|
| `--insulet-c-grape-50` | `#743DBC` | **Distinctive Omnipod accent** — success/positive framing, "Set yourself free" callouts, hero backdrops |
| `--insulet-c-grape-60` | `#8250C3` | Slightly lighter grape (hover state) |
| `--insulet-c-grape-70` | `#8D61C8` | Grape used on decorative panels |
| `--insulet-c-grape-tint` | `#C7B1E4` | Backdrops behind grape typography |

### Tertiary — sunlight (the "100" digit accent on the Pod controller image)

| Token | Hex | Usage |
|-------|-----|-------|
| `--insulet-c-sunlight-50` | `#FFA700` | Positive numeric highlights (glucose target 100 mg/dL, "In Range" badges) |
| `--insulet-c-coral-50` | `#F75E4C` | Warning / attention (Podder-facing, less severe than error) — used sparingly |

### Info / semantic (leans on omnipod.com's `--info-color`)

| Token | Hex | Usage |
|-------|-----|-------|
| `--insulet-c-info-50` | `#1AD1DB` | Info banners, informational callouts (bright cyan-teal) |
| `--insulet-c-info-tint` | `#D2F6F8` | Info banner backgrounds |

### Neutrals (Omnipod warm palette)

| Token | Hex | Usage |
|-------|-----|-------|
| `--insulet-c-black` | `#1B1A1A` | Body text, headings (warm off-black, softer than pure `#000`) |
| `--insulet-c-gray-body` | `#4D4D4D` | Secondary text |
| `--insulet-c-gray-border` | `#BBBBBB` | Borders, dividers |
| `--insulet-c-gray-lt` | `#E8E8E8` | Card backgrounds, disabled surfaces |
| `--insulet-c-footer-bg` | `#F1EFE9` | Warm off-white — footer bar, page wash |
| `--insulet-c-footer-sep` | `#8D8C8C` | Footer separators |

### Semantic — errors (Insulet-branded)

| Token | Hex | Usage |
|-------|-----|-------|
| `--insulet-c-error-50` | `#BB0000` | Error text / stop icons |
| `--insulet-c-error-tint` | `#F1CCCC` | Error banner backgrounds |

---

## Typography

Insulet's site uses two Google-hosted webfonts, both are free to embed in a prototype.

| Face | Weight range | Usage |
|------|--------------|-------|
| **IBM Plex Sans** *(primary — verified via `--ibm-plex-sans-font`)* | 300, 400, 500, 600, 700 | Body copy, UI labels, form controls |
| **Open Sans** *(secondary — verified via `--open-sans-font`)* | 400, 600, 700 | Long-form content, marketing copy |

**Fallback stack:** `"IBM Plex Sans", "Helvetica Neue", Helvetica, Arial, sans-serif`

For headline/hero moments only, Insulet also uses **Apis Medium** (verified in
their CSS as `"Apis Medium","noto-sans",Arial,...`). Apis is not a free webfont;
substitute IBM Plex Sans SemiBold for hero headings in prototypes.

**Type scale (recommended for HC prototypes):**

| Role | Size | Weight | Line-height |
|------|------|--------|-------------|
| Display / hero | 40px | 600 | 1.15 |
| H1 page title | 24px | 600 | 1.2 |
| H2 section | 18px | 600 | 1.3 |
| H3 card | 14px | 600 | 1.4 |
| Body | 14px | 400 | 1.5 |
| Small / meta | 12px | 400 | 1.45 |
| Caption | 11px | 500 | 1.4 |

---

## Iconography & imagery

- **Product photography:** the Pod on skin (arm, abdomen, thigh) — **do not** draw the Pod as a cartoon disc. Use a placeholder box labelled *"Pod photography (lifestyle)"* rather than inventing generic clip art.
- **Icons:** use SLDS 2 utility & standard icons unchanged (`utility:success`, `standard:contact`, etc.) — omnipod.com does the same. Don't invent bespoke icons.
- **Illustration accents:** rounded, human-scale, warm palette. When mocking a "Podder" avatar, use a first name + colour swatch, not stock photography.
- **Data viz colour ramp (CGM/glucose):**
  - In Range (70–180 mg/dL) → `--insulet-c-grape-50` (`#743DBC`)
  - Above Range → `--insulet-c-sunlight-50` (`#FFA700`)
  - Below Range / Low → `--insulet-c-coral-50` (`#F75E4C`) or `--insulet-c-error-50` (`#BB0000`) if severe
  - Neutral / no data → `--insulet-c-gray-lt` (`#E8E8E8`)

---

## Legal, footer, and safety boilerplate

Any Insulet prototype that displays **outside a purely internal PSR desktop
context** (i.e. anything Podder-facing) MUST include the two mandatory notices
that appear on omnipod.com:

1. **Trademark disclosure (footer):**
   > *Omnipod, the Omnipod logos, Omnipod DASH, Omnipod 5, SmartAdjust, SmartBolus, Podder, PodderCentral, Simplify Life, and Toby the Turtle are trademarks or registered trademarks of Insulet Corporation. All rights reserved. ©2018–2026 Insulet Corporation.*

2. **Safety-info link (footer, must be visible without scrolling):**
   > *Important Safety Information — visit [omnipod.com/safety](https://www.omnipod.com/safety) for indications, contraindications, warnings, cautions, and instructions.*

3. **HIPAA notice:** since Health Cloud prototypes handle PHI, always include the standard PHI banner from `slds2-healthcloud-primer.md` PLUS a link to Insulet's [HIPAA Privacy Notice](https://www.omnipod.com/hipaa-privacy-notice).

4. **Customer support number** (contact-center prototypes only): **1-800-591-3455** (Insulet Customer Support, verified on omnipod.com).

Internal PSR-desktop prototypes may omit items 1–2 but must keep item 3 (HIPAA).

---

## Ready-to-paste CSS override block

Paste this block into the `<style>` element of any Insulet-skinned prototype
**immediately after** the SLDS 2 primer's `:root` block. It overlays Insulet
tokens without breaking SLDS semantics.

```html
<!-- Insulet / OmniPod brand skin — verified against omnipod.com CSS 2026-08-11.
     Overlays SLDS 2 baseline; do not remove the SLDS primer :root above this block. -->
<style>
  @import url("https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:wght@300;400;500;600;700&family=Open+Sans:wght@400;600;700&display=swap");

  :root {
    /* Insulet brand accents (SLDS-compatible token names) */
    --insulet-c-grape-50:   #743DBC;
    --insulet-c-grape-60:   #8250C3;
    --insulet-c-grape-70:   #8D61C8;
    --insulet-c-grape-tint: #C7B1E4;

    --insulet-c-sunlight-50: #FFA700;
    --insulet-c-coral-50:    #F75E4C;

    --insulet-c-info-50:   #1AD1DB;
    --insulet-c-info-tint: #D2F6F8;

    --insulet-c-black:      #1B1A1A;
    --insulet-c-gray-body:  #4D4D4D;
    --insulet-c-gray-border:#BBBBBB;
    --insulet-c-gray-lt:    #E8E8E8;
    --insulet-c-footer-bg:  #F1EFE9;
    --insulet-c-footer-sep: #8D8C8C;

    --insulet-c-error-50:   #BB0000;
    --insulet-c-error-tint: #F1CCCC;

    /* Typography */
    --insulet-font-primary:   "IBM Plex Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
    --insulet-font-secondary: "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  }

  /* Global type + surface (Insulet warm neutrals overlay SLDS defaults) */
  body {
    font-family: var(--insulet-font-primary);
    color: var(--insulet-c-black);
    background: var(--insulet-c-footer-bg);
  }

  /* App chrome — Insulet uses a subtle grape-tinted brand bar rather than SF blue */
  .sfheader, .buildbanner {
    background: linear-gradient(90deg, var(--insulet-c-grape-50) 0%, #5A2A9A 100%);
    border-bottom-color: var(--insulet-c-grape-70);
  }

  /* Primary CTA — leave SLDS blue as the primary action colour to preserve
     platform affordance, but tune to Insulet's exact button styling */
  .btn-primary {
    background: var(--slds-g-color-brand-base-50, #0176d3);
    border-color: var(--slds-g-color-brand-base-50, #0176d3);
  }
  .btn-primary:hover { background: var(--slds-g-color-brand-base-40, #014486); }

  /* Success / positive framing uses grape (distinctive Insulet accent) */
  .tag-pill.green, .toast, .sla-time,
  .path li.done { background: var(--insulet-c-grape-50) !important; color: #fff; }

  /* Warning uses sunlight, not orange */
  .sla-time.warn, .tag-pill.orange { color: var(--insulet-c-sunlight-50); }

  /* Info banners use Insulet cyan-teal */
  .toast.info, .phi-banner {
    background: var(--insulet-c-info-tint);
    border-left-color: var(--insulet-c-info-50);
  }

  /* Body copy + dividers */
  .muted, .small { color: var(--insulet-c-gray-body); }
  .card, .actionbar, .flow-modal { border-color: var(--insulet-c-gray-border); }
  .card-head { background: var(--insulet-c-footer-bg); }

  /* Insulet footer boilerplate — apply the .insulet-footer class */
  .insulet-footer {
    background: var(--insulet-c-footer-bg);
    color: var(--insulet-c-gray-body);
    border-top: 1px solid var(--insulet-c-footer-sep);
    padding: 16px 20px;
    font-size: 11px;
    line-height: 1.6;
  }
  .insulet-footer a { color: var(--insulet-c-grape-50); }
  .insulet-footer .tm { margin-top: 8px; font-size: 10px; color: var(--insulet-c-footer-sep); }
</style>
```

---

## Mandatory footer HTML block

Every Insulet-skinned prototype must include this footer at the very bottom of
`<body>` (or its logical equivalent):

```html
<footer class="insulet-footer">
  <div>
    <strong>Important Safety Information</strong> —
    Visit <a href="https://www.omnipod.com/safety">omnipod.com/safety</a> for
    indications, contraindications, warnings, cautions, and instructions.
    HIPAA Privacy Notice:
    <a href="https://www.omnipod.com/hipaa-privacy-notice">omnipod.com/hipaa-privacy-notice</a>.
    Customer Support: <strong>1-800-591-3455</strong>.
  </div>
  <div class="tm">
    Omnipod, the Omnipod logos, Omnipod DASH, Omnipod 5, SmartAdjust, SmartBolus,
    Podder, PodderCentral, Simplify Life, and Toby the Turtle are trademarks or
    registered trademarks of Insulet Corporation. ©2018–2026 Insulet Corporation.
    Prototype — not for clinical use.
  </div>
</footer>
```

---

## Prototype checklist (Insulet skin engaged)

Add these to the standard §6.7 review checklist:

- [ ] CSS override block pasted below the SLDS primer `:root`.
- [ ] Google Fonts import for **IBM Plex Sans** + **Open Sans** included.
- [ ] Header/chrome uses the grape gradient (not the default SF navy).
- [ ] Success/positive states use grape (`#743DBC`), not the default SLDS green (unless the story is clinically about a positive lab result — then keep green with a small grape accent).
- [ ] "In Range" glucose framing uses the Insulet CGM colour ramp (grape / sunlight / coral).
- [ ] OmniPod-native product terms (*Podder*, *Pod*, *PDM*, *Controller*, *PodderCentral*) surface in UI copy — never generic *"user"* / *"device"* / *"remote"*.
- [ ] Health Cloud object names (Account, Case, ContactEncounter, HealthcareProvider) still label every element in the build-tech chip — do NOT swap those for OmniPod product names.
- [ ] Mandatory footer block included (Trademark + Safety Info + HIPAA link + Customer Support number).
- [ ] Prototype introduction line names the Insulet-detection signal that fired (e.g. *"Insulet skin engaged — workspace path contains `insulet`."*).
- [ ] "Prototype — not for clinical use." watermark visible in footer.

---

## Version history

| Version | Date | Changes |
|---------|------|---------|
| **v1.0** | 2026-08-11 | Initial file, extracted from omnipod.com CSS. First supported org-brand skin. |
