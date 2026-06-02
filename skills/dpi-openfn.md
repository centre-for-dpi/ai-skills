---
name: dpi-openfn
description: OpenFn — open-source DPG workflow automation for DPI data exchange (DHIS2, OpenMRS, registries). Use when designing interoperability workflows between DPI systems or government data pipelines.
---

# OpenFn — DPI Workflow Automation Platform

## What is OpenFn?

OpenFn is an **open-source Digital Public Good** and **DPI building block** designed specifically for governments, NGOs, and international organizations to automate data exchange between the systems that make up a DPI ecosystem.

> **Official description**: "Lightning is a workflow automation platform that's used to automate critical business processes and integrate information systems, boosting efficiency & effectiveness while enabling secure, stable, scalable interoperability and data integration."

OpenFn is listed as a **Digital Public Good** (DPGA registry) and positions itself explicitly as DPI infrastructure for data exchange — not a general-purpose tool.

**Key links:**
- **Platform**: https://app.openfn.org
- **Documentation**: https://docs.openfn.org
- **GitHub**: https://github.com/OpenFn
- **Community**: https://community.openfn.org
- **License**: GNU AGPLv3 (open source, copyleft)

## Why OpenFn for DPI?

OpenFn solves a specific problem in DPI deployments: **DPI building blocks (DHIS2, OpenMRS, MOSIP, payment rails, registries) need to exchange data reliably, auditably, and at scale — without building custom integration code for every pair of systems.**

Instead of bilateral, point-to-point integrations (N² problem), OpenFn provides a common integration layer:

```
Before OpenFn (point-to-point):
DHIS2 ←→ OpenMRS
DHIS2 ←→ Registry
DHIS2 ←→ Payment Rail
OpenMRS ←→ Registry
...N² integrations

After OpenFn (hub model):
DHIS2      OpenMRS     Registry     Payment Rail
  ↓            ↓           ↓             ↓
         OpenFn Workflow Layer
              ↓
         Audit Trail + Monitoring
```

## Core Concepts

### Jobs

A **job** is the atomic unit of work in OpenFn — a discrete task at a given point in a workflow:

```javascript
// Example OpenFn job: Fetch new births from CRVS → enroll in health system
fn(state => {
  // Get new registrations from civil registry (CRVS)
  return state;
});

get('/api/births', {
  params: { 
    registrationDate: state.data.lastRunDate,
    status: 'new'
  }
});

fn(state => {
  // Transform CRVS birth record to DHIS2 enrollment format
  state.enrollments = state.data.births.map(birth => ({
    trackedEntityType: 'CHILD',
    orgUnit: birth.facilityCode,
    enrollmentDate: birth.registrationDate,
    attributes: [
      { attribute: 'NationalID', value: birth.nationalId },
      { attribute: 'DateOfBirth', value: birth.dob }
    ]
  }));
  return state;
});

// Post enrollments to DHIS2 tracker
post('/api/trackedEntityInstances', { data: state.enrollments });
```

### Triggers

OpenFn supports three trigger types:

| Trigger | Description | DPI Use Case |
|---------|-------------|-------------|
| **Cron** | Scheduled execution (daily sync, hourly batch) | Nightly reconciliation, weekly reports |
| **Webhook** | Triggered by inbound HTTP POST | Real-time: new enrollment triggers health record creation |
| **Kafka** | Consumes Kafka topic messages | Event-driven: payment event triggers benefit status update |

### Workflows (Lightning v2)

In OpenFn Lightning (v2, current), workflows chain multiple jobs using conditional logic:

```
Workflow: "New Birth → National ID → Health Record → Notification"

[Webhook Trigger: CRVS new birth]
         ↓
[Job 1: Validate birth record]
         ↓
    ┌────┴─────┐
   OK        FAIL
    ↓           ↓
[Job 2: Request  [Job: Log error
 National ID]     → Notify admin]
    ↓
[Job 3: Create DHIS2
 enrollment]
    ↓
[Job 4: Send SMS
 to mother]
```

### Adaptors (Connectors)

Adaptors are pre-built, maintained JavaScript connectors for common DPI building blocks. OpenFn has **70+ adaptors** covering:

**Health DPGs:**
- `@openfn/language-dhis2` — DHIS2 API (aggregate, tracker, events)
- `@openfn/language-openmrs` — OpenMRS patient records
- `@openfn/language-openimis` — OpenIMIS health insurance

**Social Services & Case Management:**
- `@openfn/language-commcare` — CommCare case management
- `@openfn/language-primero` — Primero child protection
- `@openfn/language-kobotoolbox` — KoboToolbox data collection

**Data & Infrastructure:**
- `@openfn/language-postgresql` — PostgreSQL (direct DB)
- `@openfn/language-http` — Generic REST API
- `@openfn/language-sftp` — File-based integration
- `@openfn/language-googlesheets` — Google Sheets reporting

**Payments & Financial:**
- `@openfn/language-mpesa` — M-Pesa mobile money
- `@openfn/language-beyonic` — Beyonic payments

**Building custom adaptors:**
```javascript
// Minimal custom adaptor structure
import { execute as commonExecute } from '@openfn/language-common';

export const execute = (...operations) => {
  const initialState = { references: [], data: null };
  return state => {
    return commonExecute(...operations)(state);
  };
};

export function getRecord(path, params) {
  return state => {
    const { host, apiKey } = state.configuration;
    return fetch(`${host}${path}`, { headers: { Authorization: apiKey } })
      .then(r => r.json())
      .then(data => ({ ...state, data }));
  };
}
```

## Architecture

### OpenFn Lightning (v2 — Current)

```
┌─────────────────────────────────────────────────────────┐
│                  OpenFn Lightning                        │
│                                                         │
│  ┌──────────────┐  ┌───────────────┐  ┌─────────────┐  │
│  │  Web Editor  │  │  Workflow     │  │  Monitoring │  │
│  │  (Visual +   │  │  Engine       │  │  Dashboard  │  │
│  │   Code)      │  │  (Elixir)     │  │             │  │
│  └──────────────┘  └───────────────┘  └─────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Job Runner (Node.js)                 │   │
│  │  Executes JavaScript job expressions              │   │
│  │  Loads adaptors, manages state                   │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │              PostgreSQL (state store)             │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
         ↕                    ↕
  External DPI APIs     Inbound Webhooks
  (DHIS2, MOSIP, etc.)  (from CRVS, payment rails)
```

**Technology stack:** Elixir + Phoenix framework (backend), Node.js (job execution), PostgreSQL (only external dependency)

### State Management

Each job execution receives an immutable **state** object and returns a new state:

```javascript
// State flows through the workflow
{
  configuration: { host: "https://dhis2.example.gov", apiKey: "..." },  // Credentials
  data: { births: [...] },           // Incoming data
  references: [{ id: "prev-run" }],  // Previous run references
  // ... custom fields added by jobs
}
```

### Audit Trail

Every workflow execution produces a full audit trail:
- Input data snapshot
- Output data snapshot
- Success/failure status per job step
- Timestamps for each operation
- Error messages with stack traces
- Data lineage: what came from where, what was sent where

This is critical for DPI compliance — auditors can inspect exactly what data was exchanged between systems.

## Deployment

### Cloud (SaaS) — app.openfn.org

```
Fastest path to production:
1. Register at app.openfn.org
2. Create a project
3. Add credentials (DHIS2 API key, etc.)
4. Build workflows in the visual editor
5. Deploy and monitor
```

Plans: Free tier available; paid from $50/month.

### Self-Hosted (Full Sovereignty)

OpenFn Lightning is fully self-hostable — critical for data sovereignty in DPI:

**Docker Compose (quickest):**
```yaml
version: '3.8'
services:
  lightning:
    image: openfn/lightning:latest
    ports:
      - "4000:4000"
    environment:
      DATABASE_URL: postgresql://postgres:password@postgres:5432/lightning
      SECRET_KEY_BASE: your-64-char-secret-key
      ENCRYPTION_KEY: your-32-char-encryption-key
    depends_on: [postgres]
    
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: lightning
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**Kubernetes (production):**
```bash
# OpenFn provides Kubernetes manifests
kubectl apply -f https://raw.githubusercontent.com/OpenFn/lightning/main/deployment/k8s/
```

### Portability
Export any project as `project.yaml` from cloud → import to self-hosted (or vice versa). No lock-in.

## DPI Use Cases (Real Deployments)

### Ghana: Birth Registration → National ID → Health Enrollment
```
CRVS (new birth registered)
        ↓ [Webhook trigger]
OpenFn Workflow:
  Job 1: Validate birth record
  Job 2: Call National ID API → generate ID
  Job 3: Send ID to DHIS2 → enroll in immunization tracker
  Job 4: Send SMS to mother with ID number
```

### Thailand/Cambodia: Health + Social Services Integration
UNICEF + Ministry of Public Health automate patient medical history exchange between national HIS (DHIS2) and social services case management (Primero) using OpenFn.

### Nigeria: Disease Surveillance (ALMANCH)
SwissTPH extended OpenFn with OpenHIM connector for nationwide disease surveillance data exchange across state-level health systems.

## OpenFn CLI (Local Development)

```bash
# Install OpenFn CLI
npm install -g @openfn/cli

# Run a job locally
openfn job.js -a dhis2 -s state.json

# Test with sample state
openfn job.js -a http --log info -s '{
  "configuration": { "baseUrl": "https://api.example.gov", "apiKey": "test" },
  "data": {}
}'
```

## OpenFn vs Other Workflow Tools

| Dimension | OpenFn | n8n | Zapier/Make |
|-----------|--------|-----|-------------|
| **DPI Focus** | Built for DPI/government | General purpose | General purpose |
| **DPG Status** | Yes (DPGA registered) | No | No |
| **Adaptors** | 70+ DPI-specific | 400+ general | 1000+ general |
| **Code approach** | JavaScript expressions | Visual + code | Visual/no-code |
| **AI integration** | Limited | Native LangChain agents | Limited |
| **License** | AGPLv3 (open source) | Fair-code | Proprietary |
| **Sovereignty** | Full self-host | Full self-host | Cloud-only (Zapier) |
| **Audit trail** | Built-in, DPI-grade | Execution logs | Basic |
| **Government use** | Primary use case | Supported | Limited |
| **Kafka triggers** | Yes | No (use webhook) | No |
| **Best for DPI** | G2G data exchange, DPG integration | AI workflows, general automation | Quick prototyping |

## Integration with Other DPI Skills

- **With MOSIP/e-Signet**: OpenFn can trigger on new identity enrollments (webhook from MOSIP) and cascade to health, education, or social service registrations
- **With DHIS2**: Primary use case — aggregate data collection, tracker enrollment, event reporting
- **With Payment Rails**: Trigger payment disbursement workflows when beneficiary eligibility conditions are met
- **With X-Road**: OpenFn can call X-Road security servers as HTTP endpoints; X-Road provides the trust layer while OpenFn handles workflow logic
- **With n8n**: Use OpenFn for DPI system integration (DPG adaptors); use n8n for AI-powered workflow steps — they can complement each other
