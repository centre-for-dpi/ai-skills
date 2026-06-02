---
name: dpi-use-cases
description: End-to-end DPI use cases — health, financial inclusion, social protection, agriculture, education. Use when showing how DPI layers combine to deliver real-world development outcomes.
---

# DPI Use Cases

## How DPI Layers Combine to Create Value

Each DPI use case typically stacks multiple DPI layers:

```
Use Case = Identity Layer + Payments Layer + Data Exchange Layer
           + Registries + Verifiable Credentials + Application
```

**The DPI multiplier effect**: Each new DPI layer you add doesn't just enable one more use case — it exponentially increases the possible combinations that create value.

## Financial Inclusion

### Use Case: No-KYC Account Opening (eKYC)

**Problem**: Traditional bank account opening requires in-person visits, physical documents, and takes days.

**DPI Solution**:
```
1. User authenticates via national identity system (OTP, biometric, or FIDO2)
2. Bank calls identity OIDC gateway (e.g. e-Signet) → eKYC returns name, address, photo, DOB
3. Account opened in < 2 minutes, fully remote
4. No physical documents needed
5. Biometric authentication for subsequent logins
```

**Impact (India)**:
- KYC cost: $23 → $0.15
- 300M unbanked accounts opened via eKYC (2014-2020)
- PM Jan Dhan Yojana: 500M+ accounts using Aadhaar eKYC

### Use Case: Credit for Thin-File Borrowers

**Problem**: Most rural/low-income people have no credit history — banks can't assess their creditworthiness.

**DPI Solution (India Account Aggregator)**:
```
1. Loan applicant consents via AA app
2. Bank fetches 12-month transaction history from multiple FIPs
3. Fetches GST returns (income verification)
4. Fetches MSME registry data (if business)
5. ML model scores on cash flows, not just credit history
6. Loan disbursed in hours
```

**Impact**:
- 50M MSME credit gap addressed via AA-based lending
- Kisan Credit Card: Farmers get credit against land records + AA data

### Use Case: Mobile Money Interoperability

**Problem**: M-Pesa users can't pay bKash users; wallet silos prevent inclusion.

**DPI Solution (Tanzania / Ghana model)**:
```
National Interoperability Switch
        ↓
All mobile money operators connect via standardized APIs
        ↓
Any-to-any transfer regardless of wallet provider
        ↓
Linked to national ID for KYC compliance
```

## Social Protection

### Use Case: Direct Benefit Transfer (DBT)

**Problem**: Welfare payments leak 30-40% to middlemen, ghost beneficiaries, or duplicate payments.

**DPI Solution (India DBT)**:
```
1. Beneficiary registered with Aadhaar + linked bank account
2. Central registry deduplicates against all other beneficiaries
3. Payment instruction: scheme → PFMS → NPCI → bank
4. Aadhaar Payments Bridge routes to correct account
5. Beneficiary receives SMS notification
6. Failed payment: reason code → exception handling
```

**Impact**:
- $33B saved in 10 years via leakage reduction
- 1,300+ schemes via DBT
- COVID relief: 400M payments in 48 hours (vs. weeks without DPI)

### Use Case: Conditional Cash Transfer Verification

**Problem**: Beneficiaries must prove they met conditions (child in school, prenatal visits) to receive payments.

**DPI Solution**:
```
1. Health worker visits and logs vaccine administration in HMIS
2. HMIS issues Verifiable Credential to child's health record
3. Beneficiary shares VC with welfare agency (via AA or Inji)
4. Welfare agency verifies VC → triggers payment automatically
5. Condition proof and payment linked in audit trail
```

## Healthcare

### Use Case: Longitudinal Health Records

**Problem**: A patient visits 5 hospitals; each has incomplete records. Doctors over-prescribe, miss allergies, duplicate tests.

**DPI Solution (ABDM)**:
```
Patient has ABHA (Ayushman Bharat Health Account)
        ↓
Each healthcare touchpoint links records to ABHA
        ↓
Patient consents via ABDM app to share records with new doctor
        ↓
New doctor sees full history: diagnoses, medications, allergies, labs
        ↓
Better care, fewer duplicate tests, no drug interactions missed
```

**FHIR-based data flow**:
```json
{
  "resourceType": "Bundle",
  "type": "document",
  "entry": [
    {
      "resource": {
        "resourceType": "Composition",
        "subject": { "reference": "Patient/ABHA-1234567890" },
        "title": "Discharge Summary"
      }
    }
  ]
}
```

### Use Case: Vaccination Management

**Problem**: Track 1B+ vaccination events across millions of facilities; issue verifiable certificates; prevent fraud.

**DPI Solution (CoWIN / COWIN)**:
```
1. Citizen registers with Aadhaar or mobile
2. Appointment booked at nearest facility
3. Vaccine administered → health worker logs in CoWIN
4. Verifiable Certificate generated (WHO DDCC standard)
5. Certificate stored in ABHA / DigiLocker
6. Certificate shared at airport via QR code
```

**Technical stack**:
- Sunbird RC for certificate issuance
- W3C VC for certificate format
- WHO DDCC (Digital Documentation of COVID-19 Certificates) standard

### Use Case: Insurance Claims Processing

**Problem**: Health insurance claims take 2-4 weeks; require physical discharge summaries; prone to fraud.

**DPI Solution**:
```
Hospital admission → claim initiated
        ↓
Hospital submits FHIR-based claim to insurer via AA
(Discharge summary, diagnosis codes, procedure codes)
        ↓
Insurer verifies patient identity via e-Signet/eKYC
        ↓
AI-based claim assessment
        ↓
Direct payment to hospital (UPI B2B)
        ↓
Patient receives settlement notification
Total time: Hours, not weeks
```

## Agriculture

### Use Case: Kisan Credit Card (Farm Credit)

**Problem**: Farmers need seasonal credit but have no formal credit history; banks can't verify landholding.

**DPI Solution**:
```
1. Farmer authentication via Aadhaar OTP
2. Farmer consents to share land records (via Account Aggregator)
3. Land registry data fetched: acreage, crops, irrigation
4. Credit eligibility calculated
5. Credit limit = f(land size × crop type × historical yields)
6. Kisan Credit Card issued digitally
7. Payments via UPI from loan account
```

### Use Case: Agricultural Input Subsidies

**Problem**: Fertilizer subsidies often captured by middlemen; farmers pay full price.

**DPI Solution (India PM-PRANAM / DBT for Fertilizers)**:
```
1. Farmer identified via Aadhaar at point of sale
2. Retailer's PoS device authenticates farmer (OTP/biometric)
3. Subsidy deducted directly at PoS
4. Government reimburses retailer via DBT
5. Farmer pays only net (subsidized) price
```

**PoS Transaction Flow**:
```
Farmer presents Aadhaar → PoS → Aadhaar Auth (IDA)
                                    ↓ auth success
                         → Subsidy DBT via UPI
                                    ↓
                         Retailer receives reimbursement
                         Farmer pays subsidized price
```

### Use Case: Crop Insurance (PMFBY)

**Problem**: Crop insurance claims require ground verification; slow, expensive, prone to fraud.

**DPI Solution**:
```
1. Farmer registers for insurance via Aadhaar + land records
2. Satellite imagery (Sentinel-2) monitors crop health throughout season
3. Weather station data (IoT) logs rainfall, temperature
4. AI model detects crop failure automatically
5. Claim triggered without farmer filing; evidence from satellite + weather
6. Payment via UPI/DBT within 2 weeks
```

## Education

### Use Case: Academic Credential Verification

**Problem**: Employers receive 1000s of degree certificates; verification takes weeks; fake degrees common.

**DPI Solution**:
```
University issues degree as Verifiable Credential
        ↓
Graduate stores in Inji Wallet / DigiLocker
        ↓
Employer requests credential during hiring
        ↓
Graduate shares VC via QR code or OpenID4VP
        ↓
Employer verifies in seconds (cryptographic proof, no manual check)
```

**VC Format (Education)**:
```json
{
  "type": ["VerifiableCredential", "UniversityDegreeCredential"],
  "issuer": "did:web:university.example.ac.in",
  "credentialSubject": {
    "id": "did:example:student",
    "studentId": "2019CS001234",
    "degree": "B.Tech Computer Science",
    "institution": "IIT Bombay",
    "graduationYear": 2023,
    "cgpa": 8.7,
    "honors": "Distinction"
  }
}
```

### Use Case: Scholarship Disbursement

**Problem**: Scholarships require extensive paperwork; processing takes months; funds don't reach intended students.

**DPI Solution**:
```
1. Student applies online via National Scholarship Portal
2. Identity verified via Aadhaar eKYC
3. Eligibility checked: income (via IT returns + AA), marks (via board API)
4. Institute verifies enrollment (API to institution registry)
5. Scholarship approved automatically if all conditions met
6. Disbursement via DBT to student's bank account
```

## Governance and Public Services

### Use Case: Digital Identity for G2C Services

**Problem**: Citizens need physical presence + multiple documents to access any government service.

**DPI Solution (Singapore SingPass model)**:
```
Citizen has national digital identity
        ↓
Login to any government service via OIDC
(e-Signet or equivalent)
        ↓
Service receives verified claims (name, address, etc.)
        ↓
No form-filling for already-known data
        ↓
Service rendered digitally
```

### Use Case: Property Registration

**Problem**: Land registration requires multiple visits, high stamp duty, susceptible to fraud.

**DPI Solution**:
```
1. Buyer + seller authenticate via Aadhaar/national ID
2. Land records fetched from digital registry
3. Title chain verified (no encumbrances)
4. Payment via UPI (stamp duty + registration fee)
5. Digital deed signed (using Aadhaar eSign)
6. Land registry updated automatically
7. New title deed issued as Verifiable Credential
```

### Use Case: Voting / Electoral Roll Management

**Problem**: Voter rolls contain duplicates, deceased voters, and phantom entries.

**DPI Solution**:
```
1. Voter roll linked to Aadhaar (optional, consent-based)
2. Deduplication run against ID database
3. Deaths registered in CRVS (civil registration — see **dpi-registries** skill) → automatic removal from voter roll
4. Address changes in ID system → voter roll updated
5. Voter ID as Verifiable Credential in wallet
```

## Cross-Border DPI Use Cases

### Use Case: Cross-Border Remittances

**Problem**: Migrant workers pay 5-8% to send money home; takes 1-3 days.

**DPI Solution (UPI-PayNow linkage)**:
```
Indian worker in Singapore
        ↓
Sends via Singapore PayNow (using phone number)
        ↓
PayNow ↔ UPI real-time link (bilateral agreement)
        ↓
Payment arrives at UPI VPA in India within seconds
Cost: < 1%
```

### Use Case: Cross-Border Digital Credentials

**Problem**: Indian degree not recognized in UAE without expensive attestation.

**DPI Solution**:
```
Student shares degree VC (W3C format)
        ↓
UAE verifier checks issuer DID against India's trust registry
        ↓
Signature verified cryptographically
        ↓
Credential accepted (if India-UAE mutual recognition agreement exists)
```

## DPI Use Case Design Framework

When designing a DPI use case, answer these questions:

### 1. Identity Layer Needs
- Is user verification required? At what assurance level?
- Is eKYC (demographic data) needed?
- Should authentication be real-time or deferred?

### 2. Data Exchange Needs
- What data needs to flow from where to where?
- Is consent required? From whom? When?
- What is the frequency: one-time, periodic, ongoing?

### 3. Payment Needs
- Is a payment part of the use case?
- G2P (disbursement) or P2G (collection) or P2P?
- What rails are available in the country?

### 4. Registry Needs
- What entity records are needed (beneficiary, facility, professional)?
- Is deduplication required?
- What identifiers link across registries?

### 5. Credential Needs
- Should an outcome credential be issued (proof of enrollment, completion, etc.)?
- Who will verify it? Online or offline?
- How long is the credential valid?

## Impact Metrics Reference

| Use Case | Country | Key Metric |
|---------|---------|-----------|
| eKYC bank account | India | 300M accounts opened; cost from $23 → $0.15 |
| DBT welfare | India | $33B saved; 0 ghost beneficiaries |
| COVID payments | India | 400M payments in 48 hours |
| COVID vaccination certificates | Global | 2B+ VCs issued |
| UPI payments | India | 13B transactions/month |
| PIX | Brazil | 150M users in 2 years |
| Land records | Various | 99%+ fraud reduction in digitized states |
| Academic VC | India (DigiLocker) | 6B+ documents, 1800+ issuers |
