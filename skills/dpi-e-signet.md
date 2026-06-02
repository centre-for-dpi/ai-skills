---
name: dpi-e-signet
description: e-Signet — MOSIP's OIDC authentication gateway, eKYC, auth plugins, QR login, SP integration. Use when implementing identity verification or integrating a service with MOSIP.
---

# e-Signet — OIDC Identity Gateway for DPI

## What is e-Signet?

e-Signet is an open-source OpenID Connect (OIDC) provider developed by MOSIP that enables:
- **Login with national ID**: Citizens authenticate using their national identity
- **eKYC**: Service providers receive verified identity attributes
- **Consent management**: Users control which attributes are shared
- **Multi-factor authentication**: OTP, biometric, QR login
- **Cross-DPI federation**: Works as an identity bridge across DPI systems

**GitHub**: https://github.com/mosip/esignet  
**Documentation**: https://docs.mosip.io/e-signet

e-Signet is designed to decouple the authentication experience from the underlying identity system — any identity system (MOSIP, custom ABIS, smart card, etc.) can be plugged in.

## Architecture Overview

```
┌─────────────────┐     OIDC      ┌─────────────────┐
│  Service         │ ←──────────→ │    e-Signet      │
│  Provider (RP)  │               │  (OIDC Provider) │
│  (Bank, Portal) │               │                  │
└─────────────────┘               │  ┌────────────┐  │
                                  │  │  Auth Plugin│  │
         ┌────────────────────────→  │  Interface │  │
         │                        │  └─────┬──────┘  │
         │                        └────────│──────────┘
         │                                 │
         ▼                                 ▼
┌─────────────────┐               ┌─────────────────┐
│  User's Browser  │               │  Identity System │
│  (Consent UI)    │               │  (MOSIP IDA,     │
│                  │               │   custom, etc.)  │
└─────────────────┘               └─────────────────┘
```

## OpenID Connect Flows Supported

### Authorization Code Flow (Web Applications)
```
1. RP → GET /authorize?
         client_id=rp-client
         &response_type=code
         &scope=openid+profile+email+mosip_id
         &redirect_uri=https://rp.example.com/callback
         &state=random-state
         &nonce=random-nonce

2. e-Signet presents login UI to user
3. User authenticates (OTP, biometric, etc.)
4. User reviews and approves consent

5. e-Signet → Redirect to:
   https://rp.example.com/callback?code=AUTH_CODE&state=random-state

6. RP → POST /oauth/token
   Body: grant_type=authorization_code
         &code=AUTH_CODE
         &redirect_uri=...
         &client_id=...
         &client_secret=... (or client_assertion for PKCE)

7. e-Signet → 
   {
     "access_token": "eyJ...",
     "id_token": "eyJ...",  // Contains user identity claims
     "token_type": "Bearer",
     "expires_in": 3600
   }

8. RP → GET /userinfo
   Authorization: Bearer ACCESS_TOKEN

9. e-Signet → 
   {
     "sub": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "name": "John Doe",
     "phone_number": "+91XXXXXXXXXX",
     "email": "john@example.com",
     "birthdate": "1990-01-01",
     "gender": "male",
     "address": { "country": "IN" }
   }
```

### PKCE Flow (Mobile/SPA Applications)
```
# Wallet/app generates:
code_verifier = crypto.randomBytes(32).toString('base64url')
code_challenge = sha256(code_verifier).toString('base64url')

# Authorization request includes:
&code_challenge=CODE_CHALLENGE
&code_challenge_method=S256

# Token request includes:
code_verifier=CODE_VERIFIER  (instead of client_secret)
```

## e-Signet Configuration

### Service Provider (Relying Party) Registration

```json
{
  "client_id": "my-service-portal",
  "client_name": "My Government Service Portal",
  "redirect_uris": [
    "https://service.example.gov/auth/callback"
  ],
  "grant_types": ["authorization_code"],
  "client_auth_methods": ["private_key_jwt"],
  "logo_uri": "https://service.example.gov/logo.png",
  "public_key": {
    "kty": "RSA",
    "n": "...",
    "e": "AQAB",
    "kid": "rp-key-1"
  },
  "user_claims": [
    "name", "phone_number", "email", "birthdate", "gender", "address"
  ],
  "auth_context_refs": ["mosip:idp:acr:static-code"],
  "claims_locales": ["eng", "hin"]
}
```

### Authentication Context Reference (ACR) Levels

e-Signet maps ACR values to authentication strength:

| ACR Value | Authentication Method |
|-----------|----------------------|
| `mosip:idp:acr:generated-code` | OTP sent to registered mobile |
| `mosip:idp:acr:static-code` | PIN/password |
| `mosip:idp:acr:biometrics` | Fingerprint/iris/face |
| `mosip:idp:acr:linked-wallet` | Mobile wallet QR |
| `mosip:idp:acr:knowledge` | Knowledge-based questions |

Service providers specify required ACR:
```
GET /authorize?
  ...
  &acr_values=mosip:idp:acr:biometrics+mosip:idp:acr:generated-code
```

## Authentication Plugin Interface

e-Signet uses a plugin architecture to connect to any identity backend:

### Plugin Interface (Java)
```java
// Core authentication plugin interface
public interface AuthenticationWrapper {
    
    /**
     * Authenticate user against the identity system
     */
    KycAuthResult doKycAuth(String relyingPartyId, 
                             String clientId, 
                             KycAuthDto kycAuthDto) 
        throws KycAuthException;
    
    /**
     * Exchange auth token for KYC data
     */
    KycExchangeResult doKycExchange(String relyingPartyId,
                                     String clientId,
                                     KycExchangeDto kycExchangeDto) 
        throws KycExchangeException;
    
    /**
     * Send OTP to registered channel
     */
    SendOtpResult sendOtp(String relyingPartyId,
                           String clientId,
                           SendOtpDto sendOtpDto) 
        throws SendOtpException;
    
    /**
     * Get supported authentication factors
     */
    List<KycSigningCertificateData> getAllKycSigningCertificates();
}
```

### MOSIP IDA Plugin Implementation
```java
@Component
public class MosipIdaAuthenticationWrapper implements AuthenticationWrapper {
    
    @Override
    public KycAuthResult doKycAuth(String rpId, String clientId, 
                                    KycAuthDto kycAuthDto) {
        // Call MOSIP IDA authentication API
        AuthRequest authRequest = buildMosipAuthRequest(kycAuthDto);
        AuthResponse response = mosipIdaClient.authenticate(authRequest);
        
        return KycAuthResult.builder()
            .kycToken(response.getToken())
            .partnerSpecificId(response.getPartnerVid())
            .build();
    }
}
```

### Custom Authentication Plugin (Example: Smart Card)
```java
@Component  
public class SmartCardAuthPlugin implements AuthenticationWrapper {
    
    @Override
    public KycAuthResult doKycAuth(String rpId, String clientId,
                                    KycAuthDto dto) {
        // Verify smart card certificate signature
        X509Certificate cardCert = parseCertificate(dto.getCertificateData());
        boolean valid = verifyCertificateChain(cardCert, trustedRoot);
        
        if (valid) {
            String pid = extractPID(cardCert);
            Map<String, Object> kycData = fetchFromRegistry(pid);
            return buildKycAuthResult(pid, kycData);
        }
        throw new KycAuthException("CERT_VALIDATION_FAILED");
    }
}
```

## Consent Management in e-Signet

### Consent UI Flow
```
1. Service Provider requests: scope=openid+name+phone_number+birthdate
2. e-Signet authentication succeeds
3. e-Signet shows consent page:
   ┌─────────────────────────────────────────────┐
   │  MyBank wants to access:                    │
   │  ✓ Your full name                           │
   │  ✓ Your phone number                        │
   │  ✓ Your date of birth                       │
   │                                             │
   │  This will be used for: Account opening     │
   │                                             │
   │  [Allow]              [Deny]                │
   └─────────────────────────────────────────────┘
4. User clicks Allow → tokens issued with consented claims
5. User clicks Deny → error returned to SP
```

### Consent Storage
- User consent recorded with timestamp
- Per-RP, per-scope consent history maintained
- Users can revoke consent via resident portal
- Consent artifact optionally DEPA-compatible

## QR Code Login (Linked Wallet)

e-Signet supports passwordless login via mobile wallet:

```
1. Browser shows QR code at login page
2. User opens Inji Wallet (or any OIDC wallet)
3. Wallet scans QR → receives login request
4. Wallet authenticates user (face/pin)
5. Wallet sends signed auth response
6. Browser session activates
```

### QR Login Technical Flow
```
Browser → GET /linked-authorization/link-transaction
        ← { linkTransactionId, link-code, expiry }
        
Browser shows QR with: esignetlink://...?link-code=LINK_CODE

Wallet → POST /linked-authorization/link-transaction
         { link-code }
        ← { linked-transaction-id, auth-factors, client-name, logo-uri, ... }

Wallet authenticates user locally (biometric/pin)

Wallet → POST /linked-authorization/link-auth-code
         { linked-transaction-id, auth-response }
        ← { nonce }

Browser polls GET /linked-authorization/link-status
            ← { linkStatus: "LINKED" }
            
Browser → Normal OIDC auth code exchange using link-auth-code
```

## eKYC Response Claims

e-Signet supports these OIDC claims (configurable per deployment):

### Standard OIDC Claims
```json
{
  "sub": "pairwise-identifier-per-rp",
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe",
  "middle_name": "",
  "preferred_username": "johndoe",
  "email": "john.doe@example.com",
  "email_verified": true,
  "phone_number": "+919876543210",
  "phone_number_verified": true,
  "birthdate": "1990-01-01",
  "gender": "male",
  "address": {
    "formatted": "123 Main St, Mumbai, MH 400001",
    "street_address": "123 Main St",
    "locality": "Mumbai",
    "region": "Maharashtra",
    "postal_code": "400001",
    "country": "IN"
  },
  "picture": "data:image/jpeg;base64,..."
}
```

### MOSIP-Specific Claims
```json
{
  "individual_id": "XXXXXXXX1234",   // Masked ID
  "uid_token": "pairwise-uid-token", // Per-RP token
  "resident_status": "ACTIVATED"
}
```

## Deployment

### Docker Compose (Development)
```yaml
version: '3.8'
services:
  esignet:
    image: mosip/esignet:1.4.0
    ports:
      - "8088:8088"
    environment:
      - SPRING_PROFILES_ACTIVE=default
      - MOSIP_ESIGNET_CRYPTO_CORE_BASE_URL=http://keymanager:8088
      - MOSIP_ESIGNET_DATABASE_URL=jdbc:postgresql://postgres:5432/esignet
      - AUTH_WRAPPER_IMPL=io.mosip.esignet.plugin.mosipid.MosipIdaAuthWrapper
    volumes:
      - ./config:/home/mosip/config
      
  esignet-ui:
    image: mosip/esignet-default-ui:1.4.0
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_ESIGNET_API_URL=http://esignet:8088
```

### Kubernetes (Production)
```bash
helm repo add mosip https://mosip.github.io/mosip-helm
helm install esignet mosip/esignet \
  --namespace mosip-esignet \
  --create-namespace \
  -f esignet-values.yaml
```

### key esignet-values.yaml settings
```yaml
image:
  repository: mosip/esignet
  tag: 1.4.0

esignet:
  database:
    host: postgres.mosip.svc
    port: 5432
    dbname: esignet
  
  authWrapper:
    impl: io.mosip.esignet.plugin.mosipid.MosipIdaAuthWrapper
  
  oidc:
    issuer: https://esignet.example.gov
    tokenExpiry: 3600
    idTokenExpiry: 3600
    
  ui:
    brand:
      logo: https://example.gov/logo.png
      name: National Digital Identity
```

## Integration Checklist

When integrating a service with e-Signet:

- [ ] Register RP client via admin API or UI
- [ ] Configure allowed redirect URIs
- [ ] Configure required scopes/claims
- [ ] Set ACR requirements for authentication strength
- [ ] Implement PKCE (for mobile/SPA) or client_secret_jwt (for server)
- [ ] Parse and validate ID token (signature, iss, aud, exp, nonce)
- [ ] Fetch userinfo with access token
- [ ] Handle consent revocation scenarios
- [ ] Implement token refresh if using long-lived sessions
- [ ] Test with multiple authentication factors
- [ ] Test error scenarios (user denies consent, session timeout)

## Key Endpoints

| Endpoint | Description |
|----------|-------------|
| `/.well-known/openid-configuration` | OIDC discovery document |
| `/authorize` | Authorization endpoint |
| `/oauth/token` | Token endpoint |
| `/userinfo` | UserInfo endpoint |
| `/jwks.json` | JSON Web Key Set (public keys) |
| `/client-mgmt/oidc-client` | Client registration (admin) |
| `/linked-authorization/*` | QR/wallet-based login endpoints |
| `/oidc-ui-config` | UI configuration for custom branding |
