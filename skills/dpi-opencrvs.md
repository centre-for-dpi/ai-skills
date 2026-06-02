---
name: dpi-opencrvs
description: OpenCRVS — open-source CRVS DPG, FHIR data model, birth/death/marriage registration, national ID integration. Use when designing civil registration systems or evaluating OpenCRVS.
---

# OpenCRVS — Civil Registration as DPI

> **Reference**: https://documentation.opencrvs.org  
> **GitHub**: https://github.com/opencrvs/opencrvs-core  
> **Website**: https://www.opencrvs.org

## What is OpenCRVS?

OpenCRVS is an open-source **Civil Registration and Vital Statistics (CRVS)** platform — a Digital Public Good designed to help governments achieve universal civil registration and evidence-based decision making in all country contexts.

Its mission: **every individual receives legal recognition from birth**. Birth registration is the foundation of legal identity and the prerequisite for accessing education, healthcare, social protection, and voting rights.

**Why civil registration is DPI**: CRVS is the upstream DPI layer that feeds all other DPI rails. Without birth registration:
- Children cannot get a national ID
- Deceased individuals remain in voter rolls and beneficiary registers
- Population statistics are unreliable for planning health, education, and welfare

**DPG Status**: Registered Digital Public Good. Licensed under **Mozilla Public License 2.0**.

**Stewards**: Jembi Health Systems; co-developed with Plan International and UNICEF.

---

## Architecture

OpenCRVS is built as modular, **event-driven microservices** written in TypeScript and Node.js using the HapiJS framework. Each microservice has no knowledge of other services — all inter-service communication is via events and APIs.

```
┌────────────────────────────────────────────────────────────────────┐
│                          OpenCRVS Core                             │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │              Client Applications (React/TypeScript)          │  │
│  │   Register App  │  Login  │  Performance Dashboard          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │          GraphQL Gateway (Apollo Server, JWT-secured)         │  │
│  └──────────────────────────────────────────────────────────────┘  │
│         │              │             │              │               │
│  ┌──────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐   │
│  │ auth │ │user-mgmt │ │ workflow │ │  search  │ │  metrics  │   │
│  └──────┘ └──────────┘ └──────────┘ └──────────┘ └───────────┘   │
│                          │                │              │          │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  Hearth (FHIR R4 datastore)  │  ElasticSearch  │  InfluxDB   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │   countryconfig microservice (forms, settings, certificates) │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

### Core Microservices

| Service | Purpose | Tech |
|---------|---------|------|
| **gateway** | GraphQL/Apollo API gateway; resolves FHIR resources to GraphQL | Apollo, HapiJS |
| **auth** | JWT token generation and management | HapiJS, Redis |
| **user-mgmt** | User roles, permissions, ISO27001-aligned access control | HapiJS, MongoDB |
| **workflow** | Business process orchestration; registration status transitions | HapiJS, event-driven |
| **search** | De-duplication, record lookup, search across registrations | ElasticSearch |
| **metrics** | Civil registration analytics and performance dashboards | InfluxDB (time-series) |
| **notification** | SMS and email communications via 3rd party gateways | HapiJS |
| **config** | Runtime configuration GUI for forms, certificates, settings | HapiJS |
| **countryconfig** | Country-specific forms, event declarations, reference data | Node.js (fork per country) |

### Data Store Layer

| Store | Purpose |
|-------|---------|
| **Hearth** | Open-source FHIR R4 NoSQL datastore (MongoDB-backed); primary record store |
| **ElasticSearch** | Real-time de-duplication and search; audit log |
| **InfluxDB** | Time-series database for analytics and performance metrics |
| **Redis** | JWT session management and caching |

---

## FHIR Data Model

OpenCRVS extends HL7 FHIR R4 for civil registration. Every vital event record is a **FHIR Bundle** containing a Composition and related resources.

**FHIR resource mapping:**

| FHIR Resource | CRVS Meaning |
|---------------|-------------|
| **Patient** | Person (child, parent, deceased) |
| **Practitioner** | Civil registration staff / registrar |
| **Encounter** | Interaction with a registrar |
| **Observation** | Essential data points (birth weight, cause of death) |
| **Task** | Registration workflow state (DECLARED → REGISTERED → CERTIFIED) |
| **Composition** | The vital event document (birth, death, marriage) |
| **Bundle** | Package of all resources for a vital event |
| **RelatedPerson** | Informant (parent, next of kin) |
| **Location** | Civil registration office, health facility |

### Registration Workflow States

```
NOTIFICATION (hospital notifies birth/death)
        ↓
DECLARATION (informant formally declares event)
        ↓
VALIDATION (office staff validates declaration)
        ↓
REGISTRATION (registrar registers event; legal record created)
        ↓
CERTIFICATION (birth/death certificate issued)
        ↓
(optionally) CORRECTION (amendments to registered record)
```

Each state transition updates the FHIR Task resource status and triggers notifications to relevant parties.

### FHIR Document Submission

Two methods to submit a vital event:
1. **MHD profile** (Mobile access to Health Documents): standard IHE profile for FHIR document exchange
2. **Direct FHIR API**: POST a FHIR Bundle directly to the FHIR endpoint

```http
POST /fhir
Content-Type: application/fhir+json
Authorization: Bearer <jwt>

{
  "resourceType": "Bundle",
  "type": "document",
  "entry": [
    { "resource": { "resourceType": "Composition", "type": { "coding": [{ "code": "birth-declaration" }] }, ... }},
    { "resource": { "resourceType": "Patient", ... }},        // child
    { "resource": { "resourceType": "RelatedPerson", ... }},  // mother
    { "resource": { "resourceType": "RelatedPerson", ... }},  // father
    { "resource": { "resourceType": "Task", "status": "draft", ... }}
  ]
}
```

---

## Core Features

### Offline-First Registration
- Field agents can capture registrations in offline/low-connectivity areas
- Data syncs to central server when connectivity is restored
- Supports rural and remote registration posts

### Multi-Event Registration
- **Birth registration**: child, parents/guardians, location, attendant at birth
- **Death registration**: deceased, cause of death, informant, location
- **Marriage registration**: spouses, witnesses, officiant

### Role-Based Access Control
- Configurable roles: Field Agent, Registration Agent, Registrar, Local System Admin, National System Admin
- Each role has defined permissions on which events they can declare, validate, register, certify
- ISO 27001-aligned access management

### Certificate Generation
- PDF certificate generation from configurable SVG templates
- Country-specific certificate designs
- QR code on certificates for online verification

### Performance Dashboard
- Real-time registration statistics
- Time to registration metrics
- Geographic breakdowns by district/province
- Completeness rates vs. estimated population

---

## Country Configuration

OpenCRVS is designed to be configured, not forked at the core. Each country **forks** the `opencrvs-countryconfig` repository (reference implementation: **Farajaland** — a fictional country used as the canonical example).

```
opencrvs-core          ← the platform (update as new versions release)
opencrvs-countryconfig ← your country's configuration (fork this)
```

### What the Country Config Contains

```
src/
  api/         ← countryconfig microservice endpoints
  form/        ← registration form definitions (JSON)
  certificates/ ← certificate SVG templates
  features/    ← country-specific business logic
  data/        ← reference data (admin divisions, offices, facilities)
```

**Configuration dimensions:**
- Administrative structure (provinces, districts, offices)
- Registration form fields (add/remove/translate fields)
- Certificate templates
- SMS gateway integration
- Identity verification rules
- Notification templates

### Deployment

OpenCRVS uses **Docker Compose** for development and single-node deployments, and **Kubernetes/Helm** charts for production multi-node deployments.

```yaml
# docker-compose.yml (development)
services:
  hearth: ...          # FHIR datastore
  elasticsearch: ...   # search
  influxdb: ...        # metrics
  redis: ...           # sessions
  gateway: ...         # GraphQL API
  auth: ...
  user-mgmt: ...
  workflow: ...
  search: ...
  metrics: ...
  notification: ...
  config: ...
  countryconfig: ...   # your fork
  client: ...          # React app
```

**Production**: Helm charts in `charts/` directory; designed for private or public cloud; each microservice independently scalable.

---

## Interoperability

OpenCRVS exposes and consumes integration APIs for connecting with the broader DPI ecosystem.

### Outbound Integrations (CRVS → other DPI)

```
Birth registered
    ↓ webhook / API call
National ID System (e.g., MOSIP) ← assign national ID at birth
    ↓
Health system (e.g., DHIS2) ← vital statistics reporting
    ↓
Social Protection Registry ← notify of new eligible child
```

### Integration Patterns

| Integration | Pattern | Standard |
|-------------|---------|---------|
| National ID enrollment | Webhook → ID system API | REST/JSON |
| Health facility notifications | OpenHIM event bus | HL7 FHIR R4 |
| DHIS2 vital statistics | Periodic aggregate push | DHIS2 API |
| Death → voter roll update | Webhook → electoral system | REST |
| Death → social registry | Webhook → beneficiary system | G2P Connect |
| Death → financial system | Webhook → bank/payment system | REST |

### OpenHIM Integration
OpenCRVS uses **OpenHIM** (Open Health Information Mediator) as an enterprise service bus, enabling mediators that translate between OpenCRVS FHIR events and external system APIs. OpenHIM provides:
- Audit logging of all interoperability transactions
- Transaction retry and error handling
- Message transformation mediators

---

## Deduplication

Preventing duplicate registrations is critical for CRVS integrity.

**OpenCRVS deduplication approach:**
1. **ElasticSearch fuzzy matching**: On declaration, a fuzzy search checks for potential duplicates by name, date of birth, parents' names, and location
2. **Potential duplicate flagging**: Registrars are alerted to review potential duplicates before registration
3. **Manual review workflow**: Registrars compare and resolve potential duplicates before issuing a certificate
4. **Not biometric**: OpenCRVS relies on demographic deduplication, not biometrics — appropriate for a legal process where registrars are accountable

---

## Country Deployments

OpenCRVS has been deployed and piloted in multiple countries, including:

| Country | Context |
|---------|---------|
| **Bangladesh** | National CRVS modernisation |
| **Zambia** | District-level pilots |
| **Mozambique** | Civil registration reform |
| **Rwanda** | CMU partnership (opencrvs-cmu) |
| **Ethiopia** | CRVS digitisation |

Farajaland (fictional) serves as the reference configuration for all new country implementations.

---

## Governance

| Entity | Role |
|--------|------|
| **Jembi Health Systems** | Core technical steward; primary maintainer |
| **Plan International** | Co-initiator; field deployment experience |
| **UNICEF** | DPG co-developer; funding and policy |
| **What Works** | Civil registration knowledge hub |
| **UNECA / UNESCAP** | Regional support and advocacy |

**License**: Mozilla Public License 2.0 + Civil Registration & Healthcare Disclaimer (opencrvs.org/license).

**Current version**: v1.9.x (active development; 62+ releases, 23,000+ commits).

---

## Relationship to Other DPI Skills

| DPI Layer | Link to CRVS |
|-----------|-------------|
| **dpi-digital-id** | Birth registration → triggers national ID enrollment; death → ID deactivation |
| **dpi-registries** | OpenCRVS is the canonical CRVS implementation; feeds functional registries |
| **dpi-payments** | Death registration → removes from G2P beneficiary list; prevents ghost payments |
| **dpi-health** | Shares health facility registry; birth notifications from health facilities |
| **dpi-childhood** | CRVS is the first DPI touchpoint for every child; birth certificate = access to all services |

---

## Key References

- **Documentation**: https://documentation.opencrvs.org
- **GitHub (core)**: https://github.com/opencrvs/opencrvs-core
- **Country config template**: https://github.com/opencrvs/opencrvs-countryconfig
- **Farajaland reference**: https://github.com/opencrvs/opencrvs-farajaland
- **FHIR Documents**: https://documentation.opencrvs.org/technology/standards/fhir-documents
- **IDB profile**: https://knowledge.iadb.org/knowledge-resources/code-development/open-source-solution/opencrvs
- **OpenHIM**: https://openhim.org
