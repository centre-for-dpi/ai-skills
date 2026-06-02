---
name: dpi-discovery-fulfilment
description: Discovery & Fulfilment Networks as DPI — Beckn Protocol, ONDC, open commerce/mobility/health. Use when designing open network DPI or evaluating platforms-to-protocols transitions.
---

# Discovery & Fulfilment Networks as DPI

> **Reference**: https://docs.cdpi.dev/technical-notes/digital-payment-networks/discovery-and-fulfillment-networks

## The Third DPI Pillar

CDPI frames Discovery & Fulfilment as the **third pillar of DPI**, alongside Identity and Payments:

```
DPI Stack
  ├── Identity      — Who are you?
  ├── Payments      — Can you transact?
  └── Discovery &   — Can you find and receive any service
      Fulfilment      from any provider, on any app?
```

**The core problem**: Today's digital commerce operates within walled gardens. Uber for cabs, Amazon for goods, Practo for healthcare — each platform controls its own discovery, ordering, and fulfilment. Businesses must solve the entire transaction cycle themselves. This creates winner-take-all dynamics, excludes smaller providers, and locks citizens into specific apps.

**The DPI solution**: Move from platforms to open protocols. On an open network, any buyer on any app can transact with any seller on any other app — without a central intermediary owning the rails.

## Platforms to Protocols

> **Reference**: https://docs.cdpi.dev/technical-notes/discovery-and-fulfillment-networks/platforms-to-protocols

| Closed Platform Model | Open Protocol Model (DPI) |
|----------------------|--------------------------|
| Buyer and seller must use the same app | Any buyer app + any seller app can interoperate |
| Platform owns discovery, ordering, fulfilment | Protocol defines how; anyone implements |
| Winner-take-all: network effects entrench monopoly | Network effects benefit all participants equally |
| Platform extracts rent from every transaction | Marginal cost of infrastructure approaches zero |
| Small businesses must accept platform terms | Small businesses connect once, reach all buyers |

**Analogy**: Email (SMTP) vs proprietary messaging. Anyone with an email address can send to anyone else regardless of provider. SMTP is the protocol; Gmail, Outlook, ProtonMail are all implementations. Discovery & Fulfilment DPI does the same for commerce.

## The Beckn Protocol

**Beckn** is the open-source, technology-agnostic protocol that makes interoperable discovery and fulfilment possible. It was originated by FIDE (Foundation for Interoperability in Digital Economy), co-founded by Nandan Nilekani and Dr. Pramod Varma.

Think of Beckn as:
- **HTTP** for transactions (just as HTTP enables any browser to reach any website)
- **SMTP** for commerce (just as SMTP enables any email client to reach any mailbox)

**GitHub**: https://github.com/beckn  
**Specification**: https://becknprotocol.io  
**Developer docs**: https://developers.becknprotocol.io

### Four Network Participants

```
┌──────────────────────────────────────────────────────────────┐
│                   Beckn Open Network                         │
│                                                              │
│  ┌────────────┐   search/select   ┌──────────────────────┐  │
│  │    BAP     │ ←───────────────→ │      BPP             │  │
│  │ (Buyer App)│                   │ (Provider/Seller App) │  │
│  └────────────┘                   └──────────────────────┘  │
│         ↕ lookup                          ↕ register         │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Beckn Gateway (BG) — optional broadcast layer        │  │
│  └────────────────────────────────────────────────────────┘  │
│         ↕                                                     │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Network Registry — trust, addressing, discovery       │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

**1. BAP (Beckn Application Platform) — Buyer Side**
- Consumer-facing app (the interface citizens use)
- Initiates transactions: sends `search`, `select`, `init`, `confirm`
- Any app can be a BAP — a superapp, a sector app, a bank app

**2. BPP (Beckn Provider Platform) — Seller/Provider Side**
- Receives and fulfils orders
- Responds asynchronously: `on_search`, `on_select`, `on_init`, `on_confirm`
- Represents service providers: restaurants, hospitals, cab drivers, lenders

**3. Beckn Gateway (BG) — Optional Broadcast Layer**
- Optimises discovery by broadcasting BAP search requests to all relevant BPPs
- Queries the Registry to find matching BPPs for a domain/location
- Not in the data path for ordering and fulfilment — only for discovery

**4. Network Registry — Trust and Addressing**
- Authoritative list of all participants (BAPs and BPPs)
- Stores addressing, sector, and coverage area per participant
- Maintained by Network Facilitators
- Hierarchically organised via Beckn Global Root Registries (BGRRs) — analogous to DNS for transactions

### The Transaction Lifecycle (APIs)

The Beckn Protocol defines a complete transaction lifecycle across four stages:

```
Stage 1: DISCOVERY
  BAP → search         BPP ← on_search (catalog response)
  
Stage 2: ORDERING  
  BAP → select         BPP ← on_select (quote)
  BAP → init           BPP ← on_init (payment terms, checkout)
  BAP → confirm        BPP ← on_confirm (order confirmed)

Stage 3: FULFILMENT
  BAP → track          BPP ← on_track (real-time status)
  BAP → update         BPP ← on_update (modify order)
  BAP → cancel         BPP ← on_cancel (cancellation confirmation)
  BAP → status         BPP ← on_status (order status)

Stage 4: POST-FULFILMENT
  BAP → rating         BPP ← on_rating
  BAP → support        BPP ← on_support (helpdesk)
```

All communication is **server-to-server** and **digitally signed** — no central intermediary sits in the data path for ordering and fulfilment.

### How Discovery Works (Technical Flow)

```
1. Citizen opens any BAP app, searches for "cab from airport to city"
2. BAP sends search request → Beckn Gateway (or directly to known BPPs)
3. Gateway broadcasts to all registered BPPs (mobility sector, city X)
4. BPPs respond asynchronously with availability, pricing, ETAs
5. BAP aggregates responses, presents options to citizen
6. Citizen selects → BAP sends select to chosen BPP
7. BPP confirms availability, returns quote
```

Key: the BAP does **not** need to have a bilateral integration with each BPP. The Gateway and Registry handle routing. Any BPP that joins the network is instantly discoverable by all BAPs.

### How Fulfilment Works (Technical Flow)

```
1. Citizen confirms order → BAP sends confirm
2. BPP creates order, assigns provider (driver, doctor, restaurant)
3. BPP notifies BAP asynchronously of fulfilment status
4. BAP enables citizen to track in real-time (track → on_track loop)
5. On completion, post-fulfilment APIs handle rating, support, returns
```

Fulfilment involves micro-transactions at each step — each element of delivery (pickup, drop-off confirmation, payment capture) is a discrete protocol interaction.

## ONDC — Open Network for Digital Commerce

**ONDC (Open Network for Digital Commerce)** is the flagship government-led implementation of Beckn Protocol, initiated by India's DPIIT (Department for Promotion of Industry and Internal Trade). Often described as "**UPI for e-commerce**."

**The ONDC proposition**: A buyer on Paytm can order from a seller listed only on Meesho. Neither the buyer nor seller needs to use the same app. The network makes them interoperable.

**Scale (as of 2024)**: 7M+ orders processed; 370,000+ sellers; 588+ cities covered.

**Architecture**: ONDC's backend is built on Beckn Protocol as the base layer, with model specifications customised to the Indian commerce context (GST, Indian address formats, Hindi language, etc.).

**Participants**: Buyer apps (Paytm, Magicpin, Mystore), Seller apps (Meesho, GoFrugal, eSamudaay), Logistics providers (Shiprocket, Dunzo), all interoperating on the same network.

## Sector Applications

Open discovery and fulfilment networks apply across every sector where providers and consumers need to find each other:

### Open Mobility / Transport
- Book any cab, auto-rickshaw, bus, metro, or flight from any single app
- **Kochi Open Mobility Network**: first decentralized open mobility network globally
- **Namma Yatri** (Bengaluru): auto-rickshaws on open network; zero commission to platform

### Open Financial Services
- Borrowers discover and apply for loans from multiple banks/NBFCs/insurers in any financial app
- Insurance discovery across providers
- Investment product discovery

> **Payment mechanics**: Beckn handles discovery and ordering; the payment step within a Beckn transaction uses the underlying payment rail (UPI, PIX, mobile money, instant payment system). See the **dpi-payments** skill for payment rail architecture.

### Open Healthcare
- Discover telemedicine providers, book appointments, receive prescriptions
- Any health app → any hospital, clinic, or doctor on the network
- Post-consultation fulfilment: prescription to pharmacy, lab test booking

### Open Energy
- **Unified Energy Interface (UEI)**: unifies fragmented EV charging transactions into a decentralized network
- EV drivers discover and pay at any charging station from any app
- Energy trading on open networks

### Open Logistics
- Shippers discover logistics providers across the network
- Multi-modal logistics coordination (truck → rail → last-mile) on a single network

### Non-Personal Data Discovery
CDPI notes that a decentralized **non-personal data access network** can be built on Beckn-like protocols to enable discovery of datasets across public and private agencies through standard APIs — essentially a marketplace for open data.

## Beckn Protocol Technical Reference

### Message Format (JSON)
```json
{
  "context": {
    "domain": "mobility",
    "action": "search",
    "country": "IND",
    "city": "BLR",
    "version": "1.0.0",
    "bap_id": "taxi-app.example.com",
    "bap_uri": "https://taxi-app.example.com/beckn",
    "transaction_id": "uuid",
    "message_id": "uuid",
    "timestamp": "2024-01-01T00:00:00.000Z"
  },
  "message": {
    "intent": {
      "fulfillment": {
        "start": {
          "location": { "gps": "12.9715987,77.5945627" }
        },
        "end": {
          "location": { "gps": "12.9352403,77.6244674" }
        }
      }
    }
  }
}
```

### Signing Requests
All Beckn messages are signed using the sender's private key (Ed25519) and verified by the receiver using the public key published in the Registry:
```
Authorization: Signature keyId="bap-id|key-id|ed25519",
  algorithm="ed25519",
  created="1641287772",
  expires="1641291372",
  headers="(created) (expires) digest",
  signature="base64-encoded-signature"
```

### Sector-Specific Extensions
Beckn core + domain-specific extensions:
- `beckn:mobility` — for transport
- `beckn:retail` — for commerce (ONDC)
- `beckn:healthcare` — for health services
- `beckn:energy` — for EV charging
- Custom domain specs published by network facilitators

## Governance Model

### Network Facilitators
- Define the rules and specifications for their network (e.g., ONDC for retail India)
- Onboard and verify participants (BAPs and BPPs)
- Maintain the Network Registry
- Resolve disputes between participants

### FIDE (Foundation for Interoperability in Digital Economy)
- Stewards the Beckn Protocol specification
- Analogous to W3C for web standards
- Ensures the protocol remains open and not controlled by any single entity

### Participant Trust
- Every participant registered with public key in Registry
- All messages signed; receiver verifies signature against Registry
- Fraudulent messages (invalid signature) rejected at the protocol level

## Key Resources

- **Beckn Protocol**: https://becknprotocol.io
- **Beckn Developer Docs**: https://developers.becknprotocol.io
- **FIDE**: https://fide.org
- **ONDC**: https://ondc.org
- **CDPI Technical Note**: https://docs.cdpi.dev/technical-notes/digital-payment-networks/discovery-and-fulfillment-networks
- **Carnegie Endowment on Open Networks as DPI**: https://carnegieendowment.org/posts/2023/11/open-networks-a-dpi-embedded-approach-for-online-marketplaces
