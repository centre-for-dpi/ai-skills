---
name: dpi-+1-approach
description: DPI +1 Approach — converting existing systems to DPI via minimal additions. Use when advising on pragmatic DPI deployment, digital transformation strategy, or incremental modernisation.
---

# The DPI +1 Approach

> **Reference**: https://docs.cdpi.dev/the-dpi-wiki/inputs-for-designing-a-dpi-informed-digital-transformation-strategy  
> **Source**: CDPI DPI Wiki — Inputs for Designing a DPI-Informed Digital Transformation Strategy

## Core Philosophy

> *"Rather than launching grand new projects or initiatives, ask what small (+1) changes across systems and ideas that already exist can help convert an existing digital system to a DPI to improve feasibility."*

The +1 Approach is a pragmatic lens for digital transformation: instead of replacing what exists, identify the minimum viable addition that unlocks DPI properties — interoperability, verifiability, population scale, open access.

**The canonical example:**

> *"An existing paper certificate can operate as DPI for a digital economy with a digitally signed QR code!"*

A paper document + a digitally signed QR code becomes:
- Machine-verifiable (any QR scanner can verify authenticity)
- Interoperable (standard QR, standard signature scheme)
- Population-scale (no app required, no connectivity required at point of verification)
- DPI — without replacing the paper certificate at all

## Key +1 Examples

| Existing System | +1 Addition | DPI Capability Unlocked |
|----------------|-------------|------------------------|
| Paper certificate | Digitally signed QR code | Machine verifiability, eKYC, offline verification |
| Physical ID card (30% coverage) | Digitally signed QR code | e-authentication, eKYC, single sign-on |
| Government department database | Open API endpoint | Interoperability, third-party integration |
| SMS notification system | Structured data payload | Machine-readable events, automated workflows |
| Existing payment network | Interoperable QR (EMVCo) | Merchant acceptance without new hardware |
| Civil registry | FHIR export layer | Health DPI integration, deduplication |

> *"If there is a physical ID card with 30% coverage of population, simply adding a digitally signed QR code to it would make it possible to include features like e-authentication, eKYC, or single-sign as well."*

## The Three Goals of DPI-Informed Transformation

The +1 approach is structured around three goals that balance government pragmatism, equity, and market growth:

### Goal 1: Asynchronous Quick Wins
Government can work towards **asynchronous quick wins** leveraging existing infrastructure with minimal cost, time, and effort investments — without requiring full-system overhauls or cross-ministry consensus before starting.

### Goal 2: Universal Equitable Access
All individuals — irrespective of diversity in financial, social, educational, or accessibility backgrounds — can **equitably access** the digital rails. The +1 approach avoids the trap of building systems that work only for the already-connected.

### Goal 3: Private Market Innovation
Private market participation is incentivised to allow **innovation over government-sanctioned DPI rails** to simultaneously support economic growth. The government builds rails; the market builds applications.

## Asynchronous Adoption Model

Traditional digital transformation requires cross-ministry alignment before launch. The +1 approach enables **asynchronous adoption**:

```
Start with first-mover departments
    → departments with forward-looking individuals
    → aligned with the +1 approach
    → focusing on DPI blocks that match their mandates and budgets

Wait for proof of value to emerge
    → other departments observe working examples
    → join the rails asynchronously as they see value

Scale without central coordination
    → each department's +1 addition strengthens the shared rail
    → network effects compound over time
```

> *"Proof of value, starting with first-mover departments who have forward-looking individuals aligned with the approach, focusing on DPI blocks that align with their mandates and budgets, and allowing other departments to asynchronously join the rails as they see the proof of value."*

This model avoids:
- Big-bang transformation failures
- Single ministry blocking entire national rollout
- Budget cycles slowing ecosystem-wide adoption

## Government Role: Rails, Not Applications

A core design principle of the +1 approach is a clear separation of responsibilities:

| Role | Who | Examples |
|------|-----|---------|
| **Minimalistic rails** | Government | Digital ID auth API, payment switch, consent registry, credential issuance |
| **Applications (innovation layer)** | Market players | Welfare apps, agritech, fintech, healthtech |
| **Adoption layer** | Market + civil society | User-facing services built on government rails |

> *"The government builds only the minimalistic rails and not necessarily the applications (innovation layer), while market players can build their own apps on top of the rails (adoption layer)."*

The government's job is to make the rail trustworthy, open, and interoperable — then step back and let market participants compete on the application layer.

## DPI Blocks as +1 Components

> *"Many DPI blocks are small +1 components that can be added to existing infrastructure in the country to convert them into high-functioning population scale resources."*

DPI blocks suitable for +1 deployment:

| DPI Block | +1 to Existing System | Result |
|-----------|----------------------|--------|
| Digitally signed QR | Paper certificate / physical ID | Verifiable credential at population scale |
| eKYC API | Existing ID database | Real-time identity verification for any service |
| Consent layer | Existing health / financial records | Data portability with citizen control |
| G2P payment rail | Existing beneficiary registry | Direct bank transfer replacing cash distribution |
| Open credential API | Existing educational / health records | Cross-sector credential portability |
| Financial address mapper | Existing bank account system | Interoperable payments without account number sharing |

## Avoiding Common Design Architecture Errors

The +1 approach explicitly anticipates failure modes that emerge when governments try to build too much too fast:

**Common errors the +1 approach guards against:**

1. **Boiling the ocean**: Requiring complete system replacement before any DPI value is delivered
2. **Vertical integration trap**: Building a closed application instead of an open rail
3. **Single-ministry dependency**: Gating population-scale rollout on cross-government consensus
4. **Technology-first design**: Picking platforms before defining governance and use cases
5. **Coverage fallacy**: Assuming 100% digital coverage is required before DPI can function

> *"CDPI also peer reviewed DPI-led digital government blueprints, providing recommendations on rapid deployment options using +1 approach, ways to anticipate frequent design architecture errors from the start (Nepal, Bhutan)."*

## Country Applications

### Nepal & Bhutan
CDPI peer-reviewed DPI-led digital government blueprints for both countries, providing:
- Rapid deployment options using the +1 approach
- Recommendations for anticipating frequent design architecture errors from the outset
- Sequencing guidance for incremental DPI build-out

## Applying the +1 Lens: Questions to Ask

When advising a government on digital transformation:

1. **What already exists?**  
   Map the current system: paper records, digital databases, ID cards, payment networks, notification systems.

2. **What is the minimum +1 addition?**  
   Identify the smallest change that unlocks a DPI property (verifiability, interoperability, population scale, open access).

3. **Which department can move first?**  
   Find the forward-looking ministry or department with mandate and budget alignment — don't wait for universal buy-in.

4. **What is the proof of value?**  
   Define what success looks like so that other departments can observe and join asynchronously.

5. **Is government building rails or applications?**  
   If government is building applications, it is probably doing too much. Redirect to the minimalistic rail.

6. **Does the +1 work for everyone?**  
   Test against low-literacy, low-connectivity, low-income, and accessibility edge cases — if it fails them, the +1 is not yet DPI.

## Relationship to DPI Design Principles

The +1 approach operationalises the core DPI design principles:

| DPI Principle | +1 Approach Expression |
|--------------|----------------------|
| **Open standards** | The +1 addition uses open, non-proprietary formats (QR, JWT, REST) |
| **Interoperability** | The +1 makes existing systems callable by other systems |
| **Minimalism** | Government adds only what the market cannot or will not build |
| **Inclusion** | The +1 works for people without smartphones, internet, or literacy |
| **Sovereignty** | The +1 keeps data and control within the public infrastructure |
| **Reversibility** | The +1 can be removed or replaced without breaking the underlying system |
