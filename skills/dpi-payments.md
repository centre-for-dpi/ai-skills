---
name: dpi-payments
description: Payments as DPI — instant payment rails, interoperability, ISO 20022, mobile money, G2P, CBDC. Use when designing payment infrastructure, advising on payment DPI, or evaluating interoperability.
---

# Payments as DPI

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-payment-networks

## The Payment Rail

Interoperable instant payment infrastructure is the second DPI layer. It answers: **can this person send and receive value digitally, to and from anyone on the same rail?**

When payment infrastructure is designed as a public rail:
- Any licensed institution can participate (no exclusive club)
- Any-to-any transfers are possible regardless of which institution a person uses
- Cost approaches zero because the rail is a shared cost, not a profit center
- Government can disburse benefits directly on the same rail used for private commerce
- The private sector builds payment products competing on service, not on rail access

**The key design test**: Can two competing payment apps, built by different private companies, both route payments through the same infrastructure? If yes, it is a rail. If one app owns the infrastructure, it is a platform.

## Landmark Payments DPI Systems

### UPI (Unified Payments Interface) — India
- **Launched**: 2016 by NPCI (National Payments Corporation of India)
- **Volume**: 13+ billion transactions/month (2024)
- **Architecture**: Federated; 300+ banks and payment apps participate
- **Addressing**: VPA (Virtual Payment Address) like user@bank
- **Settlement**: Real-time gross settlement via RBI
- **APIs**: NPCI-specified XML/JSON over HTTPS
- **Key innovation**: 24/7/365, any-to-any, mobile-first, merchant QR codes

### PIX — Brazil
- **Launched**: 2020 by Banco Central do Brasil
- **Volume**: 4+ billion transactions/month; 150M+ registered users
- **Architecture**: DICT (directory service) + PSP-to-PSP direct settlement
- **Addressing**: CPF/CNPJ, phone, email, random key
- **Key innovation**: Mandatory for all financial institutions with 500K+ accounts

### FedNow / RTP — United States
- **FedNow** (Fed, 2023): Real-time 24/7 settlement for banks
- **RTP** (The Clearing House, 2017): Private sector instant payments
- Adoption slower due to fragmented banking landscape

### Faster Payments — UK
- **Launched**: 2008
- **Volume**: 4+ billion transactions/year
- **Architecture**: Central infrastructure (Pay.UK); New Payments Architecture (NPA) in progress
- **Key innovation**: One of world's first real-time payment systems

### GhIPSS — Ghana
- **Ghana Interbank Payment and Settlement Systems**
- Operates GhQR (QR payments), GIP (instant pay), GHIPSS Instant Pay
- Mobile money interoperability with banks

### mVISA, Mastercard Send, etc.
- Scheme-operated interoperability layers
- Proprietary but widely adopted in emerging markets

## Architecture of Instant Payment Systems

### Key Actors

```
Payer → PSP (Payer's bank/wallet)
                ↓
         Payment Switch / Clearing
                ↓
PSP (Payee's bank/wallet) → Payee

Central Components:
- Directory/DICT: Resolve alias → account
- Switch: Route & clear transactions
- RTGS/Settlement: Final money movement
- NPCL/Fraud: Monitoring and limits
```

### Settlement Models

| Model | Description | Example |
|-------|-------------|---------|
| **Deferred Net Settlement (DNS)** | Net positions settled in batches | ACH, SWIFT |
| **Real-Time Gross Settlement (RTGS)** | Each transaction settled immediately | CHAPS, Fedwire |
| **Hybrid RTGS** | Intraday net + RTGS | UPI, PIX |
| **PSP-to-PSP Settlement** | Direct interbank after clearing | PIX direct settlement |

### Addressing / Directory Services
- **Alias-based**: Phone, email, national ID mapped to account
- **IBAN/BBAN**: Account number standards
- **VPA**: UPI virtual payment address (user@bank)
- **Proxy Lookup**: Central directory resolves alias before routing

## ISO 20022 — The Global Standard

ISO 20022 is the emerging universal financial messaging standard, replacing legacy formats (SWIFT MT, NACHA, etc.):

### Key Message Sets
- **pacs.008**: Credit transfer (customer payment)
- **pacs.009**: Credit transfer (financial institution)
- **camt.056**: Payment cancellation
- **pain.001**: Customer credit transfer initiation
- **pain.013**: Creditor payment activation
- **remt.001**: Remittance advice

### Richness of ISO 20022 vs Legacy
- Structured data (vs. unstructured narrative)
- Full legal entity names (vs. truncated)
- Purpose codes, structured addresses
- Enables AML/fraud analytics on richer data

### Migration Timeline
- SWIFT CBPR+: 2022-2025 migration
- Fedwire: 2023 onwards
- ECB TARGET2: 2023
- Most national RTGS systems migrating by 2025

## Mobile Money as Payments DPI

### Key Systems
- **M-Pesa** (Kenya/Africa): 50M+ users; first major mobile money success
- **bKash** (Bangladesh): 60M+ users
- **Wave** (West Africa): Low-fee mobile money
- **Airtel Money**, **MTN MoMo**: Pan-Africa mobile money

### Mobile Money Interoperability
- Historical problem: Siloed wallets (M-Pesa can't pay bKash easily)
- Solutions: National switches, GSMA Interoperability API, bilateral agreements
- **Tanzania**: First country to mandate mobile money interoperability
- **Ghana, Senegal, Côte d'Ivoire**: Regional interoperability initiatives

### Mobile Money → DPI Bridge
- Link mobile wallets to bank rails
- Enable G2P (government-to-person) via mobile money
- Use national ID for KYC (if digital ID DPI exists)

## CBDC (Central Bank Digital Currency) and DPI

### Retail CBDC Models
- **Direct CBDC**: Central bank issues directly to public (disintermediation risk)
- **Indirect CBDC**: Banks distribute CBDC (PSP model)
- **Hybrid CBDC**: Central bank holds ledger, PSPs handle distribution

### DPI-Compatible CBDC Design
- CBDC as bearer instrument (privacy-preserving)
- Programmable money for conditional payments (social protection)
- Offline-capable (NFC, hardware wallets)
- Interoperability with existing payment rails

### Active CBDC Projects
- **e-CNY** (China): Largest deployment, 200M+ wallets
- **e-Naira** (Nigeria): First African CBDC
- **DCash** (Eastern Caribbean): Multi-country CBDC
- **Digital Rupee** (India): RBI pilot 2022-
- **Project mBridge**: Cross-border CBDC (BIS, China, UAE, Thailand, HK)

## G2P (Government-to-Person) Payments

### DPI for Social Protection
- **Direct Benefit Transfer (India)**: $300B+ transferred via Aadhaar-linked accounts
- **Bolsa Família (Brazil)**: Cash transfers via PIX
- **NSSF (Kenya)**: Pension via mobile money
- **COVID relief**: Countries with DPI disbursed faster than those without

### G2P Design Requirements
- Last-mile access (rural banking agents, mobile money)
- Grievance redressal for failed transfers
- Deduplication (prevent duplicate beneficiaries)
- Real-time reconciliation
- Offline capability where needed

---

## CDPI Digital Payment Networks Technical Notes

CDPI defines three specific interoperability capabilities that upgrade existing payment infrastructure into a true DPI rail. These are complementary building blocks that can be added incrementally to any payment system.

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-payment-networks

### Building Block 1: Financial Address

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-payment-networks/financial-address

A **financial address** is a human-readable alias that uniquely identifies where a person's money is stored — decoupling the payment destination from the underlying account details.

**The problem it solves**: Traditional payment requires knowing a beneficiary's full account number, bank name, and branch code. This creates friction, errors, and excludes people who cannot remember or share long account strings.

**Financial address examples:**

| Format | Example | System |
|--------|---------|--------|
| Phone number | `+254712345678` | M-Pesa, GhIPSS, PIX |
| National ID | `123-45-6789` | G2P Connect |
| Email | `user@example.com` | PIX, UPI |
| VPA (virtual payment address) | `username@bank` | UPI |
| Random alias | `abc-xyz-123` | PIX random key |

**Financial Address Mapper**: A central directory service that resolves any financial address alias → the underlying store-of-value (bank account, mobile wallet, prepaid card). This is the backbone of any-to-any interoperable payments.

```
Payer sends to: phone@+254712345678
        ↓
Financial Address Mapper (DICT / alias registry)
        ↓ resolves to: Account No. X, Bank Y
        ↓
Payment routed to Account Y
```

**Key properties of a good financial address system:**
- One person → multiple aliases (all resolve to same account)
- Aliases are portable — citizen changes bank, updates mapper, old alias still works
- Privacy-preserving — the mapper reveals the destination to payment systems, not to payers
- Auditable — every resolution logged

**CDPI framing**: *"The combination of 'Bank A/c No., Code, Bank Name + Branch' or 'Card number, Provider, Expiry, CVV' serves as your financial address (where your money stays)."* The financial address system makes this navigable for ordinary people.

---

### Building Block 2: Interoperable QR Code

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-payment-networks/interoperable-qr-code

An **interoperable QR code** allows any payment application to scan a merchant's QR code and complete a payment — regardless of which payment app the merchant uses or which bank/wallet the payer uses.

**The problem it solves**: Without a standard QR, each payment provider issues its own QR — merchants display 5 QR codes, payers can only scan their own provider's code. This fragments the market and excludes smaller providers.

**How interoperable QR works:**
```
Merchant generates / displays one standard QR code
        ↓ customer scans with any payment app
Payment app reads QR payload (standardised format)
        ↓ identifies merchant + amount
App routes payment via national switch
        ↓
Settlement to merchant's account (any bank/wallet)
```

**QR code types:**

| Type | Description | Use case |
|------|-------------|---------|
| **Static QR** | Fixed, reusable; no amount encoded | Small merchants, informal traders |
| **Dynamic QR** | Generated per transaction; amount + reference encoded | Formal merchants, e-commerce, utilities |

**The EMVCo standard**: CDPI recommends **EMVCo QR Code Specification** as the global standard for interoperable merchant-presented QR codes:
- **Payload format**: Tag-Length-Value (TLV) structure
- **Merchant-Presented Mode (MPM)**: Merchant shows QR; customer scans and pays
- **Consumer-Presented Mode (CPM)**: Customer shows QR; merchant terminal scans

**Payload structure (EMVCo MPM):**
```
QR Payload:
  00 - Payload Format Indicator: "01"
  01 - Point of Initiation Method: "11" (static) / "12" (dynamic)
  26-51 - Merchant Account Information (per payment network)
  52 - Merchant Category Code
  53 - Transaction Currency
  54 - Transaction Amount (dynamic QR only)
  58 - Country Code
  59 - Merchant Name
  60 - Merchant City
  62 - Additional Data (reference, terminal ID)
  63 - CRC checksum
```

**National QR implementations built on EMVCo:**

| Country | Standard | Notes |
|---------|---------|-------|
| **India** | BharatQR / UPI QR | Carries Visa, Mastercard, RuPay, UPI in one QR |
| **Thailand** | PromptPay QR | National standard; all wallets and banks |
| **Ghana** | GhQR | GhIPSS national interoperable QR |
| **Ethiopia** | National Bank interoperable QR | Based on EMVCo |
| **Singapore** | SGQR | Single unified QR across all payment schemes |
| **EU** | EPC QR (SEPA) | Mobile-initiated SEPA credit transfers |

**Insertion as "+1 step" to existing rails**: CDPI notes that QR interoperability can be achieved by inserting a QR code specification layer onto existing payment systems — existing wallets and providers adopt the spec, not a new system.

---

### Building Block 3: Interoperable Authentication (P2P / P2M)

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-payment-networks/interoperable-authentication-p2p-p2m

**Interoperable authentication** means the ability to confirm a payment transaction in a **standardised manner** — so that any payer app can authenticate any transaction on any rail, with any level of assurance required.

**CDPI framing**: *"Interoperable, in this context, means standardised. Thus, interoperable authentication means the ability to confirm the transaction in a standardised manner."*

**Authentication layer in the payment switch:**
The authentication layer is handled by the payment switch to **securely capture** private PIN, OTP, and biometric data — and **verify each transaction** before settlement.

**Authentication modes for payments:**

| Mode | Description | Assurance |
|------|-------------|-----------|
| **PIN** | Static secret, entered at POS or in app | Medium |
| **OTP** | One-time password sent to registered phone | Medium |
| **Biometric (device)** | Fingerprint/face on the payer's device | High |
| **Biometric (remote)** | Biometric verified against ID authority (e.g., UIDAI) | High |
| **FIDO2 / passkey** | Hardware-bound, phishing-resistant | Highest |

**P2P vs. P2M authentication:**
- **P2P (Person-to-Person)**: Payer authenticates in their own app; payee is identified by financial address
- **P2M (Person-to-Merchant)**: Payer authenticates in app after scanning merchant QR; merchant is pre-registered and verified

**Step-up authentication**: High-value transactions require stronger authentication:
```
Transaction < threshold → PIN or OTP sufficient
Transaction > threshold → Biometric required
New payee + large amount → Step-up + confirmation delay
```

---

### Building Block 4: G2P Payments — Four Building Blocks

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-payment-networks/g2p-payments

CDPI frames G2P (Government-to-Person) payments as an "overarching architecture as a simple +1 step to your existing systems that ensures interoperability, inclusion, privacy, security, autonomy, and asynchronous adoption by design."

**The four G2P building blocks:**

#### 1. Unique ID / eKYC Layer
- Uniquely identifies and authenticates each beneficiary within seconds
- Eliminates duplicate payments (one-person-one-payment)
- Reduces leakage; saves time and effort on both sides
- Links to national ID or foundational identity system (see dpi-digital-id skill)

#### 2. Financial Address Mapper
- Connects a beneficiary's ID to their preferred store-of-value (bank account, mobile wallet, prepaid card)
- Beneficiary chooses where they receive — government doesn't dictate the bank
- Updated by the beneficiary as they change accounts — government doesn't need to know
- **G2P Connect** specification: interoperable mapper standard for government use

```
Beneficiary register (ID → eligibility) + Financial Address Mapper (ID → account)
                    ↓
Government disbursal system
                    ↓ resolves financial address
Payment rail → beneficiary account (any bank/wallet)
```

#### 3. Last-Mile Agents
- Serve those with limited or no digital literacy
- Biometric-authenticated cash-out at banking agents or kiosks
- Agents serve as human intermediaries — not gatekeepers
- Regulated; transaction limits; real-time monitoring

**Agent banking requirements for G2P:**
- Agent must verify beneficiary biometrically (not rely on card/phone alone)
- Transaction logged immediately; no offline accumulation
- Agent float managed by sponsoring bank

#### 4. Network of Cross-Functional Registries
- Identifies and sorts individuals according to eligibility criteria for different schemes
- Data stays decentralized — each registry owned by its sector (health, agriculture, education)
- G2P Connect APIs allow government disbursement systems to query eligibility across registries without centralizing data
- **Key principle**: *The payment is unified; the eligibility data is federated*

```
Health Registry ─┐
Education Registry ─┤→ G2P Connect eligibility APIs → Disbursement engine → Payment rail
Agriculture Registry ─┘
```

---

## Merchant Payments & QR Codes

### QR Code Standards
- **UPI QR / BharatQR** (India): EMVCo-based; carries multiple schemes in one QR
- **PromptPay QR** (Thailand): National standard; all wallets and banks
- **GhQR** (Ghana): GhIPSS national interoperable QR
- **SGQR** (Singapore): Single QR across all payment schemes
- **EMVCo QR**: Global standard; adopted as basis for national implementations

### Tap-to-Pay / NFC
- ISO 14443, ISO 15693 (contactless standards)
- EMV Contactless (Visa payWave, Mastercard Contactless)
- Increasingly: NFC-based CBDC offline payments

## Cross-Border Payments

### Current Challenges
- **Speed**: Days for correspondent banking
- **Cost**: 6-7% average remittance cost (target: <3%)
- **Transparency**: FX and fees opaque
- **Access**: Unbanked cannot receive international transfers

### DPI Approaches to Cross-Border
- **Nexus Project (BIS)**: Interlinking domestic instant payment systems
- **Project Dunbar**: Multi-CBDC platform
- **Bilateral linkages**: UPI-PayNow (India-Singapore), UPI-PromptPay (India-Thailand)
- **SWIFT GPI**: Improved correspondent banking tracking

### G20 Roadmap on Cross-Border Payments
- Building blocks: Harmonized data (ISO 20022), API standards, 24/7 systems, legal frameworks
- Target: <1% cost, <1 hour settlement by 2027

## Payment Security & Fraud

### Security Standards
- **PCI DSS**: Payment card industry data security
- **3D Secure 2.0**: Card-not-present authentication
- **FIDO2**: Phishing-resistant device authentication
- **Token services**: Card tokenization (network tokens)

### Fraud Prevention in Open Payments
- Device binding and device fingerprinting
- Transaction monitoring and ML-based anomaly detection
- Velocity checks (limits per time period)
- Beneficiary verification (UK Confirmation of Payee)
- Cooling-off periods for new payees

## Key Metrics for Payment DPI Assessment

| Metric | Target |
|--------|--------|
| Transaction cost | < $0.01 per transaction |
| Settlement time | < 30 seconds |
| Availability | 99.9%+ (24/7/365) |
| Interoperability | 100% of regulated institutions |
| Dispute resolution | < 3 business days |
| Financial inclusion | > 80% adult population with account |

## Standards & References

- **ISO 20022**: Financial messaging standard
- **EMVCo**: Card payment standards
- **PCI DSS**: Payment security standard
- **NPCI UPI**: India's UPI technical specifications
- **BIS CPMI**: Committee on Payments and Market Infrastructures reports
- **GSMA**: Mobile money interoperability guidelines
- **World Bank Remittances**: KNOMAD remittance data
