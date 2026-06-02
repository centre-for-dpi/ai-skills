---
name: dpi-un-safeguards
description: Universal DPI Safeguards Framework — 18 principles, 13 risks, 5 lifecycle stages, 300+ recommendations. Use when evaluating DPI governance, rights-based design, or responsible DPI deployment.
---

# Universal DPI Safeguards Framework

> **Framework**: https://www.dpi-safeguards.org/framework  
> **Resources**: https://www.dpi-safeguards.org/resources  
> **Full PDF**: https://framework-dpi-safeguards.org/frameworkpdf  
> **GitBook**: https://safedpi.gitbook.io/safeguards

## What is the Universal DPI Safeguards Framework?

The **Universal DPI Safeguards Framework** is a practical, rights-based guidance document published by the United Nations in 2024 (updated 2025) to ensure that Digital Public Infrastructure is **safe, inclusive, and effective**.

**Full title**: *"A Guide to Building Safe and Inclusive DPI for Societies"*

**The core premise**: DPI is powerful infrastructure — and like all powerful infrastructure, it can harm as well as help. Without intentional safeguards, DPI can enable surveillance, exclude marginalized populations, create vendor lock-in, and undermine democratic accountability. The Framework provides a structured methodology to anticipate, prevent, and mitigate these risks across the full DPI lifecycle.

**Governance**: Developed under the stewardship of:
- **OSET** (Office of the Secretary-General's Envoy on Technology)
- **UNDP** (United Nations Development Programme)

**Process**: Multi-stakeholder; 44 working group members; global convenings; in-country consultations; ecosystem feedback cycles.

**Scale**: Adopted as part of the **Global Digital Compact** — signed by 193 UN member states, which recognized DPI's role in achieving the Sustainable Development Goals (SDGs).

---

## Framework Architecture

The Framework operates across four intersecting dimensions:

```
┌────────────────────────────────────────────────────────────────────┐
│              Universal DPI Safeguards Framework                    │
│                                                                    │
│  18 PRINCIPLES         13 RISKS           5 RESPONSIBLE           │
│  ────────────          ────────           AUTHORITIES             │
│  9 Foundational        3 Categories       R1 Government           │
│  9 Operational         Safety             R2 Regulator            │
│                        Inclusion          R3 Donor                │
│                        Structural         R4 Tech Provider        │
│                        Vulnerabilities    R5 Advocates            │
│                                                                    │
│  5 LIFECYCLE STAGES            300+ RECOMMENDATIONS               │
│  ─────────────────             ────────────────────               │
│  L1 Conception & Scoping       Per authority × per stage          │
│  L2 Strategy & Design          × per risk × per principle         │
│  L3 Development                                                   │
│  L4 Deployment                                                    │
│  L5 Operations & Maintenance                                      │
└────────────────────────────────────────────────────────────────────┘
```

The Framework generates **actionable safeguard recommendations** by combining:
- Which **risk** is being addressed
- At which **lifecycle stage** it arises
- By which **responsible authority**
- Guided by which **principles**

---

## The 18 Principles

Principles are divided into two categories: **Foundational** (apply to all DPI, at all times) and **Operational** (come into play at the operational level, may vary by context).

### Foundational Principles (F1–F9)

| # | Principle | Core meaning |
|---|-----------|-------------|
| **F1** | **Do no harm** | A human rights-based framework must be integrated throughout the DPI lifecycle to anticipate, assess, and mitigate potential human rights harms and power differentials |
| **F2** | **Do not discriminate** | All individuals, regardless of intersecting identities, must have unbiased access and equal opportunity; risks for vulnerable and historically marginalized groups must be mitigated |
| **F3** | **Do not exclude** | All individuals must have a choice of channels (digital and non-digital) to access DPI-enabled services; access must not be limiting, conditional, or mandatory |
| **F4** | **Reinforce transparency and accountability** | DPI must be developed with democratic participation, public oversight, fair market competition, and must avoid vendor lock-in |
| **F5** | **Uphold the rule of law** | DPI must operate within legal frameworks; legal safeguards must govern data use, access, and accountability |
| **F6** | **Promote autonomy and agency** | Individuals must have meaningful control over their data and how DPI affects their lives |
| **F7** | **Foster community engagement** | DPI design and governance must involve affected communities throughout the lifecycle |
| **F8** | **Ensure effective remedy and redress** | Individuals must have accessible mechanisms to challenge DPI decisions, errors, and harms |
| **F9** | **Focus on future sustainability** | DPI must be designed for long-term financial, technical, and institutional sustainability |

### Operational Principles (O1–O9)

| # | Principle | Core meaning |
|---|-----------|-------------|
| **O1** | **Leverage market dynamics** | Design DPI to promote competitive markets; prevent monopolization of the infrastructure layer |
| **O2** | **Evolve with evidence** | DPI must be iteratively improved based on monitoring data, user feedback, and impact evidence |
| **O3** | **Ensure data privacy by design** | DPI must embed legal, regulatory, and technical privacy principles (data minimization, delinking, limiting observability) from the outset |
| **O4** | **Assure data security by design** | DPI must incorporate encryption, pseudonymization, and continuously upgraded security measures |
| **O5** | **Ensure data protection during use** | Active safeguards must govern how data is accessed, retained, shared, and deleted during DPI operation |
| **O6** | **Respond to gender, ability, or age** | DPI must be designed to meet the specific needs and vulnerabilities of women, persons with disabilities, children, and the elderly |
| **O7** | **Practice inclusive governance** | DPI governance bodies must include diverse, marginalized voices; governance must not be captured by narrow interests |
| **O8** | **Sustain financial viability** | DPI financing models must be sustainable; avoid capture by commercial interests or collapse due to funding gaps |
| **O9** | **Build and share open assets** | DPI should be built on and contribute to open-source, open-standard digital public goods — avoiding proprietary lock-in |

---

## The 13 Risks

The Framework identifies 13 key risks across the DPI lifecycle, grouped into three categories:

### Category 1: Safety Risks

Risks that can directly harm individuals through DPI systems:

| Risk | Description |
|------|-------------|
| **Privacy breach** | Unauthorized access to, or exposure of, personal data held in DPI systems |
| **Surveillance and profiling** | DPI data aggregation enabling mass surveillance or discriminatory profiling by state or non-state actors |
| **Cybersecurity threats** | Attacks on DPI infrastructure — breaches, ransomware, denial of service — disrupting critical public services |
| **Data misuse** | Data collected for one purpose repurposed for unauthorized or harmful uses (scope creep, function creep) |
| **Algorithmic bias and harm** | Automated decisions embedded in DPI producing discriminatory or harmful outcomes |

### Category 2: Inclusion Risks

Risks that exclude populations from DPI benefits:

| Risk | Description |
|------|-------------|
| **Digital exclusion** | Populations without devices, connectivity, or digital literacy cannot access DPI-enabled services |
| **Identity exclusion** | Individuals without documents, biometrics, or formal identity cannot enroll in DPI |
| **Discrimination** | DPI design or implementation that produces unequal access or outcomes based on gender, age, disability, ethnicity, or other characteristics |
| **Forced digitization** | Digital channels become the only channel, excluding those who cannot or choose not to use digital systems |

### Category 3: Structural Vulnerability Risks

Systemic risks that undermine the integrity and sustainability of DPI:

| Risk | Description |
|------|-------------|
| **Vendor lock-in** | Dependence on proprietary technology constrains future flexibility, inflates long-term costs, and creates geopolitical vulnerabilities |
| **Institutional capacity gaps** | Insufficient government capacity to govern, regulate, and maintain DPI safely over time |
| **Governance capture** | DPI governance dominated by narrow interests (commercial, political, or donor) rather than the public interest |
| **Financial unsustainability** | DPI lacking viable long-term funding models collapses or becomes compromised by commercial interests |

---

## The 5 Responsible Authorities

The Framework assigns safeguard responsibilities across five categories of actors, recognizing that no single actor can implement all safeguards alone:

### R1: Government
The primary DPI owner and operator; accountable for legal frameworks, policy, funding, and public interest alignment.

**Key responsibilities:**
- Enact data protection legislation before deploying DPI
- Ensure DPI is accessible through non-digital alternatives
- Establish independent oversight bodies
- Mandate human rights impact assessments before deployment
- Set open standards requirements in procurement

### R2: Regulator
Independent regulatory and oversight bodies; ensure compliance, protect citizens, and hold operators accountable.

**Key responsibilities:**
- License and audit DPI operators
- Investigate complaints and enforce data protection rules
- Mandate interoperability and prevent monopolistic behavior
- Conduct periodic DPI impact assessments
- Require cybersecurity audits at each lifecycle stage

### R3: Donor
Development banks, bilateral donors, and philanthropic funders who finance DPI programs in LMICs.

**Key responsibilities:**
- Require safeguards compliance as a condition of funding
- Fund open-source solutions (DPGs) in preference to proprietary
- Support multi-year capacity-building, not just technology procurement
- Require independent impact evaluations
- Avoid creating aid dependency on donor-controlled DPI infrastructure

### R4: Technology Provider
Companies, open-source communities, and DPG maintainers that build DPI software and infrastructure.

**Key responsibilities:**
- Build privacy and security by design into DPI platforms
- Publish open APIs and open-source code where possible
- Maintain documentation and provide long-term support
- Disclose known vulnerabilities and risks
- Align with open standards (FHIR, ISO 20022, W3C VC, OpenID Connect)

### R5: Advocates
Civil society organizations, academia, media, and affected communities that hold DPI accountable.

**Key responsibilities:**
- Participate in DPI governance bodies
- Monitor DPI for rights violations and exclusion
- Represent marginalized communities in design processes
- Publish independent assessments
- Support grievance mechanisms for affected people

---

## The 5 Lifecycle Stages

The Framework maps safeguards to each stage of the DPI lifecycle:

### L1: Conception and Scoping
*What problem is DPI solving? Who will it affect?*

Key safeguards:
- Human rights impact assessment before any technical design begins
- Stakeholder mapping: who benefits, who is at risk
- Legal and regulatory landscape review
- Exclusion risk assessment (who will be left out)
- Define success metrics that include inclusion and safety, not just scale

### L2: Strategy and Design
*How will DPI be built? What architecture choices matter?*

Key safeguards:
- Privacy by design: data minimization, pseudonymization, consent architecture
- Federated architecture: no single point of surveillance or failure
- Interoperability-first: open APIs, open standards, no proprietary lock-in
- Non-digital fallback: every DPI service must have an analog alternative
- Security architecture: threat modeling, encryption by default, penetration testing plan
- Inclusive design: accessibility standards, multilingual support, low-literacy UX

### L3: Development
*Building the system — where technical safeguards are embedded.*

Key safeguards:
- Security code review and penetration testing
- Open-source audit trails where possible
- Disability and gender accessibility testing
- Consent flow implementation and testing
- Data retention and deletion mechanisms built in
- Incident response plan developed before go-live

### L4: Deployment
*Going live — managing rollout risks.*

Key safeguards:
- Phased rollout with non-digital fallback operational from day one
- Training for frontline staff on rights and safeguards
- Public communication in local languages about data rights
- Grievance mechanism live before DPI is live
- Real-time monitoring dashboard (inclusion metrics, error rates)
- Rollback plan if deployment causes harm

### L5: Operations and Maintenance
*Sustaining DPI safely over time.*

Key safeguards:
- Periodic privacy and security audits (minimum annually)
- Continuous inclusion monitoring (who is using, who is excluded)
- Algorithm audits for bias where AI/ML is used
- Evolving legal framework kept current with DPI capabilities
- Financial sustainability review
- Technology refresh plan to avoid obsolescence or lock-in
- Community feedback loops — affected people can report problems

---

## 300+ Recommendations: How They Work

The Framework's 300+ recommendations are not a single list — they are **generated by intersection**:

```
Recommendation = Risk × Lifecycle Stage × Responsible Authority

Example:
  Risk: Surveillance and profiling
  Stage: L2 Strategy and Design
  Authority: R1 Government
  
→ Recommendation: "Governments must establish legal prohibitions on 
   secondary use of DPI data for law enforcement or intelligence 
   purposes without judicial authorization, embedded in enabling 
   legislation before the system is designed."
```

This matrix approach means:
- Every responsible authority gets specific, actionable guidance
- Recommendations are stage-appropriate (not generic)
- The full set covers the entire DPI lifecycle comprehensively

---

## Country Implementation Pathway

### The DPI Safeguards Accelerator (2025)

Launched at the 80th UN General Assembly (September 2025), the **DPI Safeguards Accelerator** is the operational mechanism for country implementation:

```
Country commits to DPI Safeguards Framework
        ↓
Onboarding pathway: Initial safeguards assessment
        ↓
Technical assistance: Training, expert support, peer learning
        ↓
Assessments:
  - Digital readiness assessment
  - Inclusion assessment (who is excluded)
  - Technological interoperability assessment
  - Regulatory framework assessment
        ↓
Safeguards adoption roadmap developed
        ↓
Implementation support (through Accelerator hub)
        ↓
Monitoring and feedback loop
```

### 50-in-5 Campaign

**50-in-5** is the country-led campaign linked to the DPI Safeguards Initiative:
- Goal: 50 countries design, launch, and scale DPI components by 2028
- Integrated with the Universal DPI Safeguards Framework — countries commit to safeguards as part of the 50-in-5 pledge
- Focuses on three DPI components: digital identity, digital payments, and data exchange

### Country Onboarding Pathways

The DPI Safeguards Initiative has developed structured onboarding pathways for government actors:
1. **Commitment**: Political sign-on to the Universal DPI Safeguards Framework
2. **Assessment**: Baseline assessment of current DPI safeguards gaps
3. **Design integration**: Embed safeguards into DPI roadmaps and procurement
4. **Implementation**: Technical assistance for specific safeguard measures
5. **Monitoring**: Ongoing metrics and feedback

---

## Relationship to Other Global Standards

| Standard / Initiative | Relationship to DPI Safeguards Framework |
|----------------------|------------------------------------------|
| **Global Digital Compact (GDC)** | The Framework is the technical implementation arm of the GDC DPI commitments — 193 states signed the GDC |
| **DPG Standard** | Aligned — the DPG Standard embeds Framework safeguards; DPGs that meet the DPG Standard are preferred DPI building blocks |
| **GDPR / data protection laws** | Framework O3/O4/O5 principles align with GDPR; Framework designed to be compatible with national data protection legislation |
| **UNICEF Child Safeguards** | Framework O6 (respond to gender/ability/age) maps to UNICEF's DPI for Children safeguards — see dpi-childhood skill |
| **CDPI DPI Principles** | Complementary — CDPI focuses on technical architecture; DPI Safeguards Framework focuses on governance and rights |
| **eIDAS / PKI frameworks** | Framework O3/O4 principles align with electronic trust infrastructure requirements |
| **OpenHIE / health data standards** | Framework O3–O5 apply to health DPI (see dpi-health skill) |

---

## Using the Framework in Practice

### For Governments Designing New DPI
1. Run an **L1 Conception assessment** against all 13 risks before any procurement
2. Require **F3 (do not exclude)** compliance — mandate non-digital alternatives in all DPI tenders
3. Enact **data protection legislation** before deploying DPI (F5 — uphold the rule of law)
4. Require **O9 (open assets)** — preference for DPGs in procurement
5. Establish **R2 independent regulator** with DPI oversight mandate before go-live

### For Donors Funding DPI
1. Make **Framework alignment a condition of funding** — not a box-ticking exercise
2. Fund **multi-year capacity building** (R3 responsibilities), not just software procurement
3. Require **independent inclusion assessments** at L4/L5 stages
4. Prefer **DPG-based solutions** (O9) to avoid creating proprietary lock-in
5. Support countries in standing up **grievance mechanisms** (F8) before DPI goes live

### For Technology Providers
1. Map your product against the **18 principles** — document where each is addressed
2. Publish **privacy impact assessments** for DPI deployments
3. Support **open APIs and open standards** (O9) — avoid proprietary integration patterns
4. Build **consent flows** and **data deletion** into the product, not as afterthoughts (O3)
5. Participate in **Framework assessments** transparently

### For Civil Society Advocates (R5)
1. Use the **13 risks** as an advocacy checklist — demand governments show how each is mitigated
2. Demand **public inclusion metrics** — who is using DPI, who is excluded, at what rates
3. Use **F8 (effective remedy)** to hold governments accountable when DPI causes harm
4. Participate in **DPI governance bodies** — F7 and O7 require your presence

---

## Assessment Tools

The DPI Safeguards Initiative provides the following assessment tools:

| Assessment | Purpose | Audience |
|-----------|---------|---------|
| **Digital Readiness Assessment** | Is the country ready to implement DPI safely? | Government, Donors |
| **Inclusion Assessment** | Who is currently excluded from DPI benefits? | Government, Advocates |
| **Technological Interoperability Assessment** | Are DPI components interoperable with open standards? | Technology Providers, Government |
| **Regulatory Framework Assessment** | Does the legal framework enable safe DPI? | Regulators, Government |
| **Safeguards Gap Assessment** | Which safeguards are missing from current DPI? | All authorities |

Access at: https://www.dpi-safeguards.org/assessments

---

## Key References

- **Framework homepage**: https://www.dpi-safeguards.org/framework
- **Resources and tools**: https://www.dpi-safeguards.org/resources
- **Full PDF guide**: https://framework-dpi-safeguards.org/frameworkpdf
- **GitBook (full reference)**: https://safedpi.gitbook.io/safeguards
- **UNDP press release**: https://www.undp.org/press-releases/un-releases-universal-dpi-safeguards-framework-promote-safe-and-inclusive-digital-public-infrastructure
- **50-in-5 campaign**: https://50in5.net
- **DPI Safeguards Accelerator**: https://www.undp.org/digital/blog/putting-safeguards-action-50-5-and-dpi-safeguards-initiative-launch-accelerator-countries
- **Global Digital Compact**: https://www.un.org/techenvoy/global-digital-compact
- **DPG Standard alignment**: https://www.digitalpublicgoods.net/blog/dpg-standard-universal-dpi-safeguards-safe-inclusive-digital-future
- **Assessments portal**: https://www.dpi-safeguards.org/assessments
