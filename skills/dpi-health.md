---
name: dpi-health
description: DPI for Healthcare — health identity, HIE, OpenHIE, FHIR R4, consent-based records, telemedicine, disease surveillance. Use when designing health DPI or evaluating health DPGs.
---

# DPI for Healthcare

> **Reference**: https://cdpi.gitbook.io/dpi-for-healthcare

## Health as Public Infrastructure

Healthcare delivery depends on information flows: a patient's identity verified, their records accessible, their consent captured, their prescriptions fulfilled, their insurance claimed, and their outcomes counted. When these flows are siloed, care is fragmented, costly, and inequitable.

**DPI reframes health digitization**: instead of each hospital building its own app, countries build **shared health rails** — open, standards-based infrastructure that every provider, payer, and patient app can connect to. The result is interoperable care at population scale.

```
Health DPI Stack
  ├── Identity Rail       → Patient identity (health ID linked to foundational ID)
  ├── Data Exchange Rail  → Consent-based health record sharing (HIE)
  ├── Payments Rail       → Insurance claims, G2P health benefits
  └── Discovery Rail      → Telemedicine, provider/facility discovery
```

**The health DPI difference**: Private health apps come and go. Health DPI persists because it is owned by governments, governed as a public good, and accessible to any authorized actor — from a rural community health worker to a tertiary hospital.

---

## Health DPI Building Blocks

CDPI identifies seven foundational building blocks for health DPI. All seven must interoperate for health DPI to deliver at population scale.

```
┌──────────────────────────────────────────────────────────────────┐
│                      Health DPI Building Blocks                  │
│                                                                  │
│  1. Health Identity      2. Health Registries                    │
│     (Health ID / UHI)       (Facility, Provider, Drug)          │
│                                                                  │
│  3. Health Information   4. Shared Health Record                 │
│     Exchange (HIE)          (Longitudinal patient record)        │
│                                                                  │
│  5. Consent Manager      6. Telemedicine / Discovery             │
│     (health-specific)       (provider ↔ patient matching)        │
│                                                                  │
│  7. Disease Surveillance / Public Health Analytics               │
│                                                                  │
│  Underlying Rails: Digital Identity + Payments + Data Exchange   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Building Block 1: Health Identity (Health ID)

Every patient needs a **unique, persistent health identifier** that travels with them across providers, systems, and life stages — without requiring re-registration at every touchpoint.

**Design principles:**
- Linked to (but distinct from) foundational national ID — protects privacy; health data is sensitive
- One-way pseudonymous token from national ID → health ID prevents cross-sector re-identification
- Patient controls which providers can access their health record
- Health ID ≠ health record — the ID is a key; records are distributed across providers

**Architecture pattern:**
```
National ID System
        ↓ authenticated consent
Health ID System (Client Registry / Master Patient Index)
        ↓ issues Health ID (pseudonymous token)
Patient's Health App / Card
        ↓ presents at provider
Provider System → fetches records from HIE using Health ID
```

**Country example — India ABHA (Ayushman Bharat Health Account):**
- 740M+ ABHA addresses generated (as of 2024)
- 14-digit health identifier; linked to Aadhaar but with a separate namespace
- Patient controls consent for each record-sharing episode
- Any ABDM-certified health app can issue/receive records

**Standards:**
- Master Patient Index (MPI) / Client Registry per OpenHIE spec
- HL7 FHIR R4 Patient resource
- IHE PIXm / PDQm (Patient Identity Cross-referencing)

---

## Building Block 2: Health Registries

Registries establish the **trusted reference data** that all health systems share. Three core registries:

### Facility Registry
- Authoritative list of every health facility (hospital, clinic, lab, pharmacy, health post)
- Each facility has a unique identifier, type, location, contact, operating hours
- Used by: routing of patient referrals, supply chain, disease surveillance, payment claims
- **Standard**: OpenHIE Facility Registry spec; mFacility; GeoHIV

### Provider Registry (Health Worker Registry)
- Authoritative registry of all licensed health professionals
- Links provider ID to qualifications, license status, specialty, facility affiliation
- Used by: prescription validation, telemedicine credentialing, referral routing
- **Standard**: OpenHIE Health Worker Registry; iHRIS (open-source DPG)

### Drug/Commodity Registry
- Standardized list of medicines, vaccines, medical devices with codes
- Used by: prescription systems, supply chain, claims adjudication
- **Standards**: ICD-10/11, SNOMED CT, RxNorm, WHO essential medicines list, GS1

### Population/Patient Registry
- National client registry (MPI): uniquely identifies all patients receiving health services
- De-duplication across providers
- Linked to the foundational ID and health ID
- **Standard**: IHE PIXm, FHIR Patient resource

---

## Building Block 3: Health Information Exchange (HIE)

An HIE enables **authorized parties to securely share patient health records** across institutions and geographies — without centralizing all data.

**Architecture model (OpenHIE):**
```
Provider A (Hospital EHR)      Provider B (Lab System)
       ↓ share                         ↓ share
  ┌─────────────────────────────────────────────┐
  │     Interoperability Layer (OpenHIM)         │
  │  - Message routing and transformation        │
  │  - Audit logging (ATNA)                      │
  │  - Security and authentication               │
  └─────────────────────────────────────────────┘
               ↓ stores
  Shared Health Record (SHR) / Longitudinal Record
               ↑ fetches with consent
          Patient / Provider C
```

**OpenHIE components:**
| Component | Function |
|-----------|---------|
| **OpenHIM** | Interoperability layer / mediator bus; audit, routing, auth |
| **Client Registry** | Master patient index; links records across providers |
| **Shared Health Record** | Central or federated document repository |
| **Terminology Service** | Standardizes codes (ICD-10, SNOMED, LOINC) across systems |
| **Facility Registry** | Authoritative facility list |
| **Health Worker Registry** | Provider credentials |

**Federated vs. centralized HIE:**
- **Federated (preferred)**: Records stay at the source provider; HIE only stores pointers and handles routing
- **Central SHR**: All records stored centrally — simpler, but higher data sovereignty risk
- **Hybrid**: Summary records centrally; detailed records federated at providers

---

## Building Block 4: Shared Health Record (Longitudinal Record)

A **longitudinal health record** assembles a patient's complete health history across all providers, over their lifetime.

**Contents of a longitudinal record:**
- Demographics (name, DOB, gender, address)
- Encounters (visits, hospitalisations)
- Diagnoses (ICD-10/11 coded)
- Medications (prescriptions, dispensing)
- Lab results (LOINC-coded observations)
- Imaging (DICOM references)
- Immunisation history
- Allergies
- Vital signs
- Care plans

**FHIR R4 as the universal format:**

Every clinical document in a health DPI system should conform to HL7 FHIR R4:

```json
{
  "resourceType": "Bundle",
  "type": "document",
  "entry": [
    { "resource": { "resourceType": "Composition", ... }},  // document metadata
    { "resource": { "resourceType": "Patient", ... }},       // patient identity
    { "resource": { "resourceType": "Encounter", ... }},     // visit details
    { "resource": { "resourceType": "Condition", ... }},     // diagnosis (ICD-10)
    { "resource": { "resourceType": "MedicationRequest", ... }}, // prescription
    { "resource": { "resourceType": "Observation", ... }}    // lab result (LOINC)
  ]
}
```

**Key FHIR R4 resources for health DPI:**

| Resource | Use |
|---------|-----|
| Patient | Patient demographics, identifier |
| Practitioner | Provider identity |
| Organization | Facility/institution |
| Encounter | Clinical visit or episode |
| Condition | Diagnosis (ICD-10/11) |
| Observation | Lab results, vitals (LOINC) |
| MedicationRequest | Prescription |
| Immunization | Vaccination record |
| DiagnosticReport | Lab report with results |
| DocumentReference | Pointer to external document |

---

## Building Block 5: Consent Manager (Health-Specific)

Health data is uniquely sensitive. **Purpose-specific, revocable, auditable consent** is required for every health data flow.

**Health consent properties:**
- **Granular**: Patient specifies which records (by type, date range, provider) can be shared
- **Purpose-limited**: GP referral consent ≠ insurance underwriting consent
- **Emergency override**: Break-glass access for emergency care (audited, notified)
- **Minor consent**: Guardian consent for children; progressive autonomy as age increases
- **Revocable**: Patient can withdraw consent; data access stops immediately

**Consent flow (DEPA/ABDM model):**
```
Health Information User (HIU: doctor, insurer, researcher)
        ↓ consent request (purpose, data type, duration)
Health Consent Manager (data-blind intermediary)
        ↓ presents consent to patient
Patient reviews and approves (mobile app)
        ↓ signed consent artefact
Consent Manager → Health Information Provider (hospital, lab)
HIP sends records directly to HIU (encrypted)
        [Consent Manager never sees the records]
```

**FHIR Consent resource** — the machine-readable consent artefact:
```json
{
  "resourceType": "Consent",
  "status": "active",
  "scope": { "coding": [{ "code": "patient-privacy" }] },
  "category": [{ "coding": [{ "code": "INFAO", "display": "Information Access" }] }],
  "patient": { "reference": "Patient/health-id-123" },
  "dateTime": "2024-01-15T10:00:00Z",
  "performer": [{ "reference": "Patient/health-id-123" }],
  "organization": [{ "reference": "Organization/consent-manager-id" }],
  "provision": {
    "type": "permit",
    "period": { "start": "2024-01-15", "end": "2025-01-15" },
    "purpose": [{ "code": "TREAT", "display": "Treatment" }],
    "class": [{ "code": "Observation" }, { "code": "MedicationRequest" }]
  }
}
```

---

## Building Block 6: Telemedicine and Provider Discovery

**The DPI approach to telemedicine**: instead of a single telemedicine platform, an **open discovery network** (Beckn-protocol-based) allows patients to find and consult any registered provider on any app.

```
Patient App (BAP)
        ↓ search: "cardiologist, available today, within 20km"
Beckn Gateway → all registered telemedicine BPPs
        ↓ responses from Provider 1, 2, 3...
Patient selects provider → init → confirm
        ↓
Video consultation via the platform
        ↓ post-consultation
FHIR DocumentReference → Health record updated in SHR
Prescription → sent to pharmacy discovery network
```

**India UHI (Unified Health Interface)**: Beckn-based open protocol for healthcare discovery and consultation — enables any health app to connect to any provider network.

**Components needed:**
- Provider registry (with telemedicine capability flag)
- Appointment scheduling FHIR resource
- Video consultation infrastructure (Jitsi, Zoom SDK, or WebRTC)
- Post-consultation FHIR document generation
- Prescription routing to pharmacy network

---

## Building Block 7: Disease Surveillance and Public Health Analytics

Aggregated, de-identified health data from the HIE powers population health intelligence — without exposing individual patient data.

**Architecture:**
```
Clinical Systems (HIS, EHR, Lab LIMS)
        ↓ aggregate / de-identified feeds
Health Analytics Platform (DHIS2, custom)
        ↓ real-time disease surveillance
Public Health Dashboards
        ↓ outbreak detection, response
National/WHO Reporting (IHR compliance)
```

**DHIS2** — the primary open-source DPG for health surveillance:
- Aggregate health data from facilities
- Individual-level tracker for case surveillance
- Real-time outbreak alerting
- Used in 73+ countries
- Integrates with OpenHIE via FHIR APIs

**Syndromic surveillance**: Aggregate lab results, emergency room visits, pharmacy sales → detect outbreaks before confirmed diagnoses.

---

## Health Claims and Insurance (Health Payments Rail)

Health insurance and benefits delivery requires the payments DPI rail:

```
Patient receives care
        ↓
Provider submits FHIR-based claim
        ↓
Insurance/National Health Authority
        ↓ validates eligibility via identity rail
        ↓ verifies claim via provider registry
        ↓ adjudicates claim
        ↓
Payment via payment rail → provider account
        ↓
G2P health benefit → patient account (co-pay, subsidy)
```

**India NHCX (National Health Claims Exchange)**: Beckn-based open claims exchange network — any insurer, any provider, standardized FHIR claims.

**Standards**: HL7 FHIR R4 Claim resource, X12 (US), EDIFACT (global), ICD-10 procedure codes.

---

## Open-Source Health DPGs

| DPG | Function | Language/Stack |
|-----|---------|---------------|
| **OpenMRS** | Electronic health records platform | Java, Spring |
| **DHIS2** | Health information management, surveillance | Java, Angular |
| **Bahmni** | Integrated hospital system (OpenMRS + ODOO) | Java |
| **OpenHIM** | HIE interoperability middleware | Node.js |
| **iHRIS** | Health worker registry and HR management | PHP |
| **OpenCRVS** | Civil registration (birth/death) — see dpi-opencrvs skill | TypeScript |
| **SORMAS** | Disease surveillance (outbreaks, contact tracing) | Java |
| **OpenLMIS** | Logistics management (vaccines, medicines supply) | Java, Angular |
| **GNU Health** | Hospital and public health system | Python |
| **HAPI FHIR Server** | Reference FHIR R4 server | Java |
| **Hearth** | Lightweight FHIR R4 datastore | Node.js |

---

## Data Standards for Health DPI

| Standard | Use in Health DPI |
|---------|------------------|
| **HL7 FHIR R4** | Universal health data format; all clinical documents |
| **ICD-10 / ICD-11** | Diagnosis coding (WHO standard) |
| **LOINC** | Lab observation codes (tests, results) |
| **SNOMED CT** | Clinical terminology (diagnoses, procedures) |
| **RxNorm** | Drug coding (prescriptions, dispensing) |
| **DICOM** | Medical imaging |
| **GS1** | Medical supply chain (product codes, barcodes) |
| **SMART on FHIR** | App authorization on top of FHIR APIs |
| **IHE profiles** | Integration patterns (PIXm, PDQm, MHD, ATNA) |
| **WHO SMART Guidelines** | L3 machine-readable clinical guidelines as FHIR |

---

## Privacy Architecture for Health DPI

Health data requires stronger privacy protections than most DPI:

1. **Pseudonymization**: Health ID ≠ national ID — one-way token prevents cross-sector profiling
2. **Purpose limitation**: Consent for treatment ≠ consent for research ≠ consent for insurance
3. **Data minimization**: Providers see only what they need for the consented purpose
4. **Break-glass access**: Emergency access overrides consent but is logged and patient is notified
5. **Retention limits**: Health records have statutory retention periods; auto-delete after expiry
6. **Anonymization for research**: Aggregate/de-identified data available without consent; differential privacy for small-n queries
7. **Audit trails**: Every access logged against consent artefact; patient can review who accessed what

---

## Country Implementations

| Country | Health DPI Component | Notes |
|---------|---------------------|-------|
| **India** | ABDM (full stack) | ABHA ID, FHIR HIE, UHI telemedicine, NHCX claims |
| **Rwanda** | OpenMRS national EHR | Rolled out across all district hospitals |
| **Tanzania** | OpenHIE + DHIS2 | Integrated HIS with facility registry |
| **Ethiopia** | DHIS2 national surveillance | Used for COVID response, routine surveillance |
| **Kenya** | KenyaEMR (OpenMRS) | HIV/AIDS patient management nationally |
| **Thailand** | National Health Exchange | FHIR-based HIE, 75M population |
| **Germany** | ePA (electronic patient record) | FHIR R4, patient-controlled, opt-in |
| **Australia** | My Health Record | National shared health record, 24M patients |
| **Ghana** | DHIS2 + OpenMRS + Zipline | Drone delivery integrated with digital supply chain |

---

## Choosing Health DPI Components

| Question | Recommendation |
|---------|---------------|
| Starting point for health DPI? | **Health ID + FHIR HIE** — identity and exchange before records |
| Need a hospital EHR? | **OpenMRS / Bahmni** (open-source) |
| Need national surveillance? | **DHIS2** |
| Need HIE middleware? | **OpenHIM** |
| Need health worker registry? | **iHRIS** |
| Need claims/insurance exchange? | Beckn-based claims network (UHI model) |
| Need patient consent management? | DEPA-based consent manager (see dpi-data-sharing-models skill) |
| Need telemedicine discovery? | Beckn-based open health network (UHI model) |
| Need FHIR server? | **HAPI FHIR** (Java) or **Hearth** (Node.js) |

---

## Key References

- **CDPI DPI for Healthcare**: https://cdpi.gitbook.io/dpi-for-healthcare
- **OpenHIE Architecture**: https://guides.ohie.org/arch-spec
- **WHO SMART Guidelines**: https://smart.who.int/ra/
- **HL7 FHIR R4**: https://hl7.org/fhir/R4/
- **DHIS2**: https://dhis2.org
- **OpenMRS**: https://openmrs.org
- **OpenHIM**: https://openhim.org
- **iHRIS**: https://www.ihris.org
- **India ABDM**: https://abdm.gov.in
- **HAPI FHIR**: https://hapifhir.io
