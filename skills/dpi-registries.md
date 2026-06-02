---
name: dpi-registries
description: DPI Registries — civil registration, functional registries, master data, Sunbird RC, OpenG2P. Use when designing registry systems, evaluating registry DPGs, or building population databases.
---

# DPI Registries

## What are Registries in DPI?

A registry is a canonical, authoritative, and structured record of entities — people, organizations, facilities, products, or events. Registries are foundational to DPI because they provide the shared source of truth that all services can reference.

**Registry as DPI infrastructure:**
- Prevents each service from maintaining its own siloed master data
- Enables government services to verify and build on the same facts
- Supports identity, service delivery, and data exchange
- Open APIs make registry data accessible to authorized parties

## Types of Registries

### 1. Civil Registration (Life Events)
Records of fundamental life events that establish legal identity:
- **Birth registration**: Creates the foundational record; triggers right to identity
- **Death registration**: Removes from active population, triggers benefits cessation
- **Marriage registration**: Establishes legal relationship; affects benefits, inheritance
- **Divorce registration**
- **Adoption registration**

**Key challenge**: Rural/remote civil registration is often incomplete. CRVS (Civil Registration and Vital Statistics) systems aim to achieve universal registration.

### 2. Population Registry
- Comprehensive record of all residents in a jurisdiction
- Links civil registration events to a persistent person record
- Foundation for national ID issuance (links to Aadhaar, MOSIP)
- Maintained by civil registration authority or population secretariat

### 3. Foundational Identity Registry
The database underlying national ID programs:
- In MOSIP: the ID Repository
- Stores demographic + biometric records
- Used by authentication, eKYC, and deduplication services
- Not to be confused with a civil registration system (though they may be linked)

### 4. Functional Registries
Purpose-specific registries for a particular sector or program:

| Registry | Purpose | Examples |
|----------|---------|---------|
| Voter Registry | Electoral rolls | Linked to population registry |
| Health Facility Registry | Locations and services | WHO mHero, OpenHIE |
| Healthcare Professional Registry | Licensed practitioners | Medical Council registries |
| Business Registry | Companies and entities | MCA21 (India), Companies House (UK) |
| Land Registry | Property ownership | Digital land records |
| Vehicle Registry | Motor vehicles | VAHAN (India) |
| Education Institution Registry | Schools, colleges | UDISE+ (India) |
| Beneficiary Registry | Social protection recipients | OpenG2P |
| Agricultural Registry | Farmers, land holdings | PM-KISAN (India) |

## Registry Design Principles

### 1. Single Source of Truth
- One authoritative registry per entity type
- All services point to the registry, not their own copy
- Updates propagate from registry to all consumers
- Deduplication enforced at registry level

### 2. Persistent, Stable Identifiers
- Entities receive a stable, unique identifier at registration
- Identifier never reused (even after death/deregistration)
- Identifier is meaningful but not sensitive (not SSN-style)
- Example: Farm ID linked to farmer's national ID

### 3. Minimal Required Fields
- Capture only what is needed for registry purpose
- Additional data linked via relationships, not stored in registry
- Reduces privacy risk and maintenance burden

### 4. Open APIs
- CRUD operations via REST API
- Bulk import/export for migration and batch processing
- Webhooks for event notifications (new registration, update)
- OAuth 2.0 access control with scoped permissions

### 5. Audit and History
- Every record change is logged with timestamp and actor
- Previous versions retained for audit
- Non-repudiable: cannot be modified without trace

## MOSIP Registry Architecture

MOSIP's identity registry (ID Repository):

```
┌───────────────────────────────────────────────────────┐
│                    ID Repository                       │
│                                                       │
│  ┌─────────────────┐    ┌──────────────────────────┐  │
│  │   Identity       │    │    Document Storage      │  │
│  │   Store          │    │   (encrypted biometrics, │  │
│  │   (PostgreSQL)   │    │    demographic docs)     │  │
│  └─────────────────┘    └──────────────────────────┘  │
│                                                       │
│  ┌─────────────────┐    ┌──────────────────────────┐  │
│  │   VID Manager    │    │    Credential Store      │  │
│  │   (Virtual IDs)  │    │   (issued VCs, tokens)   │  │
│  └─────────────────┘    └──────────────────────────┘  │
└───────────────────────────────────────────────────────┘
           ↕                          ↕
  ID Authentication          Resident Services
  (Auth queries)             (Self-service updates)
```

## Civil Registration and Vital Statistics (CRVS)

### Why CRVS Matters for DPI
- Birth certificate → basis for national ID
- Death certificate → triggers removal from voter/beneficiary registries
- Without CRVS, population registry is incomplete and DPI fails at inclusion

### Completeness Challenges
- Sub-Saharan Africa: ~50% births unregistered
- South Asia: Better but rural gaps persist
- Causes: Distance to registration centers, cost, lack of awareness, language barriers

### Digital CRVS Architecture
```
Life Event Occurs
       ↓
  Notification (hospital, midwife, community)
       ↓
  Registration Officer verifies
       ↓
  CRVS System creates record (OpenCRVS or similar)
       ↓
  Certificate generated (digitally signed)
       ↓
  Population Registry updated
       ↓
  National ID System notified
       ↓
  Certificate stored in DigiLocker / Document Registry
```

### OpenCRVS
Leading open-source CRVS platform:
- **GitHub**: https://github.com/opencrvs/opencrvs-core
- **Architecture**: React frontend + Node.js microservices + Elasticsearch + InfluxDB
- **Deployments**: Bangladesh, Zimbabwe, Farajaland (demo)
- **Key features**: Mobile field registration, offline support, deduplication, analytics

## Health Registries

### Health Facility Registry (HFR)
Master list of all healthcare facilities:
```json
{
  "facilityId": "IN-MH-BOM-0012345",
  "name": "Mumbai Central Hospital",
  "type": "PUBLIC_HOSPITAL",
  "level": "TERTIARY",
  "location": {
    "state": "Maharashtra",
    "district": "Mumbai",
    "lat": 18.9388,
    "lon": 72.8354
  },
  "services": ["OPD", "IPD", "ICU", "LABORATORY", "PHARMACY"],
  "operatingHours": "24/7",
  "contactPhone": "+912222446688",
  "abhmApproved": true,
  "geoCoordinates": { "lat": 18.9388, "lon": 72.8354 }
}
```

### Healthcare Professionals Registry (HPR)
Maintains verified records of licensed professionals:
- Doctor registration number, specialization, council approval
- Nurse, pharmacist, paramedic credentials
- Linked to national ID for deduplication
- Enables automated eKYC for healthcare professionals

### Product Registry
- Drug/medicine registry (CDSCO, India)
- Medical device registry
- Enables e-prescriptions that link to verified products

## Beneficiary Registries

### Social Protection Registry
Core registry for welfare program delivery:

```
┌──────────────────────────────────────────────────────┐
│               Social Registry                        │
│                                                      │
│  Household → Members → Socioeconomic Profile         │
│                             ↓                        │
│  Eligibility Assessment     ↓                        │
│  ┌──────────────────────────┐                        │
│  │ Program Enrollment       │                        │
│  │ PM-KISAN, NFSA, MGNREGS  │                        │
│  └──────────────────────────┘                        │
│                             ↓                        │
│  Beneficiary Registry → G2P Payment Registry         │
└──────────────────────────────────────────────────────┘
```

### OpenG2P
Open-source platform for government-to-person programs:
- **GitHub**: https://github.com/OpenG2P
- Built on Odoo framework
- Beneficiary registry management
- Program management and eligibility
- G2P payment integration (UPI, mobile money)
- Used in: Mozambique, Ethiopia, Philippines

### Deduplication in Beneficiary Registries
Critical to prevent duplicate beneficiaries:
```
Enrollment → Identity Verification (e-Signet/IDA)
                    ↓
             Deduplication Check
             (against existing beneficiaries)
                    ↓
             ┌─────────────┐     ┌───────────────┐
             │ No duplicate│     │ Duplicate found│
             │             │     │               │
             ▼             │     ▼               │
      Enroll beneficiary   │  Flag for review    │
             │             └─────────────────────┘
             ▼
      Link to payment account
```

## Land and Property Registries

### Digital Land Records
- Property title as authoritative record
- Prevents land disputes, enables mortgage financing
- Linked to national ID of owner(s)
- India: DILRMP (Digital India Land Records Modernization Programme)
- Key data: Survey number, owner ID, area, encumbrances, title chain

### Registry Architecture for Land
```json
{
  "surveyNumber": "123/4A",
  "district": "Mumbai Suburban",
  "taluka": "Andheri",
  "village": "Chakala",
  "area": { "value": 0.5, "unit": "acres" },
  "owners": [
    {
      "nationalId": "XXXXXXXX1234",
      "ownershipType": "FREEHOLD",
      "sharePercentage": 100
    }
  ],
  "encumbrances": [],
  "lastTransactionDate": "2020-01-15",
  "digitalSignature": "..."
}
```

## Business / Entity Registries

### Company Registry
- Unique business identifier (CIN, EIN, etc.)
- Directors, shareholders, registered address
- Filing history, annual returns
- India: MCA21 (Ministry of Corporate Affairs)
- Global standard: LEI (Legal Entity Identifier)

### LEI (Legal Entity Identifier)
- Global standard for financial entity identification
- ISO 17442
- 20-character alphanumeric code
- Required for cross-border financial reporting (EMIR, MiFID II)
- GLEIF (Global Legal Entity Identifier Foundation) maintains global registry

## Registry API Standards

### REST API Pattern
```http
# Get entity by ID
GET /v1/registry/{entity-type}/{entity-id}
Authorization: Bearer <token>

# Search registry
GET /v1/registry/health-facilities?state=MH&type=PUBLIC_HOSPITAL&limit=20
Authorization: Bearer <token>

# Register new entity
POST /v1/registry/farmers
Content-Type: application/json
Authorization: Bearer <token>
{
  "nationalId": "XXXXXXXX1234",
  "farmLocation": { "lat": 18.52, "lon": 73.86 },
  "landHolding": 2.5,
  "cropType": "WHEAT"
}

# Update record
PATCH /v1/registry/farmers/{farmerId}

# Webhook for events
POST /webhooks/registry/farmers
{
  "event": "RECORD_CREATED",
  "entityId": "FARM-MH-2024-001234",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### Sunbird RC (Registry and Credentials)
Open-source registry infrastructure:
- **GitHub**: https://github.com/Sunbird-RC/sunbird-rc-core
- **Deployments**: India (ABDM, COWIN vaccine registry, education)
- **Features**: Schema-driven, verifiable credentials issuance, OpenAPI
- Backed any entity registry with VC issuance capability

```yaml
# Sunbird RC schema for Health Facility Registry
title: HealthFacility
properties:
  facilityId: { type: string, index: true }
  name: { type: string }
  type: { type: string, enum: [PUBLIC, PRIVATE, NGO] }
  ownerNationalId: { type: string }
  services: { type: array, items: { type: string } }
indexFields: [facilityId, ownerNationalId]
uniqueIndexFields: [facilityId]
roles:
  issuedTo: HFR_ADMIN
  admin: HFR_ADMIN
enableLogin: false
credentialTemplate:
  type: HealthFacilityCredential
```

## Data Quality and Deduplication

### Registry Data Quality Dimensions
| Dimension | Meaning | Measurement |
|-----------|---------|-------------|
| **Completeness** | All required fields present | % records with all mandatory fields |
| **Accuracy** | Data matches reality | Sample audit against ground truth |
| **Uniqueness** | No duplicate records | Deduplication rate |
| **Timeliness** | Data is current | Average age of records |
| **Consistency** | Same format across records | Schema validation pass rate |

### Deduplication Approaches
```python
# Deterministic deduplication (exact match on key fields)
def is_duplicate_deterministic(new_record, existing_records):
    for record in existing_records:
        if (record['national_id'] == new_record['national_id'] or
            (record['name'] == new_record['name'] and 
             record['dob'] == new_record['dob'] and
             record['father_name'] == new_record['father_name'])):
            return True
    return False

# Probabilistic deduplication (fuzzy matching)
from dedupe import Dedupe
# Handles: name variations, typos, transliterations
```

## Registry Governance

### Ownership and Stewardship
- Each registry has a designated custodian (ministry/agency)
- Clear update authority: who can register, update, delete
- Appeals/corrections mechanism for incorrect records
- Access control: who can read, who can write

### Federated vs. Centralized Registry
| Approach | Example | Pros | Cons |
|----------|---------|------|------|
| **Centralized** | Aadhaar, population registry | Simple, one source | Concentration risk |
| **Federated** | X-Road-based | Resilient | Complex query |
| **Distributed with linking** | ABDM (ABHA links records across hospitals) | Balance | Integration complexity |
