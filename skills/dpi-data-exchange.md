---
name: dpi-data-exchange
description: Data Exchange as DPI — consent frameworks, DEPA, real-time vs async patterns, FHIR, open banking. Use when designing data exchange architecture or evaluating data portability approaches.
---

# Data Exchange as DPI

## The Third Layer: Data Exchange

Data exchange is the third foundational layer of DPI, enabling consented, auditable, and purpose-limited sharing of personal and institutional data across organizations.

**The core problem it solves**: Data about citizens is siloed across government agencies, banks, hospitals, and utilities. Individuals cannot easily access, share, or leverage their own data. Organizations cannot efficiently verify information about individuals. Data exchange DPI creates the plumbing for consent-based data flows.

## Data Empowerment and Protection Architecture (DEPA)

DEPA is India's framework for consent-based data sharing, developed by iSPIRT:

### Core Components

```
Data Principal (Person)
        ↕ (consent)
Consent Manager (CM) ←→ Consent Artifact
        ↕                        ↕
Data Provider (DP)         Data Consumer (DC)
(Bank, Hospital,         (Lender, Insurer,
 Government)              Government)
```

### DEPA Principles
1. **Consent as the key**: No data moves without explicit, granular consent
2. **Individual as data principal**: The person owns and controls their data
3. **Purpose limitation**: Data can only be used for consented purpose
4. **Minimization**: Only required data is shared
5. **Audit trail**: Every consent grant and data fetch is logged
6. **Revocability**: Consent can be withdrawn at any time

### Consent Artifact Structure
```json
{
  "consentId": "uuid",
  "timestamp": "ISO8601",
  "dataConsumer": { "id": "entity-id", "name": "Lender XYZ" },
  "dataProvider": { "id": "bank-id", "name": "State Bank" },
  "purpose": { "code": "101", "text": "Loan underwriting" },
  "fiTypes": ["DEPOSIT", "TERM_DEPOSIT"],
  "dateTimeRange": { "from": "2023-01-01", "to": "2024-01-01" },
  "frequency": { "unit": "MONTH", "value": 1, "count": 12 },
  "signature": "digital-signature"
}
```

## Consent-Led Data Sharing — DEPA and Account Aggregator

DEPA (Data Empowerment and Protection Architecture) is the open framework for consent-led real-time data sharing. In DEPA, a **Consent Manager** is a data-blind intermediary — it manages consent but never sees the underlying data. Countries can implement consent-led data sharing under any governance model.

India's **Account Aggregator (AA)** framework is the most mature implementation of DEPA, adapted to India's financial sector regulatory context (RBI-licensed NBFCs act as AAs). The Consent Manager role in DEPA maps to the Account Aggregator in this implementation.

### Participants
- **Account Aggregator (AA)**: Licensed NBFC-AA; manages consent; does NOT see the data
- **Financial Information Provider (FIP)**: Banks, insurers, mutual funds (share data)
- **Financial Information User (FIU)**: Lenders, wealth managers, fintechs (consume data)

### AA Flow
```
1. FIU requests data → sends consent request to AA
2. AA presents consent to user (mobile app)
3. User reviews & approves consent
4. AA sends signed consent artifact to FIP
5. FIP verifies consent, packages data
6. Data flows encrypted directly FIP → FIU (AA not in data path)
7. FIU decrypts and uses data
```

### Technical Stack
- **Consent API**: REST/JSON between all parties
- **FI Data API**: Standardized financial data schemas (RBI defined)
- **Encryption**: Diffie-Hellman key exchange per session; end-to-end encrypted
- **Discovery API**: Locating accounts across FIPs from mobile number/PAN
- **Notification API**: Webhooks for status updates

### AA Participants (India, 2024)
- AAs: Saafe, OneMoney, Finvu, CAMS Finserv, Perfios AA, NADL
- FIPs: 130+ including SBI, HDFC, ICICI, all major banks, mutual funds, insurance
- FIUs: PhonePe, BankBazaar, Perfios, Yodlee, and hundreds of lenders

## Health Data Exchange

### Ayushman Bharat Digital Mission (ABDM)
India's health data exchange built on DEPA principles:

**Health ID (ABHA)**: 14-digit health account number linked to health records
- Links records across hospitals, labs, pharmacies
- Consent-based sharing via Health Locker

**Health Information Exchange (HIE)**:
- **FHIR R4** as the data standard
- Health Facility Registry, Healthcare Professional Registry
- PHR App (Personal Health Records application)

### Global Health Data Standards
- **HL7 FHIR R4**: Resource-based health data model; RESTful APIs
- **IHE Profiles**: Integration profiles for specific health workflows
- **OpenHIE**: Reference architecture for health information exchanges
- **SMART on FHIR**: App authorization for EHR data

## Government Data Exchange

### G2G (Government-to-Government) Data Sharing
- **DigiLocker (India)**: Document sharing from government issuers to citizens to verifiers
- **X-Road (Estonia/Nordics)**: Federated secure data exchange between government agencies
- **Government Integration Platforms**: API gateways between ministries

### X-Road Architecture
```
Service Client → Security Server (client org)
                        ↕ (TLS, signed messages)
               Central Server (config)
                        ↕
              Security Server (provider org) → Service Provider
```

### Principles for G2G Exchange
- Once-only principle: Citizens should not submit data the government already has
- Digital-by-default: All government services accessible digitally
- Trustworthy: Authenticated, encrypted, non-repudiable
- Reusable infrastructure: One integration platform vs. bilateral integrations

## API-Based Data Portability

### Open Banking / Open Finance
- **PSD2 (EU)**: Mandated bank API access for third parties
- **UK Open Banking**: Implementation of PSD2 with standardized APIs
- **RBI Account Aggregator**: Indian equivalent with consent framework
- **Australia CDR**: Consumer Data Right — banking, energy, telecoms
- **Brazil Open Finance**: OpenID Connect + FAPI-based

### Common API Patterns
```
/accounts              → List accounts
/accounts/{id}         → Account details
/accounts/{id}/transactions → Transaction history
/consents              → Manage consent grants
/tokens                → OAuth 2.0 token endpoints
```

### Financial API (FAPI) Security Profile
- Built on OAuth 2.0 + OIDC
- Requires mTLS or PKCE
- Signed request objects (JAR)
- Sender-constrained access tokens
- Used by: UK Open Banking, Brazil Open Finance, Australia CDR

## Data Localization vs. Data Portability Tension

| Concern | Implication |
|---------|-------------|
| **Sovereignty** | Data about citizens stored in-country |
| **Portability** | Citizen can take data across borders |
| **Privacy** | Minimize data movement |
| **Innovation** | Data flows fuel services |

DPI approach: data stays at source (FIP/provider), only moves on explicit consent; localization at rest, portability in motion.

## Key Design Decisions for Data Exchange DPI

### 1. Centralized vs. Federated Architecture
- **Centralized data lake**: Government holds all data; high risk, anti-DPI
- **Federated with hub-and-spoke**: Central registry/routing, data stays at source
- **Fully decentralized**: Peer-to-peer with consent artifacts; complex discovery

### 2. Consent Models
- **Push consent**: Provider pushes data when consent received
- **Pull consent**: Consumer pulls data with consent token
- **Streaming consent**: Ongoing data flow (account aggregation)
- **One-time consent**: Single fetch (document verification)

### 3. Data Standards
- Always use published standards (FHIR, ISO 20022, IOFFA)
- Define once, reuse across all participants
- Version management and backward compatibility

### 4. Trust and Liability
- Regulated entities only (licensed FIPs/FIUs/AAs)
- Insurance / indemnity for data breaches
- Audit logs retained (typically 5-7 years)
- Grievance redressal mechanisms

## Implementation Stack

```
Layer               | Technology Options
--------------------|----------------------------------
Identity & Auth     | OIDC/OAuth 2.0, FAPI, e-Signet
Consent Management  | AA framework, DEPA consent manager
API Gateway         | Kong, APIGEE, AWS API GW, custom
Data Standards      | FHIR R4, ISO 20022, custom JSON
Encryption          | ECDH key exchange, AES-256-GCM
Audit Logging       | Immutable logs, blockchain anchoring
Discovery           | Central registry + federation
Notification        | Webhooks, Server-Sent Events
```

## Real-Time vs. Asynchronous Data Exchange in DPI

Choosing between synchronous and asynchronous data exchange is one of the most consequential architectural decisions in DPI design. The wrong choice creates either poor user experience (unnecessary waiting) or unreliable integrations (dropped events, missed updates).

### Synchronous (Real-Time) Data Exchange

**What it is**: The requesting system waits for the response before proceeding. The entire interaction completes in a single request-response cycle.

**When to use:**
- Authentication decisions (yes/no within milliseconds)
- eKYC at point of service (user waiting at bank counter)
- Payment authorization (merchant waiting for confirmation)
- Document verification at border crossing
- Real-time eligibility checks

**Technical patterns:**
```
Client → REST API (HTTPS/GET/POST) → Provider → Response (200 OK + data)
                                    ↑
                         Timeout: typically 3-30 seconds
                         Retry: exponential backoff on failure
                         Circuit breaker: fail fast if provider is down
```

**Design requirements for real-time DPI:**
- P99 latency SLAs (typically <300ms for auth, <1s for eKYC)
- Synchronous error codes with actionable information
- Graceful degradation if provider is unavailable
- Rate limiting on the consumer side to prevent DDoS

**Failure modes:**
- Timeout: Provider took too long → must return error, not silent failure
- Partial response: Provider returned error mid-response → idempotency design
- Network partition: Temporary unreachability → retry with exponential backoff

### Asynchronous Data Exchange

**What it is**: The requesting system sends a request and receives a confirmation; the actual data arrives later via callback, webhook, or polling.

**When to use:**
- Document generation (certificate issuance takes 5-30 seconds)
- Enrollment processing (biometric deduplication takes minutes)
- Batch payment processing (bulk G2P disbursement)
- Health record aggregation (pulling from 10 providers takes variable time)
- Long-running consent-based data fetch (Account Aggregator model)
- Audit log delivery (best-effort, not in critical path)

**Technical patterns:**

**Pattern 1: Webhook / Callback**
```
Client → POST /request { callbackUrl: "https://client/callback" }
       ← 202 Accepted { requestId: "uuid" }

[Processing happens asynchronously]

Provider → POST https://client/callback
          { requestId: "uuid", status: "complete", data: {...} }
```

**Pattern 2: Polling**
```
Client → POST /request
       ← 202 Accepted { requestId: "uuid", pollUrl: "/status/uuid" }

Client → GET /status/uuid  (every N seconds)
       ← 202 Processing   (while pending)
       ← 200 Complete { data: {...} }  (when done)
```

**Pattern 3: Event-Driven (Pub/Sub)**
```
Provider publishes events to → Topic (Kafka/EventBridge/NATS)
                                    ↓
                         Consumer subscribes and processes
                         (decoupled; consumer controls processing pace)
```

**Pattern 4: Server-Sent Events (SSE) / WebSockets**
```
Client → GET /events (persistent connection)
Provider → data: { event: "consent_approved", ... }
Provider → data: { event: "data_ready", ... }
(Real-time push without polling, good for UI status updates)
```

### Decision Matrix: Sync vs Async

| Scenario | Sync or Async? | Rationale |
|---------|---------------|-----------|
| Authentication / login | **Sync** | User is waiting; <500ms expected |
| eKYC at onboarding | **Sync** | User is present; 1-3s acceptable |
| Enrollment biometric processing | **Async** | Takes 5-30 min; don't hold connection |
| Payment authorization | **Sync** | Merchant is waiting for confirmation |
| Bulk G2P disbursement | **Async** | Millions of records; process in batches |
| Document issuance | **Async** (+ webhook) | Takes variable time; notify when ready |
| Health record aggregation | **Async** (+ webhook) | Pulls from multiple systems; variable |
| Account statement fetch (AA) | **Async** | FIP may take seconds to minutes |
| Fraud detection at payment | **Sync** | Must respond before payment is approved |
| Audit log delivery | **Async** (best-effort) | Not in critical path; batch OK |
| Credential revocation | **Async** (+ periodic refresh) | Not instant; status list cache model |
| VC presentation (verifiable credential) | **Async** | Holder-led; issuer not contacted at verification time |

### Hybrid Patterns in DPI

Most production DPI systems use hybrid patterns:

**Account Aggregator (Hybrid)**:
```
1. Consent grant: Sync (user waits for confirmation)
2. Consent artifact delivery to FIP: Async (webhook)
3. Data packaging at FIP: Async (takes variable time)
4. Data delivery to FIU: Async (webhook when ready)
5. FIU processing: Async
Total: Minutes, but user is not blocked after step 1
```

**Payment System (Hybrid)**:
```
1. Payment initiation: Sync (payer gets immediate transaction ID)
2. Routing and clearing: Near-sync (milliseconds in switch)
3. Final settlement: Async (RTGS at end of day)
4. Notification to payee: Async (push notification)
User experience is sync; settlement is async
```

### Reliability Patterns for Async DPI

**Idempotency**: Every async operation must have an idempotency key. If the same request is submitted twice (due to retry), only one operation executes.

```http
POST /consents
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
{ ... consent request ... }
```

**At-Least-Once Delivery**: Design consumers to handle duplicate events. Use idempotency keys to deduplicate.

**Dead Letter Queue (DLQ)**: Events that cannot be processed after N retries go to DLQ for manual review. Never silently drop events.

**Event Ordering**: Don't assume events arrive in order. Design state machines that handle out-of-order events gracefully.

**Outbox Pattern**: For reliable event publishing:
```
1. Write event to database "outbox" table (same transaction as business data)
2. Background worker reads outbox and publishes to event bus
3. Guarantee: event is published if and only if the business transaction committed
```

## The Data Exchange Rail Principle

Like identity and payments, data exchange should be a shared rail:
- Consent and audit infrastructure is shared (every sector uses same consent mechanism)
- Data schemas are standardized and published (not per-provider)
- Any authorized consumer can access any authorized provider through the same rails
- Competitive advantage is in how you use data, not in owning exclusive data pipelines

**Anti-pattern**: Each government agency builds its own data sharing API, its own consent form, its own audit log → N silos, no portability, no reuse.

**DPI pattern**: One consent framework, multiple data providers and consumers all using the same consent management infrastructure.

## Standards & References

- **DEPA**: Data Empowerment and Protection Architecture (open framework)
- **FAPI 2.0**: Financial-grade API security profile (OAuth 2.0 extension)
- **HL7 FHIR R4**: Fast Healthcare Interoperability Resources
- **OpenID Connect**: Authentication/authorization protocol
- **ISO/IEC 29101**: Privacy architecture framework
- **X-Road**: Federated data exchange layer (open source, Estonia/Nordic) — see dpi-x-road skill
- **AsyncAPI**: Specification for event-driven / async APIs (like OpenAPI for async)
- **CloudEvents**: Standard for event data format (CNCF)
- **NATS / Apache Kafka**: Open-source message buses for async DPI patterns
