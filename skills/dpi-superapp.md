---
name: dpi-superapp
description: DPI Superapps — unified citizen apps, mini-app architecture, SSO, open agency APIs, federated data sharing. Use when designing citizen-facing DPI applications or evaluating superapp models.
---

# DPI Superapp

> **CDPI Reference**: https://docs.cdpi.dev/the-dpi-wiki/building-a-superapp-a-three-step-guide-to-apply-dpi-thinking

## Critical Distinction: A SuperApp is NOT DPI

> *"A SuperApp, or any application, is not Digital Public Infrastructure by itself. Aggregating government services in one place is a digital government solution, but it does not constitute DPI."* — CDPI

This is the most important framing to get right:

| What it is | What it is NOT |
|-----------|---------------|
| Application layer sitting on top of DPI | DPI itself |
| One possible interface to DPI rails | The only way to access DPI |
| A convenience layer for citizens | A replacement for open, interoperable infrastructure |

**Why the distinction matters**: If a government builds a superapp without opening the underlying APIs to third parties, it creates a digital government portal — useful, but not DPI. DPI requires that any actor (private sector, NGO, other government agencies) can build their own interface on top of the same open rails.

A SuperApp built *on* DPI — where the underlying identity, payments, and data exchange APIs are open — has far greater potential than a closed government portal.

---

## Three-Layer Architecture (CDPI Model)

```
┌─────────────────────────────────────────────────┐
│  Application Layer                               │
│  (Superapp UI, private apps, third-party apps)  │
│  — Any actor can build here —                   │
├─────────────────────────────────────────────────┤
│  Service Layer                                   │
│  (Individual ministry/agency APIs and services) │
│  — Agencies expose their services as APIs —     │
├─────────────────────────────────────────────────┤
│  Infrastructure Layer (THIS is the DPI)          │
│  Open, shared rails:                            │
│  Identity · Payments · Data Exchange            │
│  — Open standards, shared public goods —        │
└─────────────────────────────────────────────────┘
```

**The DPI principle**: The infrastructure layer must be open to ALL applications — not just the government superapp. Once agency APIs are open, any player (private company, civil society, other government) can design the best interface for their context, support specific languages, or integrate AI.

---

## CDPI Three-Step Guide to DPI-Enabling a SuperApp

### Step 1: Integrate National ID SSO

Enable citizens to log in once across all services using the national identity system.

```
Citizen opens superapp
        ↓
Redirected to national ID provider (e.g. e-Signet / OIDC gateway)
        ↓ single authentication
Superapp receives identity token
        ↓
All agency services accept the same token — no re-authentication
```

**What this unlocks**: Citizens don't need separate passwords for each ministry. The identity assertion is trusted by all agencies because it comes from the authoritative national ID system.

**Design requirement**: National ID provider must support OIDC so any app (not just the superapp) can integrate.

### Step 2: Open Agency APIs to Third Parties

This is the step that transforms a government portal into DPI-enabling infrastructure.

```
Ministry A exposes its services as APIs
Ministry B exposes its services as APIs
Ministry C exposes its services as APIs
        ↓
API Gateway / Government API Registry (discoverable, documented)
        ↓
Any authorised actor can call these APIs:
  - The government superapp  ✓
  - A private sector health app  ✓
  - An NGO social protection portal  ✓
  - A state government dashboard  ✓
  - An AI agent  ✓
```

**The DPI multiplier**: When APIs are open, private sector innovation fills gaps the government cannot. A fintech company might build the best G2P disbursement interface for rural users; a health startup might integrate immunisation records with teleconsultation booking.

**Governance challenge**: Different ministry departments may resist cooperating with a central entity that controls the superapp. Opening APIs to all actors (not just the superapp) reduces this political friction — each agency retains ownership of its service.

### Step 3: Enable Federated Data Sharing

Each agency owns its data. The superapp orchestrates consent-based sharing between agencies without centralising the data.

```
Citizen requests a composite service (e.g. income-based benefit)
        ↓ superapp requests consent
Citizen grants consent to share data from:
  - Revenue dept (income data)  → verified via identity token
  - Health dept (medical eligibility)  → FHIR record shared
        ↓ each agency shares only required fields (min disclosure)
Welfare dept receives verified composite claim
        ↓
Benefit approved without citizen having to gather paper documents
```

**Data ownership model**: Revenue dept owns income data. Health dept owns health records. No central data warehouse. The superapp is a consent orchestrator, not a data store.

**Technical pattern**: This is the Data Exchange DPI layer — see the `dpi-data-exchange` and `dpi-data-sharing-models` skills for DEPA / consent manager implementation.

---

## What is a DPI Superapp?

A DPI Superapp is a unified digital interface — mobile or web — that aggregates multiple DPI-powered services into a single, coherent citizen experience. Rather than requiring citizens to use 10 different apps for 10 different government services, a superapp provides one front-door to the DPI ecosystem.

**Key distinction from commercial superapps (WeChat, Grab):**
- Built on open DPI rails (not proprietary)
- Citizen-controlled data (not platform-captured)
- Non-extractive business model (public good)
- Interoperable (citizens aren't locked into one app)

## Design Philosophy

### The DPI Superapp Principle
The superapp should be "thin glass" over DPI infrastructure:
- **Orchestrate, not own**: Calls DPI APIs, doesn't replicate data
- **Consent, not capture**: User controls what the app knows
- **Open, not closed**: Mini-apps can be added by any registered provider
- **Accessible, not elite**: Works for all, including low-literacy, elderly, rural

### User-Centric Design Principles
1. **Single sign-on**: Authenticate once via e-Signet/national ID, access all services
2. **Unified notifications**: All government messages in one inbox
3. **Document wallet**: All credentials/documents in one place
4. **Payment integration**: Pay for any service without leaving the app
5. **Grievance resolution**: One place to file and track complaints
6. **Proactive delivery**: Services find citizens, not citizens finding services

## Architecture Patterns

### Mini-App Architecture (WeChat/Alipay style for DPI)

```
┌─────────────────────────────────────────────────────────┐
│                    Superapp Shell                        │
│  ┌─────────┐  ┌──────────┐  ┌───────────┐  ┌────────┐ │
│  │ ID/Auth │  │ Document │  │ Payments  │  │ Notif. │ │
│  │ Module  │  │ Wallet   │  │ Module    │  │ Center │ │
│  └─────────┘  └──────────┘  └───────────┘  └────────┘ │
├─────────────────────────────────────────────────────────┤
│                    Mini-App Framework                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ Ration   │  │ Health   │  │ Pension  │  │ +Add   │ │
│  │ Card     │  │ Records  │  │ Status   │  │ More   │ │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘ │
├─────────────────────────────────────────────────────────┤
│                    DPI API Layer                         │
│   e-Signet | ID Auth | UPI/Payments | AA | DigiLocker   │
└─────────────────────────────────────────────────────────┘
```

### Reference Architecture

```
Mobile App (React Native / Flutter)
           ↓
     API Gateway / BFF (Backend for Frontend)
           ↓
  ┌────────────────────────────────────────┐
  │           Core Superapp Services       │
  │  Auth  │ Profile │ Notif │ Documents  │
  └────────────────────────────────────────┘
           ↓
  ┌────────────────────────────────────────┐
  │        DPI Integration Layer           │
  │  e-Signet │ UPI │ AA │ DigiLocker     │
  └────────────────────────────────────────┘
           ↓
  ┌────────────────────────────────────────┐
  │       Ministry/Department APIs         │
  │  Health│ Education │ Revenue │ Welfare │
  └────────────────────────────────────────┘
```

## Global DPI Superapp Examples

### India: DigiLocker + Umang
- **DigiLocker**: Document wallet; 6B+ documents from 1800+ issuers
- **Umang** (Unified Mobile Application for New-age Governance): 1200+ government services in one app

### Singapore: Singpass App
- Login for 2000+ government/private sector services
- MyInfo: Consented sharing of profile data
- Digital signatures (contracts, documents)
- Document storage (PDFs from government)

### UAE: UAE PASS
- National digital identity for government + private services
- Digital wallet for licenses, credentials
- Single login for all 9,000+ UAE government services

### Estonia: Smart-ID + eesti.ee
- Smart-ID: Mobile-based signing and authentication
- eesti.ee: Citizen portal with all government services

### Brazil: Gov.br
- Unified federal government app
- Login using CPF (tax ID) or biometrics
- Access to 4,000+ government services
- Social security, COVID vaccination, driving license

### Philippines: PhilSys / PSA Apps
- PhilSys Number as universal ID
- Plans for unified service delivery app

## Technical Implementation

### Frontend: React Native for Cross-Platform
```javascript
// Superapp navigation structure
const Tab = createBottomTabNavigator();
const Stack = createStackNavigator();

function SuperappNavigator() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Documents" component={DocumentWallet} />
      <Tab.Screen name="Payments" component={PaymentsHub} />
      <Tab.Screen name="Services" component={ServicesDirectory} />
      <Tab.Screen name="Profile" component={CitizenProfile} />
    </Tab.Navigator>
  );
}

// DPI Authentication using e-Signet OIDC
async function loginWithNationalID() {
  const authResult = await openIDConnect.authorize({
    issuer: ESIGNET_URL,
    clientId: SUPERAPP_CLIENT_ID,
    redirectUrl: 'superapp://auth/callback',
    scopes: ['openid', 'profile', 'phone'],
    additionalParameters: {
      acr_values: 'mosip:idp:acr:generated-code'
    }
  });
  return authResult;
}
```

### Backend for Frontend (BFF) Pattern
```typescript
// NestJS BFF for superapp
@Controller('citizen')
export class CitizenController {
  
  @Get('dashboard')
  @UseGuards(EsignetAuthGuard)
  async getDashboard(@CurrentUser() user: CitizenUser) {
    // Orchestrate multiple DPI calls in parallel
    const [documents, payments, notifications] = await Promise.all([
      this.digilockerService.getDocuments(user.sub),
      this.paymentService.getRecentTransactions(user.uid),
      this.notificationService.getUnread(user.sub)
    ]);
    
    return { documents, payments, notifications };
  }
  
  @Post('payments/initiate')
  @UseGuards(EsignetAuthGuard)
  async initiatePayment(
    @CurrentUser() user: CitizenUser,
    @Body() paymentDto: InitiatePaymentDto
  ) {
    // Verify identity before payment
    const authToken = await this.esignetService.getToken(user.sub);
    return this.upiService.initiatePayment(paymentDto, authToken);
  }
}
```

### Document Wallet Integration
```typescript
// DigiLocker / Inji credential integration
class DocumentWalletService {
  
  async getIssuedDocuments(userId: string): Promise<Document[]> {
    // Fetch from DigiLocker API with consent
    const digilockerDocs = await this.digilocker.getDocuments(userId);
    
    // Fetch VCs from Inji Certify
    const vcs = await this.injiCertify.getCredentials(userId);
    
    // Normalize to unified document format
    return [...digilockerDocs, ...vcs].map(this.normalizeDocument);
  }
  
  async shareDocument(docId: string, verifierId: string): Promise<string> {
    const doc = await this.getDocument(docId);
    const shareToken = await this.generateQRShare(doc, verifierId);
    return shareToken; // QR code data or deep link
  }
}
```

## Service Integration Patterns

### Service Registry / App Store
```json
// Service catalog entry
{
  "serviceId": "ration-card-management",
  "serviceName": {
    "en": "Ration Card Management",
    "hi": "राशन कार्ड प्रबंधन"
  },
  "ministry": "Department of Food & Public Distribution",
  "category": "welfare",
  "icon": "https://service.gov.in/ration/icon.png",
  "deepLink": "https://ration.service.gov.in",
  "miniAppUrl": "https://ration.service.gov.in/mini-app",
  "requiredAuth": true,
  "requiredClaims": ["name", "ration_card_number"],
  "platforms": ["android", "ios", "web"],
  "states": ["all"]
}
```

### Deep Linking for Services
```
superapp://service/ration-card
superapp://document/driving-license
superapp://payment/electricity-bill?billerCode=MSEB&accountId=1234567
superapp://profile/update-address
```

## Notification Architecture

### Unified Notification Center
```typescript
// Notification from any DPI-connected service
interface CitizenNotification {
  id: string;
  source: string;         // "ration-dept" | "income-tax" | "health"
  title: string;
  body: string;
  category: "alert" | "reminder" | "action_required" | "info";
  deepLink?: string;      // Where to go when tapped
  expiresAt?: Date;
  priority: "high" | "normal" | "low";
  requiresAction?: boolean;
}

// Delivery channels
type Channel = "push" | "sms" | "email" | "in_app";

// Priority → Channel mapping
const channelPolicy: Record<string, Channel[]> = {
  "action_required": ["push", "sms", "in_app"],
  "alert": ["push", "in_app"],
  "reminder": ["in_app", "push"],
  "info": ["in_app"]
};
```

## Payments Hub

### Multi-Rail Payment Integration
```typescript
class PaymentsHub {
  
  async payBill(bill: BillPayment): Promise<PaymentResult> {
    // Detect optimal payment rail based on context
    const rail = this.selectRail(bill);
    
    switch (rail) {
      case 'upi':
        return this.upiService.pay({
          vpa: bill.merchantVpa,
          amount: bill.amount,
          note: bill.description
        });
        
      case 'netbanking':
        return this.netbankingService.pay(bill);
        
      case 'wallet':
        return this.walletService.debit(bill);
    }
  }
  
  async receiveG2PTransfer(): Promise<void> {
    // Listen for government-to-person payment webhook
    // DBT, scholarship, pension, etc.
  }
}
```

## Accessibility Requirements

### WCAG & Inclusion Standards
- **Screen reader**: Full VoiceOver/TalkBack support
- **Font scaling**: Support up to 200% font size
- **Color contrast**: WCAG AA minimum (4.5:1)
- **Language**: Auto-detect + manual override
- **Offline mode**: Core features work without internet

### Low-Literacy / Assisted Access
```
Progressive complexity approach:
Tier 1: Icon-only navigation + voice instructions
Tier 2: Simplified text + icons
Tier 3: Full text + advanced features

Voice support:
- IVR fallback for feature phones
- Text-to-speech for all content
- Voice command for navigation
```

## Security Architecture

### Token Management
```typescript
// Short-lived tokens for sensitive operations
const TOKEN_CONFIG = {
  idToken: { ttl: '1h', renewable: true },
  paymentToken: { ttl: '5m', renewable: false, pinRequired: true },
  documentShare: { ttl: '10m', oneTime: true },
  dashboardToken: { ttl: '8h', renewable: true }
};

// Re-auth for sensitive actions
async function requireReauth(action: SensitiveAction): Promise<void> {
  if (action.requiresBiometric) {
    await BiometricAuth.authenticate({ reason: action.description });
  }
  if (action.requiresPin) {
    const pin = await PinDialog.show();
    await validatePin(pin);
  }
}
```

### Jailbreak / Root Detection
```typescript
// Security checks at app launch
const securityCheck = await SecurityModule.check();
if (securityCheck.isRooted || securityCheck.isJailbroken) {
  Alert.show("Security Warning", "App cannot run on rooted devices");
  // Limit functionality but don't block completely (for testing/banking)
}
```

## Governance and Ecosystem

### Mini-App Onboarding Policy
- Registered government entities only (or licensed private sector)
- API compliance certification required
- Security audit for each mini-app
- Privacy policy review
- Accessibility compliance check
- Performance SLA commitment

### Data Governance
- Superapp is a "data passthrough", not a data store
- No behavioral profiling of citizens
- No cross-service data linkage without consent
- Audit log of all DPI API calls made by app
- Annual transparency report published

## Key Performance Metrics

| Metric | Target |
|--------|--------|
| App launch time | < 2 seconds |
| Login (national ID) | < 5 seconds |
| Service directory load | < 1 second |
| Payment completion | < 10 seconds |
| Document download | < 3 seconds |
| App size | < 30 MB |
| Crash-free sessions | > 99.5% |
| Accessibility score | WCAG AA |
