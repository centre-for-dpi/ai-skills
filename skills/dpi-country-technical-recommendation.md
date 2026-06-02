---
name: dpi-country-technical-recommendation
description: CDPI guidance for countries adopting DPI — neutral tech selection, sequencing, procurement, governance. Use when advising on DPI adoption or reviewing national DPI implementation plans.
---

# DPI Country Technical Recommendations

> **Reference**: https://docs.cdpi.dev/

## Framing: The Two Outcomes DPI Must Deliver Simultaneously

Every DPI architecture decision should be evaluated against these two outcomes:

1. **Public sector scale**: Can government deliver services to every resident of the country at low cost and high quality?
2. **Private sector innovation**: Can the private sector build competitive products and services on the same infrastructure?

If a design choice serves only one of these, it is incomplete. DPI that only enables government services creates a welfare delivery channel, not a digital economy. DPI that only enables private innovation without public service delivery misses its social purpose.

The architectural expression of this is **open rails**:
- Government provides the rails (authentication, payment, data exchange, registries)
- Private sector and civil society build the trains (apps, services, products)
- Neither can own the rails exclusively

## The DPI Approach (vs. Buying Technology)

Adopting DPI is not primarily a technology procurement decision — it is an institutional and design philosophy choice:

| Anti-pattern | DPI Approach |
|-------------|-------------|
| Procure a solution that solves today's problem | Design reusable infrastructure for unknown future use cases |
| Vendor builds, vendor owns, country depends | Country specifies openly; vendor implements; country controls |
| One app for government services | Open API rails; any actor builds on top |
| Centralized database the government controls | Federated architecture; data stays close to source |
| Sector-specific silos | Cross-cutting layers reused across all sectors |
| Private platform as national infrastructure | Purpose-built public infrastructure |

## Core Architectural Principles

> **Reference**: https://docs.cdpi.dev/the-dpi-wiki/dpi-tech-architecture-principles

### 1. Minimalist DPI Layer
DPI should be the thinnest possible layer. Build only what must be shared infrastructure. Everything else should be built by the ecosystem.

*Test*: Could two competing private companies build this functionality themselves? If yes, it doesn't belong in DPI.

### 2. API-First, Not App-First
Every DPI capability must be exposed as a well-documented open API before any user interface is built. The API is the product.

*Why*: If DPI is designed around one app, it accidentally becomes an app monopoly, not infrastructure.

### 3. Open Standards, Open Specifications
Use internationally recognized open standards (OAuth 2.0, OIDC, ISO 20022, W3C VC, FHIR, etc.) for all interfaces. Publish specifications publicly.

*Why*: Private sector can build on known standards without waiting for the DPI operator. Competition begins immediately.

### 4. Federated Architecture
No single database of everything. Data stays at the source organization; DPI provides the routing, trust, and consent layer to enable sharing.

*Why*: Centralized databases are surveillance risks, single points of failure, and organizational anti-patterns (every agency fights over control).

### 5. Cloud-Agnostic Design
> **Reference**: https://docs.cdpi.dev/technical-notes/cloud-agnostic-deployment-of-dpi-and-data-portability

Design so that DPI can run on any cloud provider or on-premises. Use containers (Kubernetes), open-source databases, and portable IaC (Terraform).

*Why*: Sovereign control, negotiating power, resilience, and ability to respond to geopolitical changes. See dpi-cloud skill.

### 6. Privacy by Design
Minimize data collection. Require consent for data flows. Use tokenization (never expose root identifiers to service providers). Log all access.

*Why*: Public trust is the asset that makes DPI usable at scale. Once lost, it cannot be rebuilt quickly.

### 7. Inclusive by Design
Every DPI layer must be accessible to populations with low connectivity, low literacy, feature phones, and disabilities. Accessibility is not a late-stage add-on.

*Design tests*: Does it work on 2G? Does it work without a smartphone? Does it work for someone who cannot read?

## Technology Selection Framework

### Principle: Neutral Evaluation

Technology selection for DPI should be evaluated on these criteria, not on vendor relationships or political preferences:

| Criterion | Questions |
|-----------|-----------|
| **Openness** | Is the source code open? Is the specification public? |
| **Standards compliance** | Does it implement recognized international standards? |
| **Exit capability** | Can the country migrate to a different solution without data loss? |
| **Community / ecosystem** | Is there an active community beyond a single vendor? |
| **Proven at scale** | Has it been deployed at comparable scale in other contexts? |
| **Governance** | Who controls the roadmap? Is it community-governed? |
| **Local capacity** | Can local engineers learn, extend, and maintain it? |

### Build vs. Procure vs. Adopt Open Source

```
┌──────────────────┬─────────────────────────────────────────────┐
│ Situation        │ Recommendation                              │
├──────────────────┼─────────────────────────────────────────────┤
│ Open-source DPI  │ Adopt + customize; contribute back         │
│ platform exists  │ Avoid forks that can't be maintained       │
├──────────────────┼─────────────────────────────────────────────┤
│ Mature national  │ Modernize with open APIs; don't rebuild    │
│ system exists    │ unless it cannot be made interoperable     │
├──────────────────┼─────────────────────────────────────────────┤
│ No suitable      │ Commission custom build to open spec;      │
│ existing option  │ publish as open source from day 1         │
├──────────────────┼─────────────────────────────────────────────┤
│ Speed is         │ Procure managed service with open API;     │
│ critical         │ plan for future migration/insourcing       │
└──────────────────┴─────────────────────────────────────────────┘
```

### Identity Technology Selection

Evaluate against:
- Does it separate authentication from the underlying identity store?
- Does it support standard protocols (OIDC/OAuth 2.0)?
- Can service providers integrate without knowing which ID system is in use?
- Does it support multiple authentication factors (OTP, biometric, PIN, hardware token)?
- Does it support tokenization (partner-specific virtual IDs)?
- Can it scale to the full population simultaneously?

Reference implementations exist across multiple open-source and proprietary stacks. No single solution is right for all contexts.

### Payment Technology Selection

Evaluate against:
- Is the switching specification open (any licensed PSP can connect)?
- Does it mandate international standards (ISO 20022)?
- Is settlement final and near-real-time?
- Does it support alias-based routing (phone number → account)?
- Can it support G2P (government-to-person) disbursement?
- Does it have an interoperability path with neighboring countries / regional rails?

### Data Exchange Technology Selection

Evaluate against:
- Does consent travel with the data (not just as a policy)?
- Is the data access model federated (data stays at source)?
- Are the data schemas standardized and published?
- Can it work across sectors (health, finance, education) with the same consent model?
- Is there an audit trail for every data access event?

## Implementation Sequencing

### Recommended Logic (Not Mandatory Order)

```
Phase 1: Identity Foundation
  Purpose: Establish "who is this person?" as a verifiable, digital fact
  Key deliverable: Authentication API usable by government and private sector
  
Phase 2: Payments Rail
  Purpose: Enable money movement on top of verified identity
  Key deliverable: Interoperable instant payment switch with G2P capability
  
Phase 3: Data Exchange
  Purpose: Enable consented data flows that unlock new services
  Key deliverable: Consent framework + sector data APIs (financial first)
  
Phase 4: Credentials and Registries
  Purpose: Portable proofs of status; authoritative entity records
  Key deliverable: VC issuance for key documents; functional registries
  
Phase 5: Ecosystem Development
  Purpose: Unlock private innovation on the rails
  Key deliverable: Developer portal, sandbox, regulatory framework for innovators
```

### Why Identity First?
Authentication and verification are prerequisites for every other DPI layer. Payments without identity produces fraud at scale. Data exchange without identity produces unauthorized access. Identity is the anchor.

### Starting with Payments Only
Some countries start with payments (mobile money interoperability) and add identity later. This is viable but creates a bootstrapping challenge: you need identity for high-value transactions and for government use cases. Plan the identity layer early even if it deploys second.

## Procurement Principles

### DPI Procurement Is Different from Standard IT Procurement

Standard IT procurement optimizes for delivering a specific system. DPI procurement must also optimize for:
- **Ecosystem openness**: Can competitors of the winning vendor also build on the DPI?
- **Country control**: Can the country operate, modify, and migrate the DPI without the vendor?
- **Specification ownership**: Does the country own the API specification, or does the vendor?
- **Data ownership**: Is all data owned by the country, with the right to export in open formats?

### Contract Must-Haves
- [ ] Open-source requirement for core DPI components, OR source code escrow
- [ ] Country owns all data; right to export in open formats at any time
- [ ] Open API specification published under open license (not vendor's proprietary spec)
- [ ] No lock-in clauses that prevent migration to alternative providers
- [ ] Data Processing Agreement that satisfies local data protection law
- [ ] Technology transfer clause: local engineers must be trained and capable of operating the system
- [ ] Exit provisions: vendor must cooperate with migration to a successor

### Procurement Models

| Model | Description | When to Use |
|-------|-------------|-------------|
| **BOOT** (Build-Own-Operate-Transfer) | Vendor builds and operates; country takes ownership after 3-5 years | When country lacks initial capacity |
| **Managed Open Source** | Vendor supports an open-source platform | Fastest to production with lowest lock-in |
| **In-house build** | Country team builds on open-source components | When technical capacity exists |
| **Consortium** | Multiple countries share a platform and costs | Regional programs; lower unit cost |

## Governance Structure

### Separation of Powers (Non-Negotiable)

```
┌──────────────────────────────────────────────────────────┐
│  Policy / Regulatory Layer                               │
│  Sets rules, standards, access conditions                │
│  Must NOT operate the DPI                                │
├──────────────────────────────────────────────────────────┤
│  Operations Layer                                        │
│  Runs the infrastructure; enforces policy                │
│  Must NOT set policy; must be auditable                  │
├──────────────────────────────────────────────────────────┤
│  Ecosystem Layer                                         │
│  Builds services on DPI rails                           │
│  Must NOT control the rails; subject to regulation      │
└──────────────────────────────────────────────────────────┘
```

When the same entity is regulator + operator + ecosystem player, DPI becomes a monopoly. This has happened and must be actively prevented in governance design.

### Multi-Stakeholder Oversight
- Executive/Finance ministry: accountability and funding
- Technical ministry/authority: operational governance
- Central bank (for payments) / data protection authority (for data exchange)
- Civil society: privacy, inclusion, and rights advocates
- Industry: private sector ecosystem representatives
- Regular public reporting and external audits

## Common Pitfalls

### Architecture Pitfalls

| Pitfall | Consequence | Prevention |
|---------|-------------|-----------|
| Building use-case-specific infrastructure | Cannot be reused; creates tech debt | Design for unknown future uses |
| Centralized database of everything | Surveillance risk; monopoly power | Federate; data stays at source |
| No API versioning from day one | Breaking changes destroy ecosystem | Semver + deprecation policy built-in |
| Biometric-only authentication | Excludes elderly, disabled, injured | Multi-factor from the start |
| No offline mode | Fails rural and low-connectivity populations | 2G/USSD/SMS fallback required |
| Cloud-native lock-in (managed services for critical paths) | Cannot migrate; vendor holds DPI hostage | Cloud-agnostic design; open-source databases |
| Single-vendor DPI stack | Vendor dependency; no competition | Multi-vendor or open-source |

### Governance Pitfalls

| Pitfall | Consequence |
|---------|-------------|
| Single entity controls DPI | Political disruption; no accountability |
| Operator sets their own rules | Conflict of interest; DPI captured |
| No grievance mechanism | Citizens wrongly excluded with no remedy |
| No sunset clauses on emergency measures | Temporary powers become permanent |
| Skipping public consultation | Resistance, protests, adoption failure |
| Vendor owns the data | Country cannot exit; data held hostage |

## DPI Readiness Dimensions

| Dimension | Assessment Questions |
|-----------|---------------------|
| **Political** | Is there sustained political leadership? Across government change? |
| **Legal** | Is there a data protection framework? Clear liability rules? |
| **Institutional** | Who owns this? Is there a mandate and budget? |
| **Technical** | Is there a technical team capable of overseeing (not just procurement)? |
| **Ecosystem** | Are there private sector players willing to build on the rails? |
| **Inclusion** | Are marginalized populations considered in the design? |
| **Connectivity** | What is the baseline connectivity? How does DPI serve the unconnected? |

## Reference Resources

- **CDPI Documentation**: https://docs.cdpi.dev/
- **CDPI Cloud Guidance**: https://docs.cdpi.dev/technical-notes/cloud-agnostic-deployment-of-dpi-and-data-portability
- **CDPI VC Decision Framework**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/decision-framework-for-verifiable-credentials
- **DPI-AI Framework**: https://digitalpublicinfrastructure.ai/
- **DPGA Registry**: https://digitalpublicgoods.net/registry/
- **World Bank ID4D Toolkit**: https://id4d.worldbank.org/guide
- **ITU DPI**: https://www.itu.int/
