---
name: dpi-trust-infra
description: Trust Infrastructure as DPI — PKI, digital signatures, eIDAS, eSign, eConsent, HSMs, consent artefacts. Use when designing signing infrastructure, evaluating trust frameworks, or implementing PKI.
---

# Electronic Signature, PKI and Trust Infrastructure

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/electronic-signature-pki-and-trust-infra

## Why Trust Infrastructure is DPI

Trust infrastructure enables individuals and organisations to interact and transact **without physical presence or paper**, using digital signatures and public key infrastructure to ensure secure, legally recognised, and non-repudiable transactions.

Without trust infrastructure, every digital interaction requires physical verification — wet signatures, in-person notarisation, paper documents. With it, the entire economy can transact digitally at population scale.

CDPI frames trust infrastructure as a foundational DPI layer because:
- It enables **eSign** — replacing wet signatures with ID-linked digital signatures
- It enables **eConsent** — machine-readable consent that powers data exchange DPI
- It anchors **Verifiable Credentials** — PKI provides the cryptographic trust layer for VCs
- It enables **high-trust digital economy** — opening social protection, health, banking, and legal services to fully digital delivery

The World Bank (2024 PKI DPI Policy Note Series) explicitly frames PKI as essential DPI: *"Electronic signatures and PKI play a foundational role in securing digital interactions, providing authentication, integrity, and legal recognition for transactions in the digital world, and are an essential part of a country's digital public infrastructure."*

## Public Key Infrastructure (PKI)

### Cryptographic Foundations

PKI is built on **asymmetric cryptography** — a mathematically related key pair:

```
Private Key (secret)  ←→  Public Key (shared openly)

Signing:   Hash(document) → Encrypt with Private Key → Digital Signature
Verifying: Decrypt Signature with Public Key → Compare hash → Valid/Invalid
```

**What a digital signature guarantees:**
- **Authenticity**: Only the private key holder could have created this signature
- **Integrity**: Any change to the document after signing invalidates the signature
- **Non-repudiation**: The signer cannot later deny having signed (private key was used)

### Digital Signatures vs Electronic Signatures

| Type | Definition | Legal Weight |
|------|-----------|-------------|
| **Electronic Signature** | Any electronic means of indicating intent to sign (typed name, scanned signature, click-to-accept) | Varies; weakest |
| **Digital Signature** | Cryptographically derived, PKI-backed electronic signature | Stronger; verifiable |
| **Qualified Digital Signature** | Digital signature from a Qualified Trust Service Provider | Highest; legally equivalent to handwritten signature |

Digital signatures are a cryptographically-derived **subset** of electronic signatures. All digital signatures are electronic signatures; not all electronic signatures are digital signatures.

### Certificate Hierarchy

PKI is organised as a trust hierarchy — each level vouches for the level below:

```
Root CA (Root Certifying Authority)
  ├── OFFLINE — never connected to internet
  ├── Trust anchor for the entire PKI
  ├── Signs Intermediate CA certificates
  └── Private key stored in HSM in physically secured facility

    Intermediate CA (Sub-CA)
      ├── Subordinate to Root CA
      ├── Issues certificates to end entities
      ├── Day-to-day certificate operations
      └── Can be multiple levels deep

        End-Entity Certificates
          ├── Individual signers
          ├── Organisations
          ├── Systems (TLS certificates)
          └── Device certificates
```

**Why the Root CA must be offline**: If the Root CA's private key is compromised, the entire trust chain fails. Keeping it air-gapped prevents remote attacks. Certificate issuance from the Root CA is rare — only to certify Intermediate CAs.

### Hardware Security Modules (HSMs)

HSMs are dedicated hardware devices for cryptographic key storage and operations:
- Keys generated and stored inside the HSM — never exported in plaintext
- Tamper-evident: physical intrusion destroys keys
- FIPS 140-2 Level 3 (or higher) certification required for national PKI
- Used for: Root CA keys, Intermediate CA keys, Time Stamping Authority (TSA) keys
- Vendors: Thales/nCipher, Utimaco, AWS CloudHSM, Azure Dedicated HSM

### Certificate Lifecycle

```
Application → Certificate Signing Request (CSR)
                    ↓
              CA validates identity of applicant
                    ↓
              CA issues X.509 certificate (signed with CA private key)
                    ↓
              Certificate used for signing / TLS / auth
                    ↓
              Periodic renewal (typically 1-3 years)
                    ↓
              Revocation if compromised → CRL / OCSP
```

**Certificate Revocation**:
- **CRL (Certificate Revocation List)**: Periodically published list of revoked certificates
- **OCSP (Online Certificate Status Protocol)**: Real-time revocation check; preferred for DPI
- **OCSP Stapling**: Server includes OCSP response with TLS handshake; reduces latency

## eIDAS — The Global Trust Framework Reference

The EU's **eIDAS Regulation** (Electronic Identification, Authentication and Trust Services) is the globally referenced framework for electronic trust services. eIDAS 1.0 (Regulation 910/2014, in force July 2016); eIDAS 2.0 (Regulation EU 2024/1183, in force May 2024).

### Three Levels of Electronic Signatures

| Level | Requirements | Legal Effect |
|-------|-------------|-------------|
| **Simple Electronic Signature (SES)** | Any electronic indication of intent (typed name, click) | Admissible as evidence; lowest weight |
| **Advanced Electronic Signature (AdES)** | Uniquely linked to signer, capable of identifying signer, created with data under signer's sole control, linked to signed data | Stronger evidentiary weight |
| **Qualified Electronic Signature (QES)** | AdES + created by Qualified Electronic Signature Creation Device (QESCD) + based on Qualified Certificate from QTSP | **Legally equivalent to handwritten signature across EU** |

### Trust Service Providers (TSPs)

Under eIDAS:
- **TSP (Trust Service Provider)**: Issues qualified certificates, time stamps, and other trust services
- **QTSP (Qualified Trust Service Provider)**: Audited and approved TSP; appears on national Trusted Lists
- **EU Trust Lists (EUTLs)**: National registries of QTSPs; have constitutive legal effect — a service is "qualified" only if it appears on the list
- **eIDAS 2.0 target**: By 2026, all EU citizens will have government-issued digital identity wallets capable of creating QES from smartphones

### Mutual Recognition

Under eIDAS, a QES from any EU member state is **legally recognised in all EU member states** — eliminating the need for bilateral recognition agreements within the EU.

For cross-border DPI globally, CDPI and the G20 Digital Economy Working Group have proposed establishing **Mutual Recognition Frameworks for PKI and Digital Signatures** to enable borderless digital transactions.

## eSign — Electronic Signature as DPI Service

> **Reference**: https://docs.cdpi.dev/technical-notes/electronic-signature-pki-and-trust-infra/esign

**What eSign is**: A protocol that allows an ecosystem of electronic signature providers to facilitate legally valid signatures as a service — built on top of any identity system.

eSign enables an individual to sign a document **remotely**, using their digital identity for authentication, without printing, scanning, or physically appearing. It is a DPI capability because:
- It is built on the identity rail (authentication uses the national ID system)
- It is exposed as an open API (any application can integrate eSign)
- It reduces friction for every document-based government and financial service

### How eSign Works

```
1. User initiates document signing in any application
         ↓
2. Application calls eSign API with document hash
         ↓
3. eSign service authenticates user via identity rail
   (OTP to registered mobile, biometric, or PIN)
         ↓
4. On successful authentication, eSign service:
   - Creates a Digital Signature Certificate (DSC) bound to the user
   - Signs the document hash with the user's private key (HSM-backed)
   - Appends a timestamp from a trusted Time Stamping Authority (TSA)
         ↓
5. Signed document returned to application
   - Signature is legally valid, non-repudiable
   - Tamper-evident: any change to document invalidates signature
```

### eSign Architecture

```
Application                eSign Service              Identity Rail
    │                           │                          │
    │── sign(doc_hash, user) ──→│                          │
    │                           │── authenticate(user) ──→│
    │                           │←── auth_token ──────────│
    │                           │                          │
    │                           │ [Generate ephemeral keypair]
    │                           │ [Sign doc_hash with private key in HSM]
    │                           │ [Timestamp from TSA]
    │                           │                          │
    │←── signed_doc, cert ──────│                          │
```

### eSign Use Cases in DPI

| Use Case | Without eSign | With eSign |
|---------|--------------|-----------|
| Bank account opening | Print, sign, scan form | Sign digitally in 2 minutes |
| Loan agreement | Visit branch, sign physically | Remote signing via app |
| Government benefits enrolment | Paper form, in-person | Online enrolment with eSign |
| Property registration | Multiple in-person visits | Digital deed signing |
| Educational certificate | Physical certificate, attested copies | Digitally signed verifiable credential |
| Business contract | Exchange physical copies | e-signed digital agreement |

### eSign Technical Standards

- **PDF Signatures**: PAdES (PDF Advanced Electronic Signatures) — ETSI EN 319 100
- **XML Signatures**: XAdES (XML Advanced Electronic Signatures) — ETSI EN 319 132
- **CMS Signatures**: CAdES (CMS Advanced Electronic Signatures) — ETSI EN 319 122
- **JSON Signatures**: JAdES — ETSI TS 119 182 (emerging)
- **W3C VC Proofs**: JSON-LD Signatures, SD-JWT, Data Integrity

## eConsent — Consent Infrastructure

> **Reference**: https://docs.cdpi.dev/technical-notes/electronic-signature-pki-and-trust-infra/econsent

**What eConsent is**: A machine-readable electronic document (consent artefact) that records and stores consent for a data-sharing agreement, specifying the parameters and scope of data sharing a user agrees to.

eConsent is the trust mechanism that makes **data exchange DPI** work. Without machine-readable, digitally signed consent, data sharing requires manual verification of paper consent forms.

### Consent Artefact Structure

A consent artefact is a structured, digitally signed JSON/XML document:

```json
{
  "consentId": "uuid",
  "timestamp": "ISO8601",
  "status": "ACTIVE",
  "consentDetail": {
    "consentStart": "2024-01-01T00:00:00Z",
    "consentExpiry": "2025-01-01T00:00:00Z",
    "dataConsumer": {
      "id": "lender-entity-id",
      "type": "FIU"
    },
    "dataProvider": {
      "id": "bank-entity-id",
      "type": "FIP"
    },
    "purpose": {
      "code": "101",
      "text": "Loan underwriting"
    },
    "fiTypes": ["DEPOSIT", "TERM_DEPOSIT", "MUTUAL_FUNDS"],
    "DataLife": { "unit": "YEAR", "value": 1 },
    "Frequency": { "unit": "MONTH", "value": 1, "count": 12 },
    "DataFilter": [
      { "type": "TRANSACTIONAMOUNT", "operator": ">=", "value": "0" }
    ]
  },
  "consentUse": { "count": 0, "lastUseDateTime": null },
  "signature": {
    "consentManagerSignature": "...",
    "userSignature": "..."
  }
}
```

**Key fields:**
- **Identifier section**: All entities involved (user, data provider, data consumer, consent manager)
- **Data section**: Type of data, specific fields, date range, duration, frequency of access
- **Purpose**: Legally specified reason for data access (purpose limitation)
- **Signature block**: Digitally signed — immutable record of consent

### Consent Models

**User-Driven (Direct)**
```
Data Consumer requests user consent directly
User reviews: data type, purpose, duration, frequency
User approves → Consent artefact generated
Artefact sent to Data Provider → Data shared
```

**Consent Manager Model (DEPA / Account Aggregator)**
```
Data Consumer → Consent Manager (request data)
Consent Manager → User (presents consent request)
User approves → Signed consent artefact
Consent Manager → Data Provider (consent artefact)
Data Provider → Data Consumer (data, encrypted)
[Consent Manager is DATA-BLIND — never sees the data]
```

The Consent Manager model is critical for privacy: a single entity manages consent on behalf of the user but never has access to the underlying data. This is the DEPA architecture (India's Account Aggregator framework).

### Consent Properties

| Property | Meaning |
|---------|---------|
| **Granular** | Purpose-specific and data-field-specific, not blanket |
| **Revocable** | User can withdraw consent at any time |
| **Time-bound** | Consent has defined start, end, and access frequency |
| **Digitally signed** | Non-repudiable; immutable record |
| **Auditable** | Every data access logged against the consent artefact |
| **Purpose-limited** | Data can only be used for the consented purpose |

## PKI for Verifiable Credentials

PKI infrastructure is the trust anchor for Verifiable Credential ecosystems:

```
Issuer's Certificate (from national PKI CA)
         ↓
Issuer signs VC with private key (HSM-backed)
         ↓
Verifier checks signature against Issuer's public key
         ↓
Issuer's public key validated against CA certificate
         ↓
CA certificate traced to trusted Root CA
         ↓
Trust established: VC is authentic and unmodified
```

**X.509 + VC interoperability**: Many VC implementations support X.509 certificates as trust anchors alongside DID-based trust — allowing countries to leverage existing PKI infrastructure for their VC programs without requiring blockchain or new registries.

**EBSI (European Blockchain Service Infrastructure)**: Uses PKI principles to anchor VC trust — stores public keys related to DIDs and Trusted Issuer Registry, validating signatures in credential verification flows.

## Country PKI Models

| Country | Model | Key Features |
|---------|-------|-------------|
| **EU** | eIDAS Trusted Lists | Multi-level; national QTSPs; mutual recognition across EU |
| **India** | CCA-anchored hierarchy | RCAI (Root CA of India) → Licensed CAs; IT Act recognition; Aadhaar eSign |
| **Brazil** | ICP-Brasil | Government-operated Root CA; 3-year certificate validity; widely used in finance |
| **Estonia** | SK ID Solutions | Embedded in ID card chip; used for authentication and QES |
| **Saudi Arabia** | NCDC | National PKI under government; used for government services |
| **South Korea** | GPKI | Government PKI + NPKI (commercial); mandatory for financial transactions |

## Implementation Guide

### Setting Up a Country PKI for DPI

```
Phase 1: Root CA Establishment
  - Procure HSM (FIPS 140-2 Level 3+)
  - Generate Root CA keypair offline in ceremony
  - Create self-signed Root CA certificate
  - Publish Root CA certificate publicly
  - Store in physically secured offline facility

Phase 2: Intermediate CA
  - Generate Intermediate CA keypair in HSM
  - Root CA signs Intermediate CA certificate (ceremony)
  - Intermediate CA goes online for day-to-day operations
  - OCSP responder deployed for revocation checking

Phase 3: Trust Service Providers
  - Define regulatory framework for TSPs/QTSPs
  - Audit and approve licensed CAs
  - Publish national Trusted List
  - Integrate with eIDAS or regional equivalents for cross-border recognition

Phase 4: eSign Integration
  - Expose eSign as open API on identity rail
  - Integrate with national ID authentication
  - Publish API specification and sandbox
  - Onboard government services and financial institutions
```

### eSign API (Conceptual)

```http
POST /v1/esign/sign
Authorization: Bearer <service-token>
Content-Type: application/json

{
  "document_hash": "sha256-hash-of-document",
  "document_type": "application/pdf",
  "auth_mode": "OTP",
  "citizen_id": "national-id-number",
  "purpose": "Loan agreement signing",
  "transaction_id": "uuid"
}

Response:
{
  "signed_document_hash": "...",
  "signature": "base64-pkcs7-signature",
  "certificate": "base64-x509-certificate",
  "timestamp": "RFC3161-timestamp",
  "txn_id": "uuid"
}
```

## Key Standards & References

| Standard | Description |
|---------|-------------|
| **X.509 v3** | Certificate format standard (ITU-T) |
| **PKCS#11** | HSM interface standard |
| **PKCS#12** | Certificate + key bundle format |
| **RFC 5280** | Internet X.509 PKI Certificate profile |
| **RFC 6960** | Online Certificate Status Protocol (OCSP) |
| **RFC 3161** | Time-Stamp Protocol (TSP) |
| **eIDAS (EU 910/2014)** | EU electronic trust framework |
| **eIDAS 2.0 (EU 2024/1183)** | Updated EU framework with digital wallets |
| **PAdES (ETSI EN 319 100)** | PDF Advanced Electronic Signatures |
| **XAdES (ETSI EN 319 132)** | XML Advanced Electronic Signatures |
| **W3C Data Integrity** | Cryptographic proofs for Linked Data / VCs |

- **CDPI Technical Note**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/electronic-signature-pki-and-trust-infra
- **World Bank PKI DPI Policy Note (2024)**: https://documents1.worldbank.org/curated/en/099122424085511614/pdf/P178616-e83a12fb-9119-430d-8afe-c813dfc7fe0f.pdf
- **eIDAS**: https://digital-strategy.ec.europa.eu/en/policies/eidas-regulation
- **India CCA**: https://cca.gov.in
