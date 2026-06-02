---
name: dpi-data-sharing-models
description: CDPI data sharing taxonomy — VCs, consent-based (DEPA), system-to-system APIs, anonymised open data. Use when choosing a data sharing model or designing consent architecture.
---

# DPI Data Sharing Models

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra

## The Data Sharing Challenge in DPI

Data about citizens is the raw material of public services — but it is siloed across government agencies, financial institutions, hospitals, and utilities. Individuals cannot easily access, share, or leverage their own data. Organisations cannot efficiently verify information they need.

The goal of data sharing DPI is not to centralise data — it is to create **trusted, auditable flows** of data between authorised parties, with the individual in control.

CDPI distinguishes two fundamentally different categories that require different architectures, policies, and governance:

```
Data Sharing Models
├── Personal Data                    Non-Personal Data
│   ├── Verifiable Credentials       ├── Anonymised Open Datasets
│   ├── Consent-Based Real-Time      ├── Aggregate Statistics
│   └── System-to-System APIs        └── Open Data for AI/ML Training
│
└── Core Infrastructure
    ├── Data Standards
    ├── Consent Standards
    └── Data Analytics Pipelines
```

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/a-primer-to-personal-data-sharing

## The Three Technical Building Blocks

For personal data sharing, CDPI identifies three foundational building blocks that must be in place:

1. **Data Sharing APIs** — standardised interfaces for data access requests and responses
2. **Sector-specific Data Schemas** — agreed data field definitions and formats (e.g. FHIR for health, ISO 20022 for finance)
3. **Consent Standards** — machine-readable, digitally signed consent artefacts (see dpi-trust-infra skill)

All three must exist for a data sharing model to work at scale. APIs without schemas produce inconsistent data. Schemas without consent standards produce data flows without accountability.

---

## Model 1: Verifiable Credentials (User-Led, Asynchronous)

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/verifiable-credentials  
> **See also**: verifiable-credentials skill

**How it works**: An authoritative source (government, university, bank) issues a digitally signed credential to the individual. The individual stores it in a digital wallet and presents it to any verifier on demand — **no contact with the issuer required at verification time**.

```
Issuer (Registry / Authority)
        ↓ issues signed VC
Holder (Citizen Wallet)
        ↓ presents VC on demand
Verifier (Bank, Employer, Government Service)
        ↓ verifies signature cryptographically
        [Issuer NOT contacted — offline verification possible]
```

**What it enables:**
- Proof of education, professional certification, business registration, income, health status
- Citizen is wholly in control of what they share and with whom
- Selective disclosure: share only the attributes the verifier needs
- Cross-border flows without bilateral agreements between issuers and verifiers

**CDPI framing**: *"A verifiable credential is a robust instrument of trust which is digitally signed, machine-readable, and intended to serve as proof. Any certificate/credential can be turned into and presented as a verifiable credential."*

**Best for:**
- Cross-border data flows (no bilateral issuer-verifier agreements needed)
- Offline / low-connectivity verification
- Citizen-empowerment use cases (education, employment, financial identity)
- Long-lived credentials (qualifications, registrations, licenses)

**Key standards**: W3C VC Data Model, SD-JWT VC, ISO 18013-5 (mDOC), OpenID4VCI/VP

---

## Model 2: Consent-Led Real-Time Data Sharing (DEPA)

**How it works**: A network facilitates the exchange of data from data providers to data consumers via the data principal (user), in real time, with no centralised data store. The user's consent is captured as a machine-readable artefact that authorises each data flow.

```
Data Consumer (Lender, Insurer, Government)
        ↓ consent request
Consent Manager (data-blind intermediary)
        ↓ presents consent to user
Data Principal (Citizen)
        ↓ approves consent artefact
Consent Manager → Data Provider (artefact + fetch request)
Data Provider → Data Consumer (encrypted data, direct)
        [Consent Manager NEVER sees the data]
```

**DEPA (Data Empowerment and Protection Architecture)**: The open framework defining this model, created by NITI Aayog, India. Predicated on individuals having control over how their personal data is used and shared.

### Participants

| Role | Description | Example |
|------|-------------|---------|
| **Data Provider (FIP)** | Holds citizen data; shares on consent | Banks, tax authorities, registries |
| **Data Consumer (FIU)** | Requests data to provide a service | Lenders, insurers, government portals |
| **Consent Manager (CM)** | Manages consent; data-blind | Account Aggregator, health consent manager |
| **Data Principal** | The citizen whose data is shared | Any individual |

### Account Aggregator (AA) — Financial Implementation

India's Account Aggregator framework is the world's most mature implementation of DEPA:
- **Regulatory basis**: RBI-licensed NBFCs (Non-Banking Financial Companies) act as AAs
- **Sahamati**: Non-profit collective governing the AA ecosystem
- **Scale**: 130+ banks as FIPs; hundreds of FIUs; millions of consented data flows
- **Impact**: Enables thin-file borrowers (MSMEs, farmers) to share cash flow data for credit without any paperwork

**Data types covered**: Bank accounts, mutual funds, insurance, GST returns, salary slips, pension data, tax returns

### ABDM — Health Implementation

Ayushman Bharat Digital Mission implements DEPA for health data:
- ABHA (Ayushman Bharat Health Account) is the health identifier
- Health Information Providers (HIP): hospitals, labs, pharmacies
- Health Information Users (HIU): doctors, insurers, researchers
- Patient consents to share records via ABDM consent manager
- Data format: FHIR R4

### Consent Artefact Properties

| Property | Definition |
|---------|-----------|
| **Granular** | Purpose, data fields, and duration all specified separately |
| **Revocable** | Citizen can withdraw at any time; data flows stop |
| **Time-bound** | Start, end, frequency, and total count of accesses defined |
| **Digitally signed** | Non-repudiable; immutable record |
| **Purpose-limited** | Data can only be used for the consented purpose |
| **Auditable** | Every access logged against the consent ID |

**Best for:**
- Financial data sharing (credit, insurance, wealth management)
- Health data flows (patient records, diagnostics, prescriptions)
- Any use case requiring granular, revocable, ongoing consent
- High-frequency data access (monthly bank statements for 12 months)

> **Implementation patterns**: Consent-based real-time sharing combines sync (consent approval) and async (data fetch) patterns. See the **dpi-data-exchange** skill for detailed sync vs async decision matrix, webhook patterns, and reliability patterns (idempotency, dead-letter queues, outbox pattern).

---

## Model 3: System-to-System via Open APIs

**How it works**: APIs of different issuing authorities are opened to enable real-time data fetch between systems. Data is fetched (not stored) based on API calls carrying the individual's consent or a pre-authorised purpose.

```
System A (Government Service)
        ↓ API call (with consent token or authorised purpose)
System B (Data Provider: Tax Authority, Registry, Social Registry)
        ↓ returns data in real time
System A processes and provides service
        [No data stored beyond immediate need]
```

**Open API standards for DPI data exchange:**
- **G2P Connect / DCI APIs**: Government-to-Person data exchange specifications
- **OAuth 2.0 / OpenID Connect**: Authorised access with scoped tokens
- **W3C VC**: When the "data" is a credential presentation
- **FHIR R4**: Health data APIs
- **ISO 20022**: Financial data messages

**Registries as data providers**: CDPI notes that registries operating as mature DPI can enable system-to-system access directly via open APIs. The first and simplest way is to generate Verifiable Credentials as a natural output of stored data — the two models are complementary.

**Examples:**
- Tax authority API → real-time income verification for loan application
- Population registry API → address/identity verification for government service
- Education registry API → degree verification for employment
- G2P Connect → interoperable beneficiary registry access across programs

**Best for:**
- Government-to-government (G2G) data exchange
- Service-level verification at point of interaction
- High-volume, automated eligibility checking
- Cases where the citizen is not present at the interaction (batch processing)

---

## Non-Personal / Anonymised Data

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/non-personal-anonymised-datasets

**What it is**: Aggregated, anonymised datasets that do not contain identifiable personal information, made available to researchers, AI/ML developers, and policymakers.

**Why it matters for DPI**: Anonymised data from DPI systems (transaction patterns, health outcomes, agricultural yields) can train AI/ML models that benefit the entire ecosystem — language translation, financial underwriting models, disease surveillance, crop failure prediction.

**CDPI framing**: *"Societies can generate open anonymised datasets to enable research across sectors, and these datasets can be used to train and publish open AI/ML models."*

### Privacy-Preserving Techniques

| Technique | How it works | Trade-offs |
|-----------|------------|-----------|
| **Aggregation** | Report group statistics, not individual records | Simple; useful for policy; loses granularity |
| **Anonymisation** | Remove/mask identifiers | Irreversible re-identification risk if done poorly |
| **Pseudonymisation** | Replace identifiers with tokens | Re-identifiable with the key; not truly anonymous |
| **Differential Privacy** | Add calibrated noise to outputs | Mathematically provable privacy guarantee; some accuracy loss |
| **Synthetic Data** | Generate statistically representative fake data | Preserves statistical properties; no real individuals |
| **Federated Analytics** | Compute statistics locally; share only aggregates | Data never leaves source; complex infrastructure |
| **Secure Multi-Party Computation** | Compute jointly without revealing inputs | Highest privacy; computationally expensive |

### Open Data Network Architecture

CDPI notes that a centralised approach to aggregated datasets is difficult to scale. A preferred model:

```
Federated Open Data Network (Beckn-protocol-based)
        ↓
Unified standards and APIs across all data custodians
        ↓
Discovery: Any researcher discovers available datasets
        ↓
Access: Standardised API access with appropriate licensing
        ↓
Compute: Optionally, compute-to-data (analysis runs at source)
```

**Key**: The data stays at the source custodian; only results or anonymised extracts move. This is "data exchange DPI for non-personal data."

---

## Data Standards

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/data-standards

Standards are what make data sharing interoperable at scale. Without agreed schemas, every data provider formats data differently — consuming systems cannot parse or compare it.

### Sector Data Standards

| Sector | Standard | Description |
|--------|---------|-------------|
| **Health** | HL7 FHIR R4 | Resource-based health data model; RESTful APIs |
| **Finance** | ISO 20022 | Financial messaging; rich, structured data |
| **Finance** | OBIE (UK Open Banking) | Account, transaction, product schemas |
| **Identity** | W3C VC Data Model | Verifiable credential schema |
| **Government** | G2P Connect DCI APIs | Government-to-person payment and data specs |
| **Agricultural** | AGROVOC | FAO thesaurus for agricultural data |
| **Statistical** | SDMX | Statistical Data and Metadata eXchange |

### Data Schema Governance Principles

1. **Publish schemas publicly**: Any party can implement without asking permission
2. **Version schemas**: Never break existing implementations; deprecate with notice
3. **Machine-readable schemas**: JSON Schema, OpenAPI 3.x, FHIR StructureDefinition
4. **Sector-neutral identifiers**: Use stable, globally unique identifiers (not internal system IDs)
5. **Localisation-aware**: Support multiple languages, scripts, and regional formats

---

## Data Analytics Pipelines

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/building-data-analytics-pipelines

DPI data exchange creates the infrastructure for analytics that were previously impossible — without centralising raw personal data.

### Privacy-Preserving Analytics Architecture

```
Data Sources (Distributed)         Analytics Layer
  Bank A records                         ↓
  Bank B records          →    Federated Learning / Secure Aggregation
  Tax records                            ↓
  Registry records               Aggregate Insights (no raw PII)
                                         ↓
                                  Policy / AI/ML Models
```

### Key Pipeline Components

**Ingestion**: Standardised APIs (FHIR, ISO 20022, G2P Connect) pull consented data
**Normalisation**: Transform sector-specific formats to a common analytical schema
**Anonymisation/Aggregation**: Apply privacy-preserving techniques before any analysis
**Storage**: Separated: raw (access-controlled, audited) vs aggregated (open/shared)
**Compute**: Run analytics at source (federated) or on anonymised extracts
**Publication**: Publish aggregate datasets and trained models as open public goods

---

## Choosing a Data Sharing Model

### Decision Framework

| Question | If YES → Consider |
|---------|-----------------|
| Is this a one-time credential presentation (qualification, registration)? | **Verifiable Credentials** |
| Does the citizen need to be in control of each data access? | **Consent-based (DEPA)** |
| Is the data needed in real-time for a service decision? | **System-to-system API** or **Consent-based** |
| Is this ongoing access over time (monthly bank statements)? | **Consent-based (DEPA)** with frequency limits |
| Is this cross-border with no bilateral agreements? | **Verifiable Credentials** |
| Is this aggregated, non-personal research/AI data? | **Anonymised open datasets** |
| Is government verifying citizen-claimed data automatically? | **System-to-system API** |
| Does the citizen need offline verification capability? | **Verifiable Credentials** (offline-capable) |

### Combination Patterns

In mature DPI ecosystems, all models co-exist and complement each other:

```
Citizen's "data portfolio":

Verifiable Credentials    Consent-based flows      Open APIs
─────────────────────    ──────────────────────   ──────────
Degree certificate       Bank transactions         Tax record
Health insurance card    Health records (ABDM)     Vehicle reg.
ID credential            Salary slips              Property record
Business registration    GST returns               Address

Each model serves different data types, contexts, and use cases.
```

---

## Cross-Border Data Sharing

**CDPI observation**: Cross-border flows are simplest with user-led methods (Verifiable Credentials) because they **do not require bilateral or multilateral agreements** between issuing parties and international verifiers — any verifying entity can locally verify the digital signature.

For system-to-system models, G2P Connect specifications enable interoperability at technology and domain layers based on OAuth2, OpenID Connect, and W3C VC standards. Bilateral or regional agreements are needed for each data-sharing relationship.

**Implication for DPI programs**: Start cross-border data sharing with VC-based credentials; add consent-based models only once mutual legal frameworks exist.

---

## Key References

- **CDPI Data Sharing Primer**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/a-primer-to-personal-data-sharing
- **CDPI Verifiable Credentials**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/verifiable-credentials
- **CDPI VC Decision Framework**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/decision-framework-for-verifiable-credentials
- **CDPI Non-Personal Data**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/non-personal-anonymised-datasets
- **CDPI Data Standards**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/data-standards
- **DEPA Framework**: https://www.niti.gov.in/sites/default/files/2023-03/Data-Empowerment-and-Protection-Architecture.pdf
- **G2P Connect Specs**: https://g2pconnect.cdpi.dev
- **HL7 FHIR R4**: https://hl7.org/fhir/R4/
- **ISO 20022**: https://www.iso20022.org/
