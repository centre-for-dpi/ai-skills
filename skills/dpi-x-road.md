---
name: dpi-x-road
description: X-Road — Estonia's federated data exchange, Security Server, trust federation, G2G/G2B interoperability. Use when designing federated data exchange or evaluating X-Road for DPI.
---

# X-Road — Federated Data Exchange for DPI

## What is X-Road?

X-Road is an open-source software solution that provides secure, standardized data exchange between organizations (government, private sector). It was developed in Estonia in 2001, has underpinned Estonia's digital government since 2002, and is now used by 30+ countries.

**X-Road as DPI**: X-Road is the canonical implementation of the "data exchange layer" in Estonia's DPI stack — the infrastructure that allows ministries, banks, hospitals, and businesses to share data securely and auditably without building bilateral integrations.

- **GitHub**: https://github.com/nordic-institute/X-Road
- **Maintained by**: Nordic Institute for Interoperability Solutions (NIIS) — a partnership between Estonia and Finland
- **License**: MIT

## Core Architecture

### Key Components

```
Organisation A                         Organisation B
────────────                           ────────────
Service Client                         Service Provider
      ↕                                      ↕
Security Server A ←────────────────→ Security Server B
      ↕                                      ↕
Central Server (configuration + OCSP)
      ↕
Management API / Admin UI
      ↕
Monitoring / OPMONITOR (optional)
```

### Security Server
The Security Server is the core X-Road component deployed by each participant:
- Handles all inbound and outbound message routing
- Enforces TLS and message-level signing/encryption
- Logs all transactions (non-repudiable audit trail)
- Validates certificates and member authorization
- Translates between REST/SOAP and X-Road message protocol

### Central Server
Operated by the X-Road governing authority (e.g., RIA in Estonia):
- Manages member registration (who can participate)
- Distributes configuration to all Security Servers
- Maintains the global trust anchor (CA certificates)
- Runs OCSP for certificate status checking

### Message Flow
```
1. Service Client (App A) sends request to own Security Server
2. Security Server A:
   - Validates client authentication
   - Looks up Service Provider's Security Server address
   - Signs the request with A's key
   - Sends over HTTPS to Security Server B

3. Security Server B:
   - Validates signature and TLS certificate of A
   - Checks authorization (is A allowed to call this service?)
   - Forwards to Service Provider (App B)
   - Signs and returns response
   - Logs the transaction

4. Security Server A:
   - Validates B's signature
   - Returns response to Service Client (App A)
   - Logs the transaction
```

## X-Road Message Protocol

### REST API Support (X-Road 6.x+)

X-Road wraps REST calls with X-Road-specific headers:

```http
GET /api/v1/persons/EE12345678901
X-Road-Client: EE/GOV/10001234/client1
X-Road-Service: EE/GOV/20002345/POPULATION-SERVICE/v1
X-Road-Id: unique-message-id
X-Road-UserId: jane.doe
X-Road-Issue: case-1234
Authorization: Bearer <token>  # Passed through to service provider
```

### SOAP Support (Legacy)
X-Road originally used SOAP/XML messaging with X-Road-specific headers embedded in SOAP header:
```xml
<SOAP-ENV:Header>
  <xrd:client id:objectType="MEMBER">
    <id:xRoadInstance>EE</id:xRoadInstance>
    <id:memberClass>GOV</id:memberClass>
    <id:memberCode>12345678</id:memberCode>
  </xrd:client>
  <xrd:service id:objectType="SERVICE">
    <id:xRoadInstance>EE</id:xRoadInstance>
    <id:memberClass>GOV</id:memberClass>
    <id:memberCode>87654321</id:memberCode>
    <id:serviceCode>GetPersonData</id:serviceCode>
    <id:serviceVersion>v1</id:serviceVersion>
  </xrd:service>
  <xrd:id>unique-request-id</xrd:id>
</SOAP-ENV:Header>
```

## Security Model

### Authentication and Authorization

**Member authentication:**
- Each X-Road member (organization) has an X-Road member certificate (from a trusted CA)
- Security Servers authenticate each other via mTLS using these certificates
- Member certificates are managed via Central Server

**Service authorization:**
- Each service has an ACL (Access Control List) of allowed clients
- ACLs configured in Security Server admin UI or via Management REST API
- Fine-grained: can allow specific subsystems/clients

**Message integrity:**
- Every request and response is signed by the sender's Security Server
- Signatures stored in Security Server logs (non-repudiable)
- Audit logs: immutable, hash-chained

### Certificate Infrastructure
```
X-Road Root CA (operated by governing authority)
    ↓
    ├── Member Authentication CA
    │       → Certificates for each Security Server
    │
    └── Signing CA
            → Message signing certificates for each member
```

## Deployment

### Security Server Deployment Options

**Traditional (VM/bare metal):**
```bash
# Ubuntu 20.04/22.04 installation
curl https://packages.niis.org/asc/niis-archive-keyring-1.0.gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/niis.gpg
echo "deb https://packages.niis.org/xroad/stable/7/ focal main" > /etc/apt/sources.list.d/xroad.list
apt-get update && apt-get install xroad-securityserver
```

**Kubernetes (containerized):**
```yaml
# X-Road Security Server Sidecar pattern
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: my-service          # The actual service
        image: myorg/myservice:1.0
        ports:
        - containerPort: 8080
      - name: xroad-proxy         # X-Road Security Server as sidecar
        image: niis/xroad-security-server-sidecar:7.3
        ports:
        - containerPort: 5500     # X-Road listener
        - containerPort: 5577     # X-Road management
        env:
        - name: XROAD_TOKEN_PIN
          valueFrom:
            secretKeyRef: { name: xroad-secret, key: pin }
```

### Configuration via Management REST API
```bash
# Add service to Security Server
curl -X POST "https://ss1.example.gov:4000/api/v1/clients/EE:GOV:12345:client/service-descriptions" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://myservice.example.gov/api",
    "type": "REST",
    "rest_service_code": "GetPersonData"
  }'

# Add access right for a client
curl -X POST "https://ss1.example.gov:4000/api/v1/services/EE:GOV:12345:client:GetPersonData/access-rights" \
  -H "Authorization: Bearer <token>" \
  -d '{"items": [{"id": "EE:GOV:87654:consumer-client"}]}'
```

## X-Road Across Countries

### Estonia (Originator)
- 2001: First version deployed
- All government agencies connected (~150+)
- 1B+ transactions per year
- Used for: tax data, population registry, health records, business registry, land records
- Citizens can see their own X-Road queries via e-Estonia portal

### Finland
- Deployed since 2017 (Suomi.fi Data Exchange Layer)
- Used for national data exchange between government agencies
- NIIS is an EE-FI joint venture

### Iceland, Norway, Sweden
- Nordic countries adopted X-Road via Nordic-Baltic interoperability agreement

### Other Countries
- **Palestine**: HLEG Data Exchange Platform
- **Faroe Islands**: Government data exchange
- **Uganda, Rwanda**: Exploring X-Road for government integration
- **Namibia, Mozambique**: DPI projects using X-Road

### Cross-Border X-Road (Nordic-Baltic)
```
Estonia X-Road ←────────────────────→ Finland X-Road
                    Trust bridge
                    (bilateral CA trust)
                    
Allows: EE government services to query FI services and vice versa
Use case: Cross-border social benefits, professional recognition
```

## X-Road vs. Other Data Exchange Approaches

| Dimension | X-Road | Account Aggregator (DEPA) | Direct API |
|-----------|--------|--------------------------|-----------|
| **Use case** | G2G, G2B, B2B any data | Financial data (FI-focused) | Any |
| **Consent** | Organizational ACL | User-level consent | Application-defined |
| **Data path** | Through Security Servers | End-to-end encrypted (AA not in data path) | Direct |
| **Audit** | Immutable per-transaction log | Consent artifact log | None by default |
| **Discovery** | Central catalog | Central directory | None |
| **Privacy** | Org-level; user data in transit | User consent at field level | None |
| **Setup cost** | Medium (Security Server per org) | Medium (FIP/FIU registration) | Low |
| **Best for** | Government interoperability | Financial consent data | Simple integrations |

## X-Road for DPI Data Exchange

### Recommended Architecture (DPI context)
```
Ministry of Health ──Security Server──→ X-Road ←──Security Server── Hospital A
Ministry of Finance ──Security Server──→ X-Road ←──Security Server── Bank XYZ
Revenue Authority ──Security Server──→ X-Road ←──Security Server── Employer Payroll

Citizens access their own data via:
Resident Portal → API Gateway → X-Road → Relevant agency
```

### Integration with Identity Layer
```
Service Provider calls X-Road service:
  X-Road request includes X-Road-UserId header (national ID or token)
  
Service Provider (Agency B) receives query:
  Validates X-Road-Client (is calling agency authorized?)
  Validates X-Road-UserId (is user authenticated at calling agency?)
  Returns data scoped to the identified user
```

## Monitoring and Analytics

### X-Road Operational Monitoring (OPMONITOR)
- Collects message-level stats from all Security Servers
- Central dashboard: transaction counts, latency, errors
- No message content (only metadata: sender, receiver, service, timestamp)
- Used for: capacity planning, SLA monitoring, usage analytics

### Log Formats
```json
{
  "messageId": "uuid",
  "timestamp": "2024-01-01T12:00:00Z",
  "clientId": "EE/GOV/12345/client",
  "serviceId": "EE/GOV/67890/service/v1",
  "userId": "EE10101010001",
  "requestSize": 1024,
  "responseSize": 2048,
  "succeeded": true,
  "duration": 145
}
```

## Key Resources

- **GitHub**: https://github.com/nordic-institute/X-Road
- **Documentation**: https://docs.x-road.global/
- **NIIS**: https://www.niis.org/
- **X-Road Community**: https://x-road.global/
- **Estonian X-Road**: https://www.ria.ee/en/state-information-system/x-road
