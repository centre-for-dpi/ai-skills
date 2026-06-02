---
name: verifiable-credentials
description: W3C Verifiable Credentials, DIDs, OpenID4VC, SD-JWT, CDPI decision framework, eLockers, VC stack comparison. Use when implementing VC systems, digital credentials, or evaluating VC platforms.
---

# Verifiable Credentials (VCs)

> **CDPI VC Platform**: https://vc.cdpi.dev/  
> **CDPI VC Sandbox**: https://vc.sandbox.cdpi.dev/  
> **CDPI VC Use Cases**: https://vc-use-cases.cdpi.dev/  
> **CDPI Technical Notes — VCs**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/verifiable-credentials  
> **CDPI eLockers**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/elockers  
> **CDPI Comparative Analysis of VC Stacks**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/decision-framework-for-verifiable-credentials/comparative-analysis-of-verifiable-credential-stacks

## What are Verifiable Credentials?

A Verifiable Credential is a tamper-evident, cryptographically signed digital document that:
- Asserts claims about a subject (person, organization, thing)
- Is issued by an issuer
- Can be held in a digital wallet
- Can be presented to a verifier and verified without contacting the issuer

**Real-world analogy**: A driving license is a credential issued by a DMV (issuer), held by a driver (holder), and verified by a police officer (verifier) — a VC is the digital, privacy-preserving equivalent.

## The Trust Triangle

```
        Issuer
       /       \
  issues     trusts
     /             \
Holder  --presents→  Verifier
(Wallet)
```

- **Issuer**: Authoritative source (government, university, bank, hospital)
- **Holder**: Person/entity the credential is about; stores in wallet
- **Verifier**: Relies on the credential to provide a service

## W3C VC Data Model (v1.1 / v2.0)

The W3C Verifiable Credentials Data Model is the global standard:

### Core VC Structure (JSON-LD)
```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://example.org/examples/v1"
  ],
  "type": ["VerifiableCredential", "UniversityDegreeCredential"],
  "issuer": "did:example:university123",
  "issuanceDate": "2023-01-01T00:00:00Z",
  "expirationDate": "2033-01-01T00:00:00Z",
  "credentialSubject": {
    "id": "did:example:student456",
    "degree": {
      "type": "BachelorDegree",
      "name": "Bachelor of Computer Science"
    }
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2023-01-01T00:00:00Z",
    "verificationMethod": "did:example:university123#key-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "z3MvGcVxzr..."
  }
}
```

### Verifiable Presentation (VP)
A presentation packages one or more VCs to present to a verifier:
```json
{
  "@context": ["https://www.w3.org/2018/credentials/v1"],
  "type": ["VerifiablePresentation"],
  "verifiableCredential": [ { ...VC... } ],
  "proof": {
    "type": "Ed25519Signature2020",
    "challenge": "random-challenge-from-verifier",
    "domain": "verifier.example.com",
    "proofValue": "holder's signature..."
  }
}
```

## Decentralized Identifiers (DIDs)

DIDs are the identifier system underlying VCs — globally unique, resolvable without a central registry.

### DID Format
```
did:method:method-specific-identifier
did:web:example.com
did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK
did:ethr:0x1234567890abcdef...
did:ion:EiAnKD8...
```

### Common DID Methods
| Method | Description | Storage |
|--------|-------------|---------|
| `did:web` | DID document at HTTPS URL | Web server |
| `did:key` | DID derived from public key | Self-contained |
| `did:jwk` | DID derived from JWK | Self-contained |
| `did:ion` | Microsoft's DID on Bitcoin | Bitcoin/IPFS |
| `did:ebsi` | EU Blockchain Service Infra | EBSI blockchain |
| `did:ethr` | Ethereum-based DID | Ethereum |
| `did:peer` | Peer-to-peer, no registry | None |

### DID Document
```json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:web:university.example.com",
  "verificationMethod": [{
    "id": "did:web:university.example.com#key-1",
    "type": "Ed25519VerificationKey2020",
    "controller": "did:web:university.example.com",
    "publicKeyMultibase": "z6MkhaXgBZ..."
  }],
  "assertionMethod": ["#key-1"],
  "authentication": ["#key-1"]
}
```

## Proof Formats and Signature Suites

### JSON-LD Proof Suites
- **Ed25519Signature2020**: EdDSA over Ed25519; compact, fast
- **BbsBlsdSignature2020**: BBS+ signatures; enables selective disclosure
- **JsonWebSignature2020**: JWT-style signature in JSON-LD
- **EcdsaSecp256k1Signature2019**: Ethereum-compatible

### JWT/SD-JWT Format
- **JWT VC** (RFC): Standard JWT with VC claims; widely adopted
- **SD-JWT** (IETF draft): Selective disclosure extension to JWT
- **SD-JWT VC**: SD-JWT formatted as VC (OpenID4VC preferred format)

### SD-JWT Selective Disclosure
```
# Issuer creates disclosures:
~WyJyYW5kb20tc2FsdCIsICJhZ2UiLCAyNV0  → {"age": 25}
~WyJyYW5kb20tc2FsdCIsICJuYW1lIiwgIkFsaWNlIl0 → {"name": "Alice"}

# Holder presents only age (omits name):
eyJhbGci...header+payload~WyJyYW5kb20tc2FsdCIsICJhZ2UiLCAyNV0
```

### MDOC / ISO 18013-5 (Mobile Driving License)
- ISO standard for mobile credentials
- Used for driver licenses in US states, EU
- CBOR encoding, device-bound, proximity presentation (NFC/BLE/QR)
- Supports offline verification

## OpenID for Verifiable Credentials (OpenID4VC)

OpenID4VC is the protocol suite for VC issuance and presentation over OIDC:

### OpenID4VCI (Credential Issuance)
```
Holder Wallet → Credential Offer Endpoint (Issuer)
              ← Credential Offer (URI or QR code)
Holder Wallet → Authorization Endpoint (authenticate)
              ← Authorization Code
Holder Wallet → Token Endpoint
              ← Access Token
Holder Wallet → Credential Endpoint
              ← Credential (VC/SD-JWT/mDOC)
```

### OpenID4VP (Verifiable Presentation)
```
Verifier → Authorization Request (request_uri or direct)
         ← VP Token (verifiable presentation)
Verifier verifies presentation & credential
```

### SIOPv2 (Self-Issued OpenID Provider v2)
- Wallet acts as OIDC provider
- User controls their own identity token
- No external IdP required

## Trust Frameworks and Governance

### What is a Trust Framework?
Rules defining:
- Which issuers are trusted for which credential types
- What credential schemas are acceptable
- Liability and compliance obligations
- Technical requirements (schema, proof type)
- Revocation mechanisms

### Examples
- **EU eIDAS 2.0**: European Digital Identity Wallet trust framework
- **EBSI**: European Blockchain Services Infrastructure
- **ToIP (Trust over IP)**: Layered trust framework model
- **DIF (Decentralized Identity Foundation)**: Interoperability standards
- **INJI Trust Registry**: MOSIP's open-source trust registry
- **Gaia-X**: European data/cloud trust framework

### Governance Layers (ToIP Model)
```
Layer 4: Application Ecosystems (specific use cases)
Layer 3: Credential Exchange Protocols (OpenID4VC, DIDComm)
Layer 2: DID Methods & Peer-to-Peer (key management)
Layer 1: Utility Ledgers & Registries (anchoring)
```

## Credential Revocation

### Methods
| Method | Description | Privacy | Scalability |
|--------|-------------|---------|-------------|
| **Status List 2021** | Bitstring, fetch from URL | Low (correlation) | High |
| **OCSP** | Real-time query to issuer | Low | Medium |
| **CRL** | Periodically published list | Medium | Medium |
| **Accumulator** | Cryptographic; no correlation | High | Low |
| **Short-lived VC** | Expires in hours/days | High | High |

### Status List 2021 (W3C)
```json
"credentialStatus": {
  "id": "https://issuer.example.com/status/3#94567",
  "type": "StatusList2021Entry",
  "statusPurpose": "revocation",
  "statusListIndex": "94567",
  "statusListCredential": "https://issuer.example.com/status/3"
}
```

## DPI Use Cases for VCs

### Government
- **National ID as VC**: Aadhaar-based VC, MOSIP-issued credentials
- **Birth/Death certificates**: Digital, shareable, verifiable
- **Education credentials**: Degree certificates (India DigiLocker, Learner Passport)
- **Property records**: Title deeds as VCs
- **Business registration**: Company credentials

### Health
- **COVID certificates**: WHO DDCC, EU Digital COVID Certificate
- **Vaccination records**: WHO SMART Health Cards
- **Prescriptions**: Verifiable prescription credentials
- **Professional licenses**: Doctor/nurse credentials

### Finance
- **KYC credentials**: Once-verified KYC portable across banks
- **Credit history VCs**: Portable credit score/history
- **Income verification**: Payroll-attested income VC

### Cross-Border
- **ICAO Travel Credentials**: Digital travel credentials
- **Educational recognition**: Cross-border degree recognition
- **Professional qualifications**: Mutual recognition

## Inji — MOSIP's VC Wallet

(See also: dpi-inji skill)

Inji Wallet is MOSIP's open-source VC wallet:
- Issues MOSIP Identity VC (OpenID4VCI)
- Stores VCs on device (encrypted)
- Presents via QR code (offline) or OpenID4VP (online)
- BLE-based proximity sharing
- Supports SD-JWT VC format

## CDPI Decision Framework for Verifiable Credentials

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/decision-framework-for-verifiable-credentials

CDPI provides a decision framework to help countries choose the right VC approach:

### Key Decision Dimensions

| Dimension | Questions to Ask |
|-----------|-----------------|
| **Use Case** | Online or offline verification? Cross-border? Selective disclosure needed? |
| **Privacy** | Can the verifier correlate holder across verifiers? Can issuer track presentations? |
| **Format** | SD-JWT VC (web/mobile), mDOC (proximity/government ID), or JSON-LD VC (rich semantics) |
| **Revocation** | How frequently does status change? Can verifier check status offline? |
| **Binding** | Device-bound (hardware key) or holder-bound (DID key)? |
| **Interoperability** | Does the ecosystem already use a specific wallet or protocol? |

### CDPI Format Recommendations
- **SD-JWT VC**: Preferred for most government-to-citizen credential use cases (compact, selective disclosure, widely supported)
- **mDOC (ISO 18013-5)**: Preferred for proximity/offline use cases (border crossing, age verification at POS)
- **JSON-LD VC**: Preferred where rich semantics and linked data are important (academic credentials with complex ontologies)

### Trust Registry Approach
CDPI recommends a **federated trust registry** model:
- Central registry publishes trusted issuer DIDs and credential schemas
- Verifiers resolve issuers against the registry
- Registry itself should be a DPI component (open, governed publicly)
- INJI Trust Registry and Sunbird RC are reference implementations

## CREDEBL — Open-Source VC Platform (LF Decentralized Trust)

> **Reference**: https://www.lfdecentralizedtrust.org/projects/credebl  
> **See also**: dpi-credebl skill

CREDEBL is an open-source platform under the Linux Foundation Decentralized Trust project for issuing and verifying verifiable credentials at scale. It is positioned as production-grade VC infrastructure suitable for DPI deployments:
- Multi-tenancy: Issue credentials for multiple organizations from one platform
- AnonCreds and W3C VC support
- OpenID4VCI and OpenID4VP protocols
- REST APIs for credential issuance and verification
- Agent-based architecture (Aries compatible)

## Key Standards & Specifications

| Standard | Body | Status |
|----------|------|--------|
| VC Data Model v2.0 | W3C | Candidate Recommendation |
| DID Core | W3C | Recommendation |
| OpenID4VCI | OpenID Foundation | Draft |
| OpenID4VP | OpenID Foundation | Draft |
| SIOPv2 | OpenID Foundation | Draft |
| SD-JWT | IETF | Draft RFC |
| SD-JWT VC | IETF | Draft |
| ISO 18013-5 (mDL) | ISO | Published |
| BBS+ Signatures | IETF | Draft |
| Status List 2021 | W3C CCG | Draft |

## Implementation Checklist

When building a VC system:
- [ ] Define credential schema (JSON Schema + JSON-LD context)
- [ ] Choose proof format (SD-JWT VC for most cases, mDOC for proximity)
- [ ] Choose DID method for issuer (did:web is simplest)
- [ ] Implement OpenID4VCI for issuance
- [ ] Implement OpenID4VP for presentation
- [ ] Define revocation mechanism (Status List 2021 recommended)
- [ ] Register in trust registry
- [ ] Define holder binding (how wallet proves it's the subject)
- [ ] Test with multiple wallet implementations
- [ ] Publish credential schema in public registry

---

## CDPI Vision: User-Centric Credentialing

> **Reference**: https://vc.cdpi.dev/ | https://cdpi.dev/verifiable-credentials-vision-paper/

Traditional credentialing is institution-centric: institutions hold data, verify data, and control access. CDPI's vision flips this:

**Institution-centric (status quo)**:
```
Institution A holds data → Citizen requests → Institution B verifies → Days or weeks → High cost
```

**User-centric (VCs as DPI)**:
```
Issuer signs credential → Citizen holds in wallet → Citizen presents to verifier → Seconds → Near-zero cost
```

**The transformation**:
- Secure, trusted data placed directly in hands of individuals
- Verification cost collapses (India: $23 KYC → $0.15 with VC-based eKYC)
- No institutional dependency for each verification
- Marginalized populations gain equitable access to healthcare, finance, education, employment

### Three Technical Building Blocks (CDPI)

| Building Block | Purpose |
|---------------|---------|
| **Data sharing APIs** | Standardised interfaces for issuers and verifiers to exchange credentials |
| **Sector-specific data schemas** | Common vocabulary per domain (health, education, finance) enabling interoperability |
| **Consent standard** | Granular, purpose-specific, revocable consent artefacts governing data flows |

### Core Design Principles

- **User in control**: Citizen specifies which data to share, with whom, and for how long
- **No central data store**: Data flows real-time on consent; no aggregation
- **Granular consent**: Purpose-specific (e.g. "for this loan application"), not blanket
- **Open network**: Multiple wallet apps and verifier apps built by anyone over the same APIs
- **Machine-readable + verifiable**: Anyone can verify without contacting the issuer

---

## eLockers — Federated Credential Repositories

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/elockers

An **eLocker** is a managed, cloud-hosted credential repository operated by one or more entities in a country. It is distinct from a personal wallet — it is a federation layer for verifiable credentials.

### eLocker vs Wallet

| Feature | eLocker | Personal Wallet |
|---------|---------|----------------|
| **Location** | Cloud-hosted by trusted entity | On-device (phone/device) |
| **Managed by** | Government, institution, or trusted operator | Individual citizen |
| **Access model** | URI-based; consent required to fetch | Holder-controlled |
| **Use case** | Repository for many citizens' documents | Individual credential storage and presentation |
| **Interoperability** | Open APIs enable any app to access | Wallet-specific protocol (OpenID4VP) |

### How eLockers Work

```
Issuer (government dept / hospital / university)
        ↓ digitally signed credential
eLocker (stores credential, assigns URI)
        ↓ citizen holds URI reference
Verifier requests document
        ↓ citizen provides URI + grants consent
eLocker gateway verifies consent artefact
        ↓ authenticates + timestamps
Verifier receives document
```

### eLocker Open API Design

eLockers publish **two sets of open APIs**:

1. **Issuer API**: Any authorized department (state, local, central) can push digitally signed credentials into the eLocker — education, health, land records, etc.

2. **Requestor API**: Any authorized verifier (bank, employer, government agency) can request a specific document from a citizen's eLocker, subject to consent.

**DPI principle**: Open APIs enable any market player to build wallet apps, verifier apps, or consent UIs on top of the eLocker infrastructure — creating an open ecosystem, not a single government application.

### eLocker Reference Implementations

| Country | System | Scope |
|---------|--------|-------|
| India | DigiLocker | 6B+ documents, 1800+ issuers, 300M+ users |
| Ethiopia | National eLocker initiative | Government document federation |
| Generic | eLocker spec (CDPI) | Country-configurable open-source model |

---

## CDPI Comparative Analysis of VC Stacks

> **Reference**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/decision-framework-for-verifiable-credentials/comparative-analysis-of-verifiable-credential-stacks

CDPI evaluated four DPGs for its DPI-as-a-Packaged-Solution (DaaS) Cohort 2 digital credentials offering:

| Stack | Lead Format | Strength | Best For |
|-------|------------|----------|---------|
| **Inji** (MOSIP) | JSON-LD + Linked Data Proofs | BLE offline, native proximity, single-issuer depth | National ID credentials, offline-first use cases |
| **walt.id** | SD-JWT VC + mDOC | Broadest format versatility (SD-JWT, mDOC, JSON-LD) | Multi-format ecosystems, eIDAS 2.0 compliance |
| **CREDEBL** (LF Decentralized Trust) | AnonCreds + SD-JWT + mDOC | SaaS multi-tenancy, broadest revocation coverage | "Credentialing as a Service" platforms |
| **QuarkID** | SD-JWT + ZKP | ZKP/AnonCreds + BLE offline | Privacy-preserving credentials, SaaS platforms |

### Detailed Comparison

| Dimension | Inji | walt.id | CREDEBL | QuarkID |
|-----------|------|---------|---------|---------|
| **SD-JWT VC** | Partial | Full | Full | Full |
| **mDOC (ISO 18013-5)** | No | Full | Partial | No |
| **AnonCreds (ZKP)** | No | No | Full | Full |
| **JSON-LD VC** | Full | Full | Partial | Partial |
| **Multi-tenant SaaS** | No (single issuer) | Yes (Enterprise) | Yes (native) | Yes (native) |
| **BLE offline** | Yes (native) | No | No | Yes |
| **Offline QR** | Yes | No | No | Yes |
| **Revocation (AnonCreds)** | N/A | N/A | Tails files + accumulators | Tails files |
| **Revocation (SD-JWT)** | Status List 2021 | Token Status List | Token Status List | Token Status List |
| **Revocation (mDOC)** | N/A | Supported | Token Status List | N/A |

### Selection Decision Guide

**Choose Inji** when:
- National ID-linked VC issuance is the core use case
- Offline proximity sharing (BLE) is required
- MOSIP ecosystem integration is needed

**Choose walt.id** when:
- Multi-format support (SD-JWT + mDOC + JSON-LD) is required in one stack
- eIDAS 2.0 / European wallet compliance matters
- Enterprise SaaS deployment with many issuers

**Choose CREDEBL** when:
- Building "Credentialing as a Service" for many organizations
- Broadest revocation coverage across all three major formats is needed
- Open-source, LF Decentralized Trust governance is required

**Choose QuarkID** when:
- Zero-knowledge proof / AnonCreds privacy is a requirement
- BLE offline sharing + SaaS multi-tenancy are both needed
- Latin America ecosystem (QuarkID originates from Argentina)

---

## CDPI VC Sandbox

> **Reference**: https://vc.sandbox.cdpi.dev/

The CDPI VC Sandbox is a **free, browser-based demonstration environment** for verifiable credentials. It lets anyone experience VC issuance and verification without needing to deploy any infrastructure.

### What the Sandbox Enables

- **Manual credential creation**: Enter data fields directly in the browser
- **Bulk credential creation**: Upload an Excel spreadsheet to generate credentials in bulk
- **Credential verification**: Verify any credential generated in the sandbox
- **End-to-end demo**: Experience the full issuer → holder → verifier flow
- **No setup required**: Works in any browser; no account needed

### Technology

- Built entirely on open-source technology
- Powered by Digital Public Goods (DPGs)
- Uses OpenID4VCI for issuance and OpenID4VP for presentation flows

### Use Cases for the Sandbox

- **Country teams**: Prototype credential schemas before committing to infrastructure
- **Government workshops**: Demonstrate VC concepts to policy officials in real time
- **CDPI Bootcamps**: Colombia's team built a live pilot starting from the sandbox

---

## CDPI VC Use Cases

> **Reference**: https://vc-use-cases.cdpi.dev/

### Colombia — Six-Day CDPI Bootcamp

Colombia's Ministry of ICT (MinTIC) led an intensive CDPI Bootcamp with 40+ officials from multiple agencies:

**Cédula Rural Digital** (Ministry of Agriculture + FINAGRO):
```
Farmer holds National ID
        ↓ FINAGRO issues agricultural credential
Cédula Rural Digital (VC in wallet)
        ↓ presented at bank or cooperative
Access to agricultural credit + benefits — no paper documents
```

**Temporary Protection Permit (PPT)** (Migración Colombia):
```
Migrant has Temporary Protection Permit (paper)
        ↓ Migración Colombia issues VC
Digital PPT in wallet
        ↓ presented to employer, bank, service provider
Validation time: 3 days → 30 seconds
```

**Full implementation in 6 days**: Data modelling, credential schema design, issuance flows, verification infrastructure, and integration architecture — with technical ownership transferred to Colombia's teams.

### Benin — Education and Social Protection

CDPI is supporting Benin in rolling out VC ecosystems for:
- Education credentials (diplomas, certificates as VCs)
- Social protection eligibility credentials (benefit access without paper documents)

### Trinidad & Tobago — National Scale Deployment

Implementing a verifiable credentials stack at national scale as part of the DaaS program — one of the first DaaS Cohort 2 implementations to go live.

### Other CDPI Country VC Work

| Country | Use Case | Status |
|---------|----------|--------|
| Brazil | Farmer credentials, sector-specific VCs | Active |
| Dominican Republic | Whole-of-government DPI strategy incl. VCs | Advisory |
| Mexico | National health ID | Active |
| Jamaica | Tax compliance credentials | Active |

---

## DaaS — Digital Credentials as a Packaged Solution

> **Reference**: https://docs.cdpi.dev/initiatives/dpi-as-a-packaged-solution-daas/cohort-1-daas-offerings/digital-credentials

CDPI's **DPI as a Packaged Solution (DaaS)** program bundles vetted DPGs with implementation support so countries can deploy production VCs rapidly.

**DaaS Cohort 2 — Digital Credentials offering**:
- Countries select from four pre-evaluated DPG stacks: Inji, walt.id, CREDEBL, QuarkID
- CDPI provides: implementation support, technical architecture guidance, capacity building
- Countries receive: schema design, issuance pipeline, verifier integration, trust registry setup
- Funding available through CDPI donor partners (World Bank, Gates Foundation, AfDB, UNDP, Co-Develop)

**DaaS value proposition**: A country can go from zero to live VC issuance in weeks, not years, using pre-evaluated open-source technology with active implementation support.
