---
name: dpi-inji
description: Inji Wallet, Inji Certify, Inji Verify — MOSIP's VC ecosystem, OpenID4VCI, offline QR. Use when implementing digital credential wallets or integrating with the Inji stack.
---

# Inji — MOSIP's Verifiable Credentials Ecosystem

## Official Documentation

> **Reference**: https://docs.inji.io/

Always refer to https://docs.inji.io/ for the most current Inji documentation, API references, deployment guides, and release notes. The content below reflects Inji's architecture and APIs as of mid-2024.

## What is Inji?

Inji is MOSIP's open-source suite for Verifiable Credentials (VCs). It enables:
- **Issuance** of VCs from any authoritative source
- **Storage** of VCs in a secure mobile wallet
- **Sharing** of VCs via QR code, NFC/BLE, or online presentation
- **Verification** of VCs by relying parties

The name "Inji" means "ginger" in Tamil — reflecting its spicy, transformative nature.

## Inji Product Suite

### 1. Inji Wallet (Mobile App)
Open-source mobile wallet for holding and sharing credentials.

**Platforms**: Android, iOS (React Native)
**GitHub**: https://github.com/mosip/inji-wallet

**Key Features**:
- Download credentials from issuers via OpenID4VCI
- Store credentials encrypted on device
- Share via QR code (offline, no internet required)
- Share via BLE (Bluetooth Low Energy) for proximity
- Share via online presentation (OpenID4VP)
- Face verification before sharing (liveness check)
- Multi-credential support (ID, health, education, etc.)
- Multi-language support

### 2. Inji Web
Browser-based credential wallet for desktop access.

**GitHub**: https://github.com/mosip/inji-web

**Key Features**:
- Download MOSIP-issued VCs
- View credential details
- Generate QR codes for offline sharing
- Works alongside mobile wallet

### 3. Inji Certify
Credential issuance server — connects to any data source and issues VCs.

**GitHub**: https://github.com/mosip/inji-certify

**Key Features**:
- OpenID4VCI compliant credential issuance
- Pluggable data sources (MOSIP, custom DBs, APIs)
- VC rendering templates (PDF, display)
- Credential schema management
- Key management for signing
- Supports SD-JWT VC format
- Supports W3C VC JSON-LD format

### 4. Inji Verify
Credential verification library and web component.

**GitHub**: https://github.com/mosip/inji-verify

**Key Features**:
- QR code scanning and decoding
- VC signature verification
- Status/revocation check
- Embeddable React component
- Works offline (no network for signature verification)

## Inji Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Inji Wallet (Mobile)                  │
│  ┌───────────┐  ┌───────────┐  ┌──────────────────────┐ │
│  │ Credential│  │  Face ID  │  │  Share Manager       │ │
│  │  Store    │  │ Verifier  │  │  (QR/BLE/Online)     │ │
│  │(Encrypted)│  │           │  │                      │ │
│  └───────────┘  └───────────┘  └──────────────────────┘ │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              OpenID4VCI Client                      │ │
│  │         (Credential Download)                       │ │
│  └─────────────────────────────────────────────────────┘ │
└────────────────────────┬────────────────────────────────┘
                         │ OpenID4VCI
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    Inji Certify                          │
│  ┌───────────┐  ┌───────────────┐  ┌─────────────────┐ │
│  │  Auth     │  │  VC Builder   │  │  Key Manager    │ │
│  │ (e-Signet)│  │  (Signing)    │  │  (HSM/Vault)    │ │
│  └───────────┘  └───────────────┘  └─────────────────┘ │
│  ┌─────────────────────────────────────────────────────┐ │
│  │             Data Connector (pluggable)              │ │
│  │   MOSIP ID Repo | Custom DB | External API          │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

## Credential Issuance Flow (OpenID4VCI)

### Step 1: Wallet discovers issuer
```
GET https://issuer.example.com/.well-known/openid-credential-issuer

Response:
{
  "credential_issuer": "https://issuer.example.com",
  "credential_endpoint": "https://issuer.example.com/credential",
  "authorization_server": "https://esignet.example.com",
  "credentials_supported": [
    {
      "id": "MOSIPNationalID",
      "format": "ldp_vc",
      "types": ["VerifiableCredential", "MOSIPVerifiableCredential"],
      "display": [{"name": "National ID", "locale": "en"}]
    }
  ]
}
```

### Step 2: Authorization (via e-Signet)
```
Wallet redirects to:
GET https://esignet.example.com/authorize
  ?client_id=inji-wallet
  &response_type=code
  &scope=openid+MOSIPNationalID
  &redirect_uri=inji://callback
  &code_challenge=...
  &code_challenge_method=S256

User authenticates (OTP, biometric, etc.)

Redirect back:
inji://callback?code=auth-code-xyz
```

### Step 3: Token Exchange
```http
POST https://esignet.example.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=auth-code-xyz
&redirect_uri=inji://callback
&code_verifier=...

Response:
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "c_nonce": "random-nonce-for-proof",
  "c_nonce_expires_in": 86400
}
```

### Step 4: Credential Request
```http
POST https://issuer.example.com/credential
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "format": "ldp_vc",
  "types": ["VerifiableCredential", "MOSIPVerifiableCredential"],
  "proof": {
    "proof_type": "jwt",
    "jwt": "eyJ..."  // Wallet proves key possession; contains c_nonce
  }
}

Response:
{
  "credential": {
    "@context": ["https://www.w3.org/2018/credentials/v1"],
    "type": ["VerifiableCredential", "MOSIPVerifiableCredential"],
    "issuer": "https://issuer.example.com",
    "issuanceDate": "2024-01-01T00:00:00Z",
    "credentialSubject": {
      "id": "did:jwk:...",
      "UIN": "XXXXXXXX1234",
      "fullName": [{"language": "eng", "value": "John Doe"}],
      "dateOfBirth": "1990-01-01",
      "gender": "Male",
      "photo": "base64-photo-data"
    },
    "proof": { "type": "Ed25519Signature2020", ... }
  }
}
```

## Offline Sharing via QR Code

Inji encodes a compressed, signed VC in a QR code for offline verification:

### QR Code Content
```
1. VC is serialized to JSON
2. JSON is compressed (zlib)
3. Compressed bytes are base45-encoded
4. Encoded string is placed in QR code

# Format indicator prefix
VP1:base45-encoded-compressed-vc-data
```

### Verifier Process (Inji Verify)
```javascript
// 1. Scan QR code
const qrContent = scanQR();

// 2. Decode
const compressed = base45.decode(qrContent.replace('VP1:', ''));
const vcJson = zlib.inflate(compressed);
const vc = JSON.parse(vcJson);

// 3. Resolve issuer DID
const didDocument = await resolveDID(vc.issuer);
const publicKey = extractPublicKey(didDocument);

// 4. Verify signature
const isValid = await verifySignature(vc, publicKey);

// 5. Check expiry
const isExpired = new Date(vc.expirationDate) < new Date();

// 6. Display result
return { valid: isValid && !isExpired, credential: vc.credentialSubject };
```

## BLE (Bluetooth Low Energy) Sharing

Inji supports proximity credential sharing via BLE, useful for:
- Offline verification where QR scanning is impractical
- High-volume verification (immigration, events)
- Low-light environments

### BLE Flow
```
Holder (Inji Wallet)          Verifier (Inji Verify App)
       ↓                              ↓
  Advertise BLE             Scan for Inji BLE devices
  Service UUID               Select device
       ↓                              ↓
  Accept connection ←————————— Connect
       ↓                              ↓
  Transfer VC ——————————————→  Receive VC
       ↓                              ↓
  Confirm transfer             Verify signature
                                Display result
```

## Inji Certify — Custom Credential Issuance

### Deploying Inji Certify for Custom Credentials

```yaml
# docker-compose for Inji Certify
version: '3.8'
services:
  inji-certify:
    image: mosip/inji-certify:0.9.0
    ports:
      - "8088:8088"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/inji_certify
      - MOSIP_ESIGNET_HOST=https://esignet.example.com
      - MOSIP_IDA_AUTH_URL=https://ida.example.com/idaAuthentication/v1
    volumes:
      - ./config:/home/mosip/config
```

### Custom Data Connector
```java
// Implement DataProviderPlugin interface for custom data source
@Component
public class CustomEducationCredentialPlugin implements DataProviderPlugin {
    
    @Override
    public Map<String, Object> fetchData(String subjectId, String credentialType) {
        // Fetch from your database/API
        Student student = studentRepository.findByNationalId(subjectId);
        
        return Map.of(
            "studentId", student.getId(),
            "degreeTitle", student.getDegree(),
            "institution", student.getUniversity(),
            "graduationDate", student.getGradDate().toString()
        );
    }
    
    @Override
    public String getCredentialType() {
        return "UniversityDegreeCredential";
    }
}
```

### Credential Schema Definition
```json
{
  "credentialSchema": {
    "id": "https://issuer.example.com/schemas/degree/v1",
    "type": "JsonSchema"
  },
  "credentialSubject": {
    "type": "object",
    "required": ["studentId", "degreeTitle", "institution"],
    "properties": {
      "studentId": { "type": "string" },
      "degreeTitle": { "type": "string" },
      "institution": { "type": "string" },
      "graduationDate": { "type": "string", "format": "date" }
    }
  }
}
```

## Inji Integration with MOSIP

When deployed as part of a full MOSIP stack:

```
MOSIP Registration → ID Repository → Inji Certify
                                          ↓
                                  Issues MOSIP VC
                                          ↓
                                   Inji Wallet
                                  (resident downloads)
                                          ↓
                               Share with service provider
                                          ↓
                                   Inji Verify
                                  (verifier scans)
                                          ↓
                            Verified → Service granted
```

### Resident Portal Integration
Users can download their VC through:
1. MOSIP Resident Portal (web) → download to Inji Wallet via QR
2. Inji Wallet → authenticate → download directly

## Trust Registry Integration

Inji Verify needs to know which issuers are trusted:

```json
// Trust Registry entry for an issuer
{
  "issuerId": "did:web:mosip.example.gov",
  "issuerName": "Department of Civil Registration",
  "credentialTypes": ["MOSIPVerifiableCredential", "BirthCertificate"],
  "publicKeyEndpoint": "https://mosip.example.gov/.well-known/did.json",
  "validFrom": "2023-01-01",
  "status": "ACTIVE"
}
```

## Configuration Reference

### Inji Wallet Key Config Options
```javascript
// config.json / environment
{
  "ESIGNET_HOST": "https://esignet.example.com",
  "CREDENTIAL_REGISTRY_EDIT": true,
  "ALLOWED_CREDENTIAL_TYPES": [
    "MOSIPVerifiableCredential",
    "InsuranceCertificate", 
    "HealthCredential"
  ],
  "FACE_AUTH_ENABLED": true,
  "BLE_ENABLED": true,
  "QR_CODE_VERSION": "VP1"
}
```

## Deployment Checklist

When deploying the full Inji stack:
- [ ] Inji Certify deployed with correct data connector
- [ ] e-Signet configured as authorization server for Inji Certify
- [ ] Credential schemas registered and published
- [ ] Issuer DID document published at well-known URL
- [ ] Trust registry populated with issuer entry
- [ ] Inji Wallet app built with environment-specific config
- [ ] QR code rendering tested at different sizes
- [ ] BLE sharing tested on Android & iOS
- [ ] Offline verification tested without network
- [ ] Revocation mechanism configured (Status List 2021)
- [ ] Key rotation procedure documented

## Choosing a VC Issuance Platform

Inji Certify is one of several open-source VC issuance platforms. Use the following guide:

| Use case | Recommended |
|---------|-------------|
| MOSIP-integrated identity stack | **Inji Certify** — native integration with MOSIP ID and e-Signet |
| Multi-tenant, multi-country credential platform | **CREDEBL** — enterprise multi-tenancy, Hyperledger Credo, AnonCreds ZKP |
| ZKP-based selective disclosure (privacy-preserving) | **CREDEBL** with AnonCreds |
| Stand-alone non-MOSIP environment | Either — CREDEBL has fewer MOSIP dependencies |
| Offline QR + BLE holder wallet | **Inji Wallet** (any issuer can target it via OpenID4VCI) |

> See the **dpi-credebl** skill for a full CREDEBL architecture reference and detailed comparison.

## Resources

- **GitHub Org**: https://github.com/mosip (all Inji repos)
- **Documentation**: https://docs.mosip.io/inji
- **Inji Wallet**: https://github.com/mosip/inji-wallet
- **Inji Certify**: https://github.com/mosip/inji-certify
- **Inji Verify**: https://github.com/mosip/inji-verify
- **Inji Web**: https://github.com/mosip/inji-web
