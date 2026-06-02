---
name: dpi-digital-id
description: Digital Identity as DPI — foundational/functional ID, eKYC, authentication protocols, privacy design, biometrics. Use when designing identity systems or evaluating ID-layer DPI decisions.
---

# Digital Identity as DPI

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id

## The Identity Rail

Digital identity is the first DPI layer because it answers the foundational question: **who is this person, and can I verify that claim digitally?** When this question can be answered by any authorized service through an open API, it becomes a shared rail that the entire economy can build on.

The identity rail has two distinct capabilities:
1. **Authentication**: "Is this person who they claim to be?" (a binary yes/no, in real time)
2. **Attribute sharing (eKYC)**: "What do we know about this person?" (demographic/biometric data, returned with consent)

Both capabilities should be exposed as open APIs under a clear authorization framework.

## Identity in the DPI Stack

Digital identity is the first and foundational layer of DPI. Without reliable identity, payments cannot be attributed, data cannot be consented, and services cannot be personalized or targeted.

**Two categories of digital ID:**

| Type | Purpose | Examples |
|------|---------|---------|
| **Foundational ID** | Establishes who someone is; civil registration basis | Aadhaar, MOSIP, Cédula, NID |
| **Functional ID** | Identity for a specific program or service | Voter ID, Passport, Tax ID, Health ID |

The DPI vision is **federated, interoperable identity**: multiple functional IDs linked to or derived from a foundational ID, each usable across systems without requiring the foundational ID to be shared.

## Core Identity Lifecycle

```
Enrollment → Deduplication → Credential Issuance → Authentication → Authorization → Revocation
```

### Enrollment
- Collection of demographic data (name, DOB, address, gender)
- Collection of biometric data (fingerprint, iris, face — optional)
- Document verification (birth certificate, existing ID)
- Field enrollment vs. self-enrollment vs. trusted introducer models

### Deduplication
- Ensures one-person-one-record (prevents duplicate registrations)
- Biometric deduplication: 1:N matching against full database
- Demographic deduplication: probabilistic matching
- Critical for welfare delivery, voter rolls, financial inclusion

### Credential Issuance
- Physical card, digital credential, or QR code
- Virtual ID numbers (masked, tokenized)
- Verifiable Credentials (W3C standard) increasingly preferred
- Credential validity, expiry, and revocation management

### Authentication
- **Something you know**: PIN, password, passphrase
- **Something you have**: OTP, hardware token, SIM
- **Something you are**: biometric (fingerprint, face, iris)
- Multi-factor combinations for higher assurance

### Authorization
- Consent-based: user explicitly allows data access
- Role-based: defined access for specific service providers
- Purpose-limited: data shared only for declared purpose

## Identity Assurance Levels (IAL)

Based on NIST SP 800-63 and ITU-T X.1254:

| Level | Assurance | Typical Use |
|-------|-----------|------------|
| IAL1 | Self-asserted | Low-stakes online services |
| IAL2 | Remote identity-proofed | Financial services, e-government |
| IAL3 | In-person proofed | High-value transactions, passports |

## Authentication Assurance Levels (AAL)

| Level | Factors Required | Examples |
|-------|-----------------|---------|
| AAL1 | Single factor | Password, biometric |
| AAL2 | Multi-factor | Password + OTP |
| AAL3 | Hardware-bound MFA | FIDO2 key + biometric |

## Key Technologies

### Biometrics in Identity
- **Fingerprint**: Most widely deployed; inexpensive sensors; aging/damage degrades quality
- **Iris**: High accuracy; expensive; used in Aadhaar
- **Face**: Convenient; privacy concerns; works with phones
- **Voice**: Accessibility use cases; spoofing risk
- Best practice: multi-modal with liveness detection

### Biometric Deduplication Systems
- ABIS (Automated Biometric Identification System)
- Typical 1:N match at 1.3B records (Aadhaar scale)
- Vendors: Idemia, Innovatrics, NEC, Aware

### Authentication Protocols
- **OTP over SMS/TOTP**: Widely used, SIM-swap vulnerable
- **FIDO2/WebAuthn**: Phishing-resistant, hardware-bound
- **OpenID Connect (OIDC)**: Standard federation protocol (used by e-Signet)
- **SAML 2.0**: Enterprise federation
- **Mobile-based biometric**: Fingerprint/face on device, result attested

### eKYC (Electronic Know Your Customer)
- Identity verification at scale for financial onboarding
- Aadhaar eKYC: consent → demographic data returned to requester
- Reduces cost from ~$23 to ~$0.15 per verification
- Privacy: only required fields returned, not full record

---

## CDPI Digital ID Technical Notes

CDPI identifies four key capabilities that a Digital ID system must expose as DPI rails. Each is a distinct API capability enabling a different class of service.

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id

**CDPI's core design position**: Enable **multimodal authentication** (OTP, QR codes, biometrics, face recognition) and **offline authentication** to solve for inclusion at scale — increasing the coverage of services while protecting privacy through minimal disclosure.

A Digital ID for DPI must be:
- **Human and system readable** — key identity attributes in QR code format
- **Digitally signed** by the issuing authority — verifiable signatures
- **Accessible online** (via APIs) and **offline** (via digital wallets or signed artefacts)
- **Capable of authenticating** users in both online and offline modes

---

### Capability 1: ID-Auth (Online Authentication)

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/id-auth

**ID authentication** is the process by which an ID number (or tokenized version), combined with one or more authentication factors, is used to verify a citizen's identity in real time.

**What it returns**: A binary yes/no — "this person is who they claim to be" — **without sharing demographic data**. Authentication does not equal data sharing.

**Authentication factors supported:**

| Factor | Mode | Notes |
|--------|------|-------|
| **OTP** | Online | Sent to registered mobile number; widely accessible |
| **Biometric (fingerprint/iris)** | Online | High assurance; requires sensor at point of service |
| **Face match** | Online / Mobile | Photo on file matched to live face via app |
| **QR code** | Offline | Digitally signed artefact verified locally |
| **PIN / Password** | Online | Low assurance; suitable for low-stakes services |
| **FIDO2 / hardware key** | Online | Highest assurance; phishing-resistant |

**Two-factor authentication (2FA)**: For higher-assurance services, combine factors — e.g., OTP + biometric, or PIN + face match.

**Offline local ID-Auth**: The ID authority issues a **digitally signed artefact** (JSON/XML file or QR code) containing the ID or ID alias, photo, and minimal information required for validation. This artefact can be verified locally without an internet connection, enabling authentication in offline or low-connectivity environments.

```
Online ID-Auth flow:
Service Provider sends auth request (ID number + factor)
        ↓ HTTPS API call
ID Authority
        ↓ verifies factor against enrolled record
Returns: YES / NO (binary — no personal data returned)
        ↓
Service Provider proceeds or denies
```

---

### Capability 2: eKYC / Identity Profile Sharing

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/ekyc-identity-profile-sharing

**eKYC** (Electronic Know Your Customer) is the consent-based release of an individual's identity profile — demographic attributes and photograph — from the ID authority to a service provider.

**How it works:**
```
1. Service provider requests eKYC (specifies required fields)
2. ID authority authenticates the user (OTP, biometric, or face)
3. User explicitly consents to release of specified attributes
4. ID authority returns a KYC packet (encrypted, signed)
   containing only the consented, minimum required fields
5. Service provider decrypts and uses the data
```

**Privacy by design in eKYC:**
- Only the **minimum required fields** are returned — not the full identity record
- User must **actively consent** to each eKYC release
- Each release is **logged and auditable** — user can see who accessed their data
- The service provider cannot store eKYC data beyond the declared purpose
- The ID authority does not see what service the data is used for

**Typical KYC packet contents** (configurable by service need):
```json
{
  "name": "Full Name",
  "dob": "1990-01-15",
  "gender": "M",
  "address": "...",
  "photo": "base64-encoded-photo",
  "phone": "masked-number",
  "email": "masked@email.com"
}
```

**eKYC use cases:**

| Service | Fields requested |
|---------|----------------|
| Bank account opening | Name, DOB, address, photo |
| SIM card registration | Name, address, photo |
| Government benefit enrollment | Name, DOB, gender, address |
| Insurance KYC | Name, DOB, photo |
| Age verification | Age ≥ 18 (boolean) — no DOB shared |

**Cost impact**: eKYC reduces verification cost from ~$23 (in-person, paper) to ~$0.15 (API call).

---

### Capability 3: Face Authentication

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/face-authentication

**Face authentication** uses the registered photo in the identity database to verify a person's identity via 1:1 face matching — without requiring fingerprint sensors or iris scanners.

**Why it matters for inclusion**: Face matching on a standard mobile phone creates **non-linear scale** in service delivery. Any smartphone becomes an authentication terminal — no specialized biometric hardware required.

**How it works:**
```
1. Service provider app captures live photo of person
2. Liveness detection check (prevents photo spoofing)
3. App calls ID-Auth API with live face image + ID number
4. ID system performs 1:1 match against enrolled photo
5. Returns: match score + YES/NO decision
```

**Offline face authentication**: If the ID is presented as a QR code or digital credential, the registered photo is embedded in the signed artefact. The service provider app performs 1:1 face matching **locally** — no network required.

```
QR code on ID card (digitally signed, contains photo)
        ↓ scan
Service provider app decodes QR
        ↓
App captures live face → 1:1 match against QR photo
        ↓ local comparison (no API call)
Returns: match / no match
```

**Privacy considerations:**
- Face images should NOT be retained by service providers after verification
- Liveness detection is mandatory — static photo spoofing is a real threat
- Face authentication is lower assurance than fingerprint/iris for high-stakes services

---

### Capability 4: QR Code for Offline ID

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/qr-code-for-offline-id

**Offline ID verification** enables service delivery in areas without internet connectivity, using a **digitally signed QR code** embedded in a physical or digital ID.

**What the QR code contains** (digitally signed by the ID authority):
- ID number or alias (not the root ID — a tokenized version)
- Name and minimal demographic attributes
- Registered photo (compressed)
- Digital signature (verifiable against authority's public key)
- Expiry date of the artefact

**Verification flow (fully offline):**
```
1. Individual presents physical card or digital ID (with QR code)
2. Service provider scans QR code with standard app
3. App verifies digital signature against ID authority's public key
   (public key pre-loaded or cached — no internet required)
4. App optionally performs 1:1 face match (live face vs. QR photo)
5. Service delivered — no network connection needed
```

**Security properties:**
- **Tamper-evident**: Any modification to the QR data breaks the digital signature
- **Offline-verifiable**: Public key verification requires no API call
- **Privacy-preserving**: Only the data in the QR is revealed — database not queried
- **Time-limited**: QR artefacts should have expiry; periodic refresh prevents indefinite offline use

**CDPI framing**: *"Offline local ID authentication can happen if the ID authority offers a digitally signed artefact... containing the foundational/functional ID or ID alias, photo, and any other minimal information required for validation."*

---

### Capability 5: Single Sign-On (SSO)

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/single-sign-on-sso

**Identity-based SSO** allows an individual to log on to any government or private portal using their government-issued ID — similar to "Login with Google" but anchored in the national identity system.

**How it works (OIDC-based):**
```
1. User visits any participating service provider
2. User clicks "Login with National ID"
3. Service redirects to national OIDC gateway (e.g. e-Signet)
4. User authenticates with any supported factor (OTP, biometric, face, QR)
5. OIDC gateway returns ID token + access token to service
6. Service can optionally request eKYC attributes (with consent)
7. User is logged in — no separate account creation needed
```

**SSO authentication modes** (all should be supported):
- OTP-based (SMS to registered mobile)
- Biometric (fingerprint/iris via sensor)
- Wallet-linked (from a digital ID wallet app)
- QR code scan (for offline-capable flows)
- Face match (on mobile)

**Benefits of identity-based SSO:**
- Citizens use one credential for all government and participating private services
- Eliminates fragmented username/password management across departments
- Service providers do not need to build their own identity systems
- Authentication events are auditable through the national system

**Standard**: Built on **OpenID Connect (OIDC)** — the same protocol used by consumer identity providers. Any OIDC-compliant application can integrate without custom development.

> See the **dpi-e-signet** skill for the open-source OIDC gateway implementation (MOSIP ecosystem).

---

## Privacy-Preserving Identity Design

### Principles
1. **Minimal disclosure**: Return only what is needed (age > 18 vs. full DOB)
2. **Tokenization**: Use service-specific virtual IDs, not the root ID
3. **Consent logging**: Every authentication/data share logged, auditable
4. **Purpose limitation**: Data used only for stated purpose
5. **Data localization**: Biometrics stored only at enrollment point

### Zero-Knowledge Proofs in Identity
- Prove attribute (age > 18, resident of region) without revealing value
- Used in some Verifiable Credential implementations
- inji/OpenID4VC implementations increasingly support selective disclosure

## Open-Source Identity Platforms

Multiple open-source platforms exist for implementing foundational identity DPI. Selection should be based on neutral criteria (see dpi-country-technical-recommendation skill), not prescription:

### MOSIP (Modular Open Source Identity Platform)
- **License**: MPL 2.0 | **GitHub**: https://github.com/mosip
- API-first, cloud-native microservices architecture
- Covers: enrollment, biometric deduplication, authentication, resident services
- Includes e-Signet (OIDC gateway) and Inji (VC wallet)
- Deployed in Africa, Asia, and the Pacific

### OpenCRVS (Civil Registration)
- **License**: MPL 2.0 | **GitHub**: https://github.com/opencrvs/opencrvs-core
- Focused on civil registration and vital statistics (births, deaths, marriages)
- Often paired with foundational ID for end-to-end identity lifecycle

### Generic Identity Architecture
Any identity system used as DPI must expose:
```
External → Enrollment/Registration
                    ↓
           Deduplication/ID Repository
                    ↓
           Authentication API (real-time)
                    ↓
           OIDC Gateway (e-Signet or equivalent)
                    ↓
           Service Providers (any licensed party)
```

## Identity Ecosystem Design

### Trust Framework
- Who can be an Authentication Service Provider (ASP)?
- Who can be an eKYC requester?
- What data can each partner access?
- Liability and compliance obligations
- Audit and grievance mechanisms

### Federated vs. Centralized
| Approach | Pros | Cons |
|----------|------|------|
| **Centralized** | Simpler, strong deduplication | Single point of failure, surveillance risk |
| **Federated** | Resilient, privacy-friendly | Deduplication harder |
| **Self-Sovereign (SSI)** | User control, no central DB | Enrollment bootstrap problem, recovery complexity |

**Reconciling federation with deduplication**: The DPI principle of federation (no single database of everything) does not mean deduplication is impossible. Federated approaches include:
- **Token-based crosswalk**: A central deduplication service sees only tokenized biometric templates, not raw PII; records stay federated
- **Secure Multi-Party Computation (MPC) ABIS**: Multiple institutions run biometric matching without any party seeing the full dataset
- **Probabilistic crosswalk registries**: Shared index of hashed identity attributes for deduplication, not a central identity store
- **Sector-level functional registries**: Each sector maintains its own population sub-set; cross-sector matching done by verified consent, not bulk dedup

Most production DPI systems use a practical middle: a **national deduplication index** (biometric hash only, no demographics) operating at enrollment time, with federated data storage for all ongoing services.

## Common Implementation Challenges

1. **Last-mile enrollment**: Remote/nomadic/illiterate populations
2. **Exception handling**: People without documents, identical twins, injured fingers
3. **Connectivity**: Registration in offline areas
4. **Biometric exclusion**: Elderly, laborers with degraded prints
5. **Update mechanisms**: Name changes, address changes, death registration
6. **Governance**: Who controls the system, oversight bodies
7. **Interoperability**: Integration with existing functional IDs

## Standards & References

- **CDPI Digital ID technical notes**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id
- **CDPI ID-Auth**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/id-auth
- **CDPI eKYC**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/ekyc-identity-profile-sharing
- **CDPI Face Authentication**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/face-authentication
- **CDPI QR Code Offline ID**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/qr-code-for-offline-id
- **CDPI SSO**: https://docs.cdpi.dev/technical-notes/digital-ids-and-electronic-registries/digital-id/single-sign-on-sso
- **ISO/IEC 24745**: Biometric information protection
- **NIST SP 800-63**: Digital Identity Guidelines
- **ITU-T X.1254**: Entity authentication assurance framework
- **W3C DID**: Decentralized Identifiers
- **W3C VC**: Verifiable Credentials Data Model
- **OpenID Connect**: Identity federation protocol
- **FIDO2/WebAuthn**: Phishing-resistant authentication
