---
name: dpi-childhood
description: DPI for Children — UNICEF framework, child identity, birth registration, biometric limits, child-safe design. Use when designing DPI systems that serve or affect children.
---

# DPI for Children

> **Reference**: https://www.unicef.org/digitalimpact/reports/safeguarding-digital-public-infrastructure-children  
> **See also**: https://www.unicef.org/digitalimpact/advancing-digital-public-infrastructure-benefits-children

## Why Children Need DPI-Specific Framing

DPI is not child-neutral. A child encounters DPI from the moment of birth — through civil registration — and DPI shapes every major life service: health, education, social protection, nutrition, and legal protection. Yet most DPI is designed for adults.

**The stakes are uniquely high for children:**
- **150 million** children under age 5 are unregistered globally — invisible to every DPI system and unable to access the services it enables
- Children cannot give informed legal consent; their data is held by parents, guardians, or states
- Children's biometrics change rapidly — standard adult biometric enrollment fails for children under 5–12
- Children are among the most vulnerable to surveillance, exploitation, and data misuse
- DPI exclusion early in life compounds into lifelong inequality

**UNICEF's position**: DPI holds transformative potential for children — but only if it is *intentionally* designed with children's rights, not adapted as an afterthought.

---

## A Child's Journey Through DPI

A child contacts DPI at every life stage. Each contact point is an opportunity — and a risk:

```
Birth
  └── Civil registration (CRVS) → birth certificate
        └── Legal identity established
              └── Enrollment in national ID (age varies by country)

Early childhood
  └── Immunisation records → health DPI
  └── Nutrition program → social protection registry
  └── Child grant/benefit → payments DPI

School age
  └── School enrollment → education registry
  └── Learning outcomes → education data systems
  └── Digital literacy → risk of online exploitation

Adolescence
  └── Progressive legal capacity → consent rights expand
  └── Exam/qualification credentials → verifiable credentials
  └── Transition to adult DPI systems

Adulthood
  └── Full legal capacity; standard DPI applies
```

**Key principle**: every DPI design decision about birth registration, child identity, school enrollment, or health records has lifetime consequences — exclusion at birth means exclusion for life.

---

## Foundational Principle: Birth Registration as First DPI Rail

Birth registration is **the first DPI touchpoint** for every child and the foundation of all subsequent DPI interactions.

**What birth registration enables:**
- Legal identity → the prerequisite for accessing all other DPI services
- Enrollment in health programs (immunisation, nutrition)
- Access to education
- Child protection (trafficking, child labour prevention)
- Social protection benefit eligibility
- Future national ID enrollment

**The gap:**
- ~150 million children under 5 lack birth registration
- Concentrated in Sub-Saharan Africa and South Asia
- Unregistered children are invisible to government planning and excluded from digital services

**DPI approach to closing the gap:**
```
Hospital / health facility notifies birth (CRVS integration)
        ↓ OpenCRVS / national CRVS system
Birth record created → certificate issued (within days, not months)
        ↓ interoperability link
Health system: child enrolled in immunisation program
        ↓
Social protection: family enrolled in child benefit program
        ↓
School registry: child pre-enrolled for school-age enrollment
```

**Critical design requirement**: CRVS systems must interoperate with health, social protection, and education registries to realize the DPI multiplier effect for children.

> See the **dpi-opencrvs** skill for the open-source CRVS platform.

---

## Child Identity — Design Safeguards

### The Biometrics Problem for Children

Standard biometric methods do not work reliably for young children:

| Biometric | Age reliability threshold | Risk |
|-----------|--------------------------|------|
| **Fingerprints** | Unreliable under 12 (under 5: very poor) | Fingerprints are too small, change rapidly, poor ridge quality |
| **Iris scan** | Reasonably stable from 2–3 years | Better for young children than fingerprints |
| **Face recognition** | Changes rapidly in childhood | High false-non-match rate across years |
| **Palmprints** | Similar to fingerprints — unreliable under 5 | |

**UNICEF recommendation:**
- Do NOT require biometrics for birth registration of infants
- Use guardian/parent biometric linkage for young children (child linked to verified parent)
- Introduce child biometrics (iris or fingerprint) at school enrollment age (6–8) when quality improves
- Recapture biometrics at age 15 and 18 as reliability improves
- Never make biometric enrollment a barrier to receiving benefits

### Child Identity Architecture

```
Birth registration (no child biometric needed)
        ↓ issues birth certificate
National ID system assigns child ID (linked to parents' IDs)
        ↓ child pseudonymous token
Sectoral systems (health, education, social protection) use child token
        [child biometric NOT required until age 5–8+]

Age 6–8: school enrollment → iris/fingerprint added to child record
Age 15: biometric re-verification (growth-adjusted)
Age 18: transition to adult identity system
```

**Parent-child linkage**: For young children, identity assertions can be made via the parents' verified identity — "this child's birth certificate is linked to verified parents A and B." This is sufficient for program enrollment without requiring child biometrics.

### Data Minimization for Child Identity

- Collect only what is required for the specific service
- Child identity data should NOT be used to build comprehensive surveillance profiles
- Health records, education records, and social protection records should be siloed with explicit cross-sector consent
- Never aggregate child data across sectors without legal basis and child-rights impact assessment

---

## Consent and Children

Legal capacity to consent varies by age, jurisdiction, and context. DPI systems must handle this:

### Consent Hierarchy

| Age | Consent model |
|-----|-------------|
| **0–12** | Parent/guardian consent required for all data processing |
| **12–15** | Progressive capacity: joint consent (child + guardian) for sensitive data |
| **15–18** | Increasing autonomy; local law governs; for sensitive data (health, sexual health) may consent alone |
| **18+** | Full legal capacity; standard adult consent |

### Design Implications

- Registration flows for children must capture guardian consent, not child consent
- Consent artefacts must record the guardian's identity and relationship to the child
- Systems must support consent revision as children age — guardian consent must transition to child consent
- Emergency override: child protection agencies may access records without consent for child safeguarding

### GDPR/Data Protection for Minors

Most data protection regimes have special provisions for children's data:
- GDPR: parental consent required for children under 16 (member state option: 13–16)
- COPPA (US): parental consent for under 13
- UNCRC: best interests of the child must govern all data decisions
- **DPI design requirement**: age verification or age estimation mechanisms must be in place before collecting children's data

---

## Key Use Cases: DPI for Children

### 1. Child Social Protection (Child Grants / Nutrition Programs)

**How DPI enables child grants at scale:**
```
Birth registration → child ID created
        ↓ automated eligibility check
Social protection registry → family meets criteria
        ↓
Payment rail → child grant disbursed to mother/guardian account
        ↓
Death/aging-out notification from CRVS → benefit stopped
```

**Risk**: If child ID is not de-duplicated, families may receive multiple grants (overpayment) or fraud. If CRVS ↔ social protection interoperability fails, deceased children remain in beneficiary registers (ghost beneficiaries).

**Impact**: Ethiopia, Zambia, South Africa use CRVS–social protection links to reduce ghost beneficiaries and ensure every eligible child receives their benefit.

### 2. Immunisation and Child Health Records

**DPI-enabled immunisation:**
```
Birth registration → child enrolled in immunisation program
        ↓ digital immunisation record (FHIR Immunization resource)
Community health worker visits → records vaccine dose in mobile app
        ↓ offline-first sync
National health information system (DHIS2) updated
        ↓
Caregiver receives SMS reminder for next dose
        ↓ at school enrollment age
Immunisation certificate as Verifiable Credential → proof of vaccination
```

**Standards**: FHIR R4 Immunization resource; WHO SMART Guidelines for immunisation; IIS (Immunization Information Systems).

### 3. Education Enrollment and Learning Outcomes

```
Birth certificate → school pre-enrollment at birth
        ↓ at age 6
School enrollment → education registry
        ↓
Learning outcomes recorded (anonymised at aggregate level)
        ↓
Certificate/credential at graduation → Verifiable Credential
        ↓
Cross-border credential recognition (no bilateral agreements needed)
```

**Privacy requirement**: Individual learning data is highly sensitive. Aggregate level acceptable for planning; individual level requires strict access controls.

### 4. Child Protection and Anti-Trafficking

DPI can protect children — but also risks enabling surveillance. The balance:

**Protective uses:**
- CRVS death registration → removes deceased children from at-risk registries
- Cross-border VC for unaccompanied minors → proves identity without carrying documents
- Child protection registries → flags known abusers (with strict access controls)

**Surveillance risks to guard against:**
- Location tracking of minors in social protection systems
- Biometric databases accessible to law enforcement without judicial oversight
- Cross-sector data aggregation creating comprehensive child profiles
- Automated decision-making on child welfare without human review

---

## Risks of DPI for Children

UNICEF identifies five categories of risk:

### 1. Exclusion Risk
- Biometric failure rates for young children → excluded from programs
- Digital literacy gaps → parents cannot navigate digital enrollment
- Connectivity gaps → rural children fail to register
- **Mitigation**: Mandatory fallback alternatives for all DPI services; no "digital only" enrollment

### 2. Surveillance Risk
- National ID systems linked to health, education, social protection, tax → comprehensive child profiles
- Law enforcement access to child data without judicial safeguards
- Location tracking via school check-in or health system
- **Mitigation**: Data silo architecture; strict purpose limitation; no cross-sector aggregation without explicit legal basis; independent oversight

### 3. Data Exploitation Risk
- Child health/education data sold or shared with commercial actors
- Behavioral data used for targeted advertising (children)
- Biometric data shared with foreign governments
- **Mitigation**: Strict prohibition on commercial use of children's DPI data; data sovereignty requirements; independent audit

### 4. Accountability Gap
- When DPI systems fail (wrong enrollment, wrong payment, wrong record), children have no recourse
- Guardians may not understand the system
- Grievance mechanisms inaccessible to children
- **Mitigation**: Child-friendly grievance mechanisms; guardian notification of all data accesses; independent ombudsman for DPI

### 5. Design Exclusion
- DPI designed for literate adults fails for children, elderly guardians, nomadic populations
- Complex digital consent flows exclude low-literacy guardians
- Language barriers in multilingual countries
- **Mitigation**: UX design standards for low-literacy users; oral/visual interfaces; local language support

---

## UNICEF Design Principles for Child-Safe DPI

The UNICEF safeguards framework (2024–2025) defines six principles:

### 1. Privacy by Design
- Children's data is sensitive by default — apply stricter protections than adult data
- Collect minimum data necessary for the service
- No biometrics before age-appropriate thresholds
- Pseudonymise child identifiers across sectors
- Delete data when no longer needed (retention schedules)

### 2. Inclusive Design
- Ensure digital services have non-digital equivalents — registration must be possible without a smartphone
- Design for low-literacy guardians
- Accessible to children with disabilities
- Works in offline/low-connectivity environments
- Supports local languages and scripts

### 3. Fallback Alternatives
- Every DPI service affecting children must have a fallback
- If biometric verification fails, document-based verification must work
- If digital payment fails, cash/voucher alternative must exist
- No child should be denied a service because the digital system failed

### 4. Accountability and Redress
- Automated decisions about children (benefit eligibility, school placement) must have human review pathway
- Children and guardians must be able to see what data is held and correct errors
- Independent oversight body with child rights mandate
- Regular child rights impact assessments of DPI systems

### 5. Data Governance Safeguards
- Legal framework explicitly covers children's data in DPI
- Prohibition on commercial use of children's DPI data
- Cross-sector data sharing requires specific legal basis + child rights impact assessment
- International data transfers subject to adequacy review for children's data

### 6. Child Participation
- Children and adolescents should participate in DPI design (age-appropriate)
- User testing with children and guardians before deployment
- Feedback mechanisms accessible to youth

---

## Interoperability for Children: The CRVS ↔ Health ↔ ID ↔ Social Protection Chain

The highest-impact intervention for children is **linking these four systems**:

```
CRVS (birth registration)
    ↕ interoperability link
Health System (immunisation, nutrition, maternal care)
    ↕
National ID System (foundational identity)
    ↕
Social Protection Registry (child grants, cash transfers, school feeding)
```

**What linking enables:**
- Automatic enrollment in immunisation at birth — no parent action required
- Automatic child grant disbursement — no paper application
- Automatic school pre-enrollment — child ready for school before age 6
- Automatic benefit termination when child dies or ages out — no ghost beneficiaries

**UNICEF partnership model**: UNICEF works with governments to design this interoperability chain, currently implementing in Tanzania and Lesotho through the Co-Develop DPI Safeguards Framework.

---

## Governance for Child-Inclusive DPI

| Mechanism | Purpose |
|-----------|---------|
| **Child Rights Impact Assessment (CRIA)** | Mandatory before DPI deployment affecting children |
| **Data Protection Authority oversight** | Enforce children's data protections in DPI |
| **Independent DPI ombudsman** | Receive and resolve child-related DPI grievances |
| **Parliamentary review** | Oversight of laws governing children's data in DPI |
| **Multi-stakeholder governance body** | Includes civil society, child protection agencies |
| **UN Universal DPI Safeguards Framework** | Global baseline standards; UNICEF implements in countries |

---

## Relationship to Other DPI Skills

| DPI Layer | Child relevance |
|-----------|----------------|
| **dpi-opencrvs** | The CRVS system that registers every child at birth — first DPI touchpoint |
| **dpi-registries** | Functional registries (education, health, social protection) that serve children |
| **dpi-digital-id** | Child identity lifecycle — biometric limitations, guardian linkage, progressive autonomy |
| **dpi-payments** | Child grants, school feeding payments, nutrition transfers via G2P rails |
| **dpi-health** | Child immunisation, maternal and child health records, FHIR Immunization |
| **dpi-data-sharing-models** | Consent-based health and education record sharing for children |
| **dpi-trust-infra** | Age-appropriate consent, guardian consent artefacts, minor data protection |

---

## Key References

- **UNICEF Safeguarding DPI for Children report**: https://www.unicef.org/digitalimpact/reports/safeguarding-digital-public-infrastructure-children
- **UNICEF DPI for Children hub**: https://www.unicef.org/digitalimpact/advancing-digital-public-infrastructure-benefits-children
- **UNICEF Digital Impact — DPI**: https://www.unicef.org/digitalimpact/digital-public-infrastructure-dpi
- **UN Universal DPI Safeguards Framework**: https://www.un.org/techenvoy/global-digital-compact/digital-public-infrastructure
- **UNICEF Birth Registration Data**: https://data.unicef.org/topic/child-protection/birth-registration/
- **UNCRC (Convention on the Rights of the Child)**: https://www.ohchr.org/en/instruments-mechanisms/instruments/convention-rights-child
- **WEF on DPI for children**: https://www.weforum.org/stories/2023/09/how-digital-public-infrastructure-can-unlock-critical-data-for-childrens-rights/
