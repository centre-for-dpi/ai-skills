---
name: dpi-credebl
description: CREDEBL — open-source enterprise VC platform (LF Decentralized Trust), multi-tenant, AnonCreds, OpenID4VC. Use when implementing VC issuance infrastructure or evaluating CREDEBL for DPI.
---

# CREDEBL — Enterprise Verifiable Credentials Platform

## What is CREDEBL?

CREDEBL (Credential Enablement for Decentralized Ledger Blockchains) is an open-source, enterprise-grade platform for issuing, managing, and verifying Verifiable Credentials at scale. It is a project under the **Linux Foundation Decentralized Trust** (LF Decentralized Trust, formerly Hyperledger).

> **Reference**: https://www.lfdecentralizedtrust.org/projects/credebl  
> **GitHub**: https://github.com/credebl/platform  
> **Documentation**: https://docs.credebl.id/

CREDEBL is designed for production DPI deployments where:
- Multiple organizations need to issue credentials (multi-tenancy)
- High volumes of credential issuance are required (millions per day)
- Standards compliance is mandatory (W3C VC, OpenID4VC)
- No proprietary vendor lock-in is acceptable

## Core Capabilities

### Multi-Tenancy
- A single CREDEBL deployment can serve hundreds of issuing organizations
- Each organization (tenant) has isolated keys, schemas, and credential definitions
- Tenant onboarding via admin API or self-service portal
- Per-tenant audit logs and analytics

### Protocol Support
- **OpenID4VCI**: Credential issuance (pre-authorized and authorization code flows)
- **OpenID4VP**: Credential presentation and verification
- **AnonCreds** (Hyperledger AnonCreds): Privacy-preserving credentials with selective disclosure and ZKP
- **W3C VC + JSON-LD**: Standard credential format
- **SD-JWT VC**: Selective disclosure JWT credentials (evolving support)

### DID Methods Supported
- `did:indy` (Hyperledger Indy ledger)
- `did:web`
- `did:key`
- `did:peer`

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CREDEBL Platform                      │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐ │
│  │  API Gateway  │  │  Studio UI   │  │  Admin Console    │ │
│  │  (REST APIs)  │  │  (Web)       │  │  (Tenant Mgmt)    │ │
│  └──────────────┘  └──────────────┘  └───────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │               Core Services                          │   │
│  │  Organization │ Connection │ Credential │ Verification│   │
│  │  Service      │ Service    │ Service    │ Service     │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Agent Service Layer                      │   │
│  │  ┌─────────────┐  ┌──────────────┐                   │   │
│  │  │ Tenant Agent│  │ Platform     │                   │   │
│  │  │ (per org)   │  │ Agent        │                   │   │
│  │  └─────────────┘  └──────────────┘                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐ │
│  │  PostgreSQL  │  │  Ledger      │  │  NATS             │ │
│  │  (state)     │  │  (Indy/EBSI) │  │  (message bus)    │ │
│  └──────────────┘  └──────────────┘  └───────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack
- **Backend**: NestJS (Node.js microservices)
- **Database**: PostgreSQL
- **Message Bus**: NATS
- **Agent Framework**: Hyperledger Credo (formerly Aries Framework JavaScript)
- **Frontend**: React (CREDEBL Studio)
- **Deployment**: Docker Compose / Kubernetes

## Credential Issuance Flow (CREDEBL)

### Step 1: Tenant Setup
```bash
# Create an organization (tenant)
POST /orgs
{
  "name": "Ministry of Education",
  "description": "Issues academic credentials",
  "logoUrl": "https://education.gov/logo.png"
}

# Response
{ "orgId": "org_uuid", "apiKey": "sk_..." }
```

### Step 2: Create Credential Schema
```bash
# Define the credential schema
POST /orgs/{orgId}/schemas
{
  "name": "DegreeCredential",
  "version": "1.0",
  "attributes": ["studentId", "degreeTitle", "institution", "graduationDate"]
}
```

### Step 3: Create Credential Definition (for AnonCreds)
```bash
POST /orgs/{orgId}/credential-definitions
{
  "schemaId": "schema_uuid",
  "tag": "degree-cred-def-v1",
  "revocationRequired": true
}
```

### Step 4: Issue Credential (Out-of-Band / OpenID4VCI)
```bash
# Issue via OpenID4VCI pre-authorized code flow
POST /orgs/{orgId}/credentials/oob/issuance
{
  "credentialDefinitionId": "cred_def_uuid",
  "attributes": {
    "studentId": "2024-CS-001234",
    "degreeTitle": "Bachelor of Computer Science",
    "institution": "National University",
    "graduationDate": "2024-05-15"
  }
}

# Returns credential offer URL / QR code
{
  "offerUrl": "openid-credential-offer://...",
  "qrCodeBase64": "data:image/png;base64,..."
}
```

### Step 5: Holder Accepts in Wallet
```
Holder scans QR in Inji Wallet / CREDEBL Wallet / ACA-Py compatible wallet
  → OpenID4VCI pre-authorized code flow
  → Credential issued and stored in wallet
```

## CREDEBL Studio (Web UI)

CREDEBL Studio provides a no-code interface for:
- Organization registration and management
- Schema and credential definition creation
- Manual credential issuance
- Connection management (DIDComm connections)
- Verification request creation
- Issuance templates for bulk operations
- Analytics dashboard (credentials issued, verified, revoked)

## Bulk Issuance

For high-volume DPI use cases (mass enrollment, universal vaccination record):

```bash
# Bulk issuance via CSV
POST /orgs/{orgId}/credentials/bulk-issuance
Content-Type: multipart/form-data

file: credentials.csv  # CSV with one row per credential
credentialDefinitionId: cred_def_uuid
```

CSV format:
```csv
studentId,degreeTitle,institution,graduationDate,recipientEmail
2024-CS-001234,BSc CS,NatUniv,2024-05-15,student1@example.com
2024-EC-001235,BSc ECE,NatUniv,2024-05-15,student2@example.com
```

## AnonCreds and Privacy

AnonCreds (Hyperledger AnonCreds) provides strong privacy guarantees:

### Key Features
- **Selective Disclosure**: Holder proves specific attributes without revealing others
- **Predicate Proofs**: Prove `age > 18` without revealing exact age
- **Zero-Knowledge Proofs**: Prove possession of credential without revealing credential ID
- **Issuer Unlinkability**: Verifier cannot correlate verification with issuance (issuer not contacted)
- **Verifier Unlinkability**: Multiple presentations to same verifier are unlinkable (using link secrets)

### AnonCreds vs SD-JWT VC Trade-offs

| Dimension | AnonCreds | SD-JWT VC |
|-----------|-----------|-----------|
| **Privacy** | Very high (ZKP, unlinkability) | High (selective disclosure only) |
| **Complexity** | Higher (Indy ledger, ZKP math) | Lower (standard JWT) |
| **Interoperability** | Indy ecosystem only | Broad (any JWT stack) |
| **Mobile performance** | Slower (ZKP computation) | Fast |
| **Standard** | Hyperledger spec | IETF draft |
| **DPI recommendation** | High-sensitivity gov credentials | Most citizen credentials |

## Verification / Presentation Flow

```bash
# Create verification request
POST /orgs/{orgId}/verification/send-request
{
  "connectionId": "connection_uuid",  # or use OOB for wallet-less
  "proofFormats": {
    "anoncreds": {
      "requested_attributes": {
        "degree_attr": {
          "name": "degreeTitle",
          "restrictions": [{ "cred_def_id": "cred_def_uuid" }]
        }
      }
    }
  }
}

# Check verification result
GET /orgs/{orgId}/verification/{proofId}
{
  "state": "done",
  "isVerified": true,
  "proofData": {
    "degreeTitle": "Bachelor of Computer Science",
    "institution": "National University"
  }
}
```

## Deployment

### Docker Compose (Development)
```yaml
version: '3.8'
services:
  platform:
    image: ghcr.io/credebl/platform:latest
    ports:
      - "5000:5000"
    environment:
      DATABASE_URL: postgresql://postgres:password@postgres:5432/credebl
      NATS_URL: nats://nats:4222
    depends_on: [postgres, nats]
      
  studio:
    image: ghcr.io/credebl/studio:latest
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://platform:5000

  agent-service:
    image: ghcr.io/credebl/agent-service:latest
    environment:
      PLATFORM_URL: http://platform:5000
      
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: credebl
      POSTGRES_PASSWORD: password
      
  nats:
    image: nats:latest
```

### Kubernetes (Production)
```bash
helm repo add credebl https://credebl.github.io/helm-charts
helm install credebl credebl/platform \
  --namespace credebl \
  --create-namespace \
  -f values.yaml
```

## CREDEBL in the DPI Stack

### Integration with MOSIP / e-Signet
```
User authenticates via e-Signet (OIDC)
        ↓
Service receives ID token (user verified)
        ↓
Service calls CREDEBL API to issue VC for this user
        ↓
CREDEBL issues VC (e.g., Voter ID, Insurance, Education)
        ↓
User downloads VC to Inji Wallet
        ↓
User presents VC at service provider
        ↓
Service provider verifies via CREDEBL Verification API
```

### CREDEBL vs Inji Certify (Comparison)

| Dimension | CREDEBL | Inji Certify |
|-----------|---------|-------------|
| **Ecosystem** | LF Decentralized Trust / Aries | MOSIP |
| **Protocol** | AnonCreds + OpenID4VC | OpenID4VCI (SD-JWT, JSON-LD) |
| **Multi-tenancy** | Native | Not primary focus |
| **DID methods** | did:indy, did:web, did:key | did:web, did:jwk |
| **Privacy tech** | AnonCreds ZKP | SD-JWT selective disclosure |
| **Maturity** | Production ready | Production ready |
| **Best for** | Multi-org ecosystems, strong privacy | MOSIP-integrated deployments |

## Resources

- **LF Decentralized Trust Project**: https://www.lfdecentralizedtrust.org/projects/credebl
- **GitHub**: https://github.com/credebl/platform
- **Documentation**: https://docs.credebl.id/
- **CREDEBL Studio (SaaS demo)**: https://app.credebl.id/
- **Hyperledger AnonCreds**: https://hyperledger.github.io/anoncreds-spec/
- **Hyperledger Credo**: https://credo.js.org/
