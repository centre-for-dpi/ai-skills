---
name: dpi-architect
description: DPI architecture patterns — API-first rails design, microservices, security, scalability, reference architectures. Use when designing or reviewing DPI system architecture.
---

# DPI Architecture

## The Architect's Mandate

DPI architects must simultaneously serve two masters:
1. **The public sector**: Infrastructure that is reliable, secure, and accessible to every citizen at population scale
2. **The private sector**: Open rails that allow any authorized party to build innovative services on top

This dual mandate shapes every architectural decision. A choice that serves government delivery but prevents private innovation has failed. A choice that enables private innovation but excludes marginalized populations has also failed.

**The architecture is the policy.** How you design access control, API rate limits, data minimization, and federation directly determines who can participate and on what terms.

## Core Architectural Principles

DPI systems must be built differently from traditional government IT. The following principles are non-negotiable:

### 1. API-First
Every function exposed as a well-documented, versioned API before any UI is built.
- RESTful or GraphQL for most use cases
- OpenAPI 3.x specifications for all APIs
- API versioning from day one (`/v1/`, `/v2/`)
- API gateway for all external access

### 2. Federated, Not Centralized
- No single database for all records
- Multiple institutions as participants, not one database owner
- Routing, discovery, and trust at the center; data at the edges
- Avoid creating surveillance infrastructure

### 3. Open Standards
- No proprietary protocols where open standards exist
- HTTP/REST, JSON, OAuth 2.0, OIDC, ISO 20022, FHIR, W3C VC
- Open-source software preferred for core components
- Published specifications enable ecosystem participation

### 4. Modular and Composable
- Each module independently deployable and replaceable
- Loose coupling via APIs (not shared databases)
- Services can be upgraded without system-wide downtime
- Countries can adopt specific modules without the full stack

### 5. Inclusive by Design
- Works on 2G/3G networks (graceful degradation)
- Offline modes where connectivity is unreliable
- Feature phone support (USSD, SMS) as well as smartphones
- Accessible to people with disabilities
- Multiple languages and scripts

### 6. Privacy and Security by Default
- Minimal data collection (data minimization)
- Encryption at rest and in transit everywhere
- Role-based access control at every layer
- Audit logging of all data access
- Consent before any data sharing

## DPI Reference Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     Citizen/Service Layer                     │
│           (Web, Mobile, USSD, IVR, Assisted Channels)        │
├──────────────────────────────────────────────────────────────┤
│                      API Gateway Layer                        │
│    (Auth, Rate Limiting, Routing, Versioning, Monitoring)    │
├────────────────┬───────────────┬─────────────────────────────┤
│  Identity      │   Payments    │     Data Exchange            │
│  ─────────     │   ────────    │     ─────────────            │
│  ID Repo       │   Switch      │     Consent Mgr              │
│  Auth Svc      │   Directory   │     Data Gateway             │
│  eKYC API      │   Settlement  │     Schema Registry          │
│  Deduplication │   FX/Fees     │     Audit Log                │
├────────────────┴───────────────┴─────────────────────────────┤
│                    Platform Services                          │
│   (Notification, Scheduler, Workflow, Document Store)        │
├──────────────────────────────────────────────────────────────┤
│                    Infrastructure Layer                       │
│     (Cloud/On-prem, Kubernetes, Databases, Message Bus)      │
└──────────────────────────────────────────────────────────────┘
```

## Identity Layer Architecture

### Identity Stack Options

The identity layer comprises modular components — each can be selected independently:

| Component | Purpose | Open-source options |
|-----------|---------|-------------------|
| Registration | Enrollment, biometric capture | MOSIP, OpenCRVS (civil reg), custom |
| ID Repository | Deduplication, ID store | MOSIP IDA, custom |
| Authentication | OIDC/FIDO2 auth | e-Signet, Keycloak + FIDO, custom |
| Credential Issuance | VC issuance to wallets | Inji Certify (MOSIP-native), CREDEBL (multi-tenant, AnonCreds), custom |
| Credential Wallet | Citizen-side storage | Inji Wallet, any OpenID4VCI-compliant wallet |
| Verification | Relying party integration | Inji Verify, CREDEBL Verify, custom |

> For credential issuance specifically: see **dpi-inji** (MOSIP-integrated) and **dpi-credebl** (multi-tenant, AnonCreds ZKP) for platform comparison.

### MOSIP Component Architecture (example implementation)
```
Pre-Registration → Registration Client → Registration Processor
                                               ↓
                                         ID Repository
                                        ↙            ↘
                               ID Authentication    Resident Services
                                     ↓
                              e-Signet (OIDC/OpenID4VP)
                                     ↓
                           Service Providers (Banks, etc.)
```

### Key Design Decisions: Identity

**Biometric storage strategy:**
- Option A: Store biometrics centrally → strong deduplication, surveillance risk
- Option B: Store on smart card/device → privacy-preserving, deduplication hard
- Option C: One-way biometric template → deduplication possible, original not recoverable
- MOSIP uses Option A with strict access controls

**Token/VID strategy:**
- Never expose root ID to service providers
- Issue per-partner tokens (Virtual IDs)
- Tokens are one-way; cannot be reverse-mapped to root ID

**Authentication flows:**
```
OTP Auth: User → Auth API (ID + OTP) → SMS Gateway → User confirms OTP → Auth Result
Bio Auth:  User (device) → Auth API (ID + encrypted biometric) → ABIS → Auth Result
Demo Auth: User → Auth API (ID + demographic hash) → Auth Result
```

## Payments Layer Architecture

### Instant Payment System Components
```
┌─────────┐    ┌──────────────┐    ┌─────────┐
│  PSP A  │←──→│   Clearing   │←──→│  PSP B  │
│ (Payer) │    │   Switch     │    │ (Payee) │
└─────────┘    │  ─────────   │    └─────────┘
               │  Routing     │
               │  Validation  │    ┌─────────────┐
               │  Limits      │←──→│  RTGS/      │
               │  Monitoring  │    │  Settlement  │
               └──────────────┘    └─────────────┘
                       ↕
               ┌──────────────┐
               │  Directory   │
               │  (Alias →    │
               │   Account)   │
               └──────────────┘
```

### Key Design Decisions: Payments

**Routing architecture:**
- Hub-and-spoke (UPI model): All transactions route through central switch
- Bilateral settlement (PIX model): PSPs settle directly, central for clearing only
- Trade-off: Hub = simpler oversight, single point; bilateral = resilient, complex

**Settlement finality:**
- RTGS: Each transaction is final; high cost
- DNS with intraday: Nets down, settles 4-6x daily; lower cost, some credit risk
- Hybrid: Small transactions net, large transactions gross

## Data Exchange Layer Architecture

### Consent Manager Architecture
```
┌─────────────┐    ┌─────────────────┐    ┌──────────────┐
│ Data         │    │  Consent         │    │ Data          │
│ Consumer    │←──→│  Manager (AA)    │←──→│ Provider      │
│ (FIU)       │    │                 │    │ (FIP)         │
└─────────────┘    │  ─────────────  │    └──────────────┘
                   │  Consent Store  │
                   │  Consent API    │         ↕
                   │  Notification   │    ┌──────────────┐
                   └─────────────────┘    │ Consent      │
                                         │ Artefact     │
                          ↕              │ Registry     │
                   ┌─────────────┐       └──────────────┘
                   │ Holder/User │
                   │ (App/Portal)│
                   └─────────────┘
```

## Microservices Patterns for DPI

### Service Decomposition Strategy
Split by:
1. **Domain boundary**: ID services ≠ payment services ≠ data services
2. **Change frequency**: Frequently-updated logic in smaller services
3. **Team ownership**: One team owns one service end-to-end
4. **Scaling needs**: High-volume auth separate from low-volume enrollment

### Communication Patterns
```
Synchronous:  Service A ──REST/gRPC──→ Service B (request-response)
Asynchronous: Service A ──Event──→ Queue ──→ Service B (fire and forget)
Pub/Sub:      Service A ──publishes──→ Topic ←──subscribes── Service B, C, D
```

**When to use async:**
- Enrollment processing (multi-step, non-instant)
- Notification dispatch
- Batch reconciliation
- Audit log writing

**When to use sync:**
- Authentication (must be real-time)
- Payment authorization (sub-second required)
- Real-time eKYC

### API Design Best Practices for DPI
```yaml
# OpenAPI example for DPI API
/v1/auth/otp:
  post:
    summary: Authenticate via OTP
    security:
      - BearerAuth: []
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [vid, otp, txn_id]
            properties:
              vid:    { type: string, description: Virtual ID }
              otp:    { type: string, minLength: 6, maxLength: 6 }
              txn_id: { type: string, format: uuid }
    responses:
      '200':
        description: Authentication result
        content:
          application/json:
            schema:
              properties:
                status:    { type: string, enum: [Y, N] }
                auth_token: { type: string }
      '429': { description: Rate limit exceeded }
      '422': { description: Validation error }
```

## Security Architecture

### Zero Trust for DPI
- Every service-to-service call is authenticated and authorized
- mTLS between internal services
- JWT/OAuth tokens with short TTL
- No implicit trust based on network location

### Encryption Architecture
```
Data at rest:   AES-256-GCM; encryption keys in HSM/KMS
Data in transit: TLS 1.3 minimum; HSTS; certificate pinning for mobile
Database:       Column-level encryption for PII (biometrics, demographics)
Backup:         Encrypted with separate key set
Audit logs:     WORM (Write Once Read Many); hash-chained
```

### Key Management
- HSM (Hardware Security Module) for root keys
- Key rotation: Automated, zero-downtime
- Key escrow: Multi-party custodianship for national keys
- Vendor: Thales, AWS CloudHSM, Azure Dedicated HSM, HashiCorp Vault

### API Security Layers
```
Internet → WAF → DDoS Protection → API Gateway → Auth/Authz → Service
                                       ↓
                                  Rate Limiting
                                  IP Allowlisting
                                  JWT Validation
                                  Input Validation
```

## High Availability and Disaster Recovery

### Availability Targets
| Component | SLA | RPO | RTO |
|-----------|-----|-----|-----|
| Authentication API | 99.99% | 0 | < 5 min |
| Payment Switch | 99.999% | 0 | < 1 min |
| Enrollment | 99.9% | 1 hour | 4 hours |
| Data Exchange | 99.99% | 0 | < 10 min |

### Multi-Region Architecture
```
Primary Region              Secondary Region
─────────────              ──────────────────
Active Services            Active/Standby Services
Primary DB                 Replica DB (real-time sync)
      ↕                          ↕
Active-Active (Payments)  Active-Passive (Enrollment)
      ↕
Global Load Balancer / GeoDNS
```

### Chaos Engineering
- Inject failures in test environments
- Validate recovery procedures
- Tools: Chaos Monkey, Gremlin, Litmus (for Kubernetes)

## Scalability Patterns

### Horizontal Scaling
- Stateless services (auth tokens, not session state)
- Database read replicas
- CDN for static assets and API responses
- Caching at multiple layers (Redis, Varnish)

### Database Patterns for DPI Scale
- **Identity at 1B scale**: PostgreSQL with partitioning + read replicas; CQRS
- **Payments at 10B tx/month**: Time-series partitioned PostgreSQL or Cassandra
- **Consent data**: PostgreSQL with JSON for flexible consent artifacts
- **Audit logs**: ClickHouse or similar columnar DB for analytics

### Performance Benchmarks (Reference)
| Operation | Target Latency | Target Throughput |
|-----------|---------------|------------------|
| OTP Authentication | < 300ms P99 | 10,000 TPS |
| Payment Authorization | < 100ms P99 | 50,000 TPS |
| eKYC Response | < 1s P99 | 1,000 TPS |
| Consent Grant | < 500ms P99 | 5,000 TPS |
| Biometric Match | < 2s P99 | 500 TPS |

## Infrastructure as Code

### Recommended Stack
```
Orchestration:  Kubernetes (K8s)
Service Mesh:   Istio or Linkerd
IaC:            Terraform + Helm
CI/CD:          GitLab CI / GitHub Actions / ArgoCD
Observability:  Prometheus + Grafana + Loki + Jaeger
Secret Mgmt:    HashiCorp Vault / AWS Secrets Manager
Registry:       Harbor (container images)
```

### MOSIP Kubernetes Deployment
```yaml
# Values.yaml structure
global:
  mosip:
    langCodes: ["eng", "hin", "tam"]
    country: IND
  postgres:
    enabled: true
    host: postgres.mosip.svc
    
modules:
  preregistration:
    enabled: true
    replicaCount: 2
  regclient:
    enabled: true
  idrepo:
    enabled: true
    replicaCount: 3
```

## Integration Architecture Patterns

### Event-Driven Integration
```
Enrollment Complete → Kafka Topic → Subscribers:
  - Deduplication Service (checks for duplicates)
  - Notification Service (sends SMS confirmation)
  - Analytics Service (updates dashboards)
  - QA Service (triggers quality checks)
```

### Saga Pattern for Distributed Transactions
For multi-step operations spanning services (e.g., payment + notification + ledger):
```
Orchestrator Saga:
  1. Reserve funds → success
  2. Debit payer account → success
  3. Credit payee account → success (or compensate step 2 if fails)
  4. Notify both parties → best effort
```

## Technical Governance

### API Versioning Policy
- Semantic versioning: major.minor.patch
- Breaking changes require new major version
- Old version supported for 12-24 months after new version
- Deprecation notices in API responses

### Change Management
- RFC process for breaking changes
- API changelog published
- Test environments for partner integration testing
- Staged rollout: canary → 10% → 50% → 100%

### Documentation Standards
- Every API documented in OpenAPI 3.x
- Postman collections published for all APIs
- Sandbox environments for each DPI component
- SDK/libraries for top languages (Java, Python, JS)
