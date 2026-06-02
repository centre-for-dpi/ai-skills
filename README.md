# DPI Skills for Claude Code

A collection of Claude Code skills covering Digital Public Infrastructure (DPI) knowledge. These skills enable Claude to act as a knowledgeable co-worker on DPI design, implementation, and strategy.

**Design principles**: Technology-neutral, vendor-neutral, globally applicable. Core framing: DPI as public rails that enable government scale *and* private sector innovation simultaneously.

## Installation

### Claude Code (CLI)

**One-line install** — clone the repo and copy all skills to Claude Code's skill directory:

```bash
git clone https://github.com/centre-for-dpi/ai-skills.git
cd ai-skills
for f in skills/*.md; do
  name=$(basename "$f" .md)
  mkdir -p ~/.claude/skills/"$name"
  cp "$f" ~/.claude/skills/"$name"/SKILL.md
done
```

Restart Claude Code (or start a new session) to pick up the new skills.

**Verify installation:**

```bash
ls ~/.claude/skills/ | grep dpi
```

You should see directories like `dpi-101`, `dpi-payments`, `verifiable-credentials`, etc.

**Use a skill** — type its name as a slash command in any Claude Code session:

```
/dpi-101
/dpi-payments
/verifiable-credentials
```

**Update to the latest skills:**

```bash
cd ai-skills
git pull
for f in skills/*.md; do
  name=$(basename "$f" .md)
  mkdir -p ~/.claude/skills/"$name"
  cp "$f" ~/.claude/skills/"$name"/SKILL.md
done
```

**Remove all DPI skills:**

```bash
ls skills/ | sed 's/\.md$//' | xargs -I{} rm -rf ~/.claude/skills/{}
```

---

### Claude Cowork

1. Download the latest `dpi-skills.plugin` file from [GitHub Releases](https://github.com/centre-for-dpi/ai-skills/releases).
2. Open Claude Cowork.
3. Drag the `.plugin` file into the Cowork window, or go to **Settings → Plugins → Install from file** and select the file.
4. Click **Accept** in the plugin preview to confirm installation.
5. Skills are immediately available — invoke them by name (e.g. `/dpi-101`).

**Update:** Download the latest release and repeat the install — Cowork will replace the previous version.

---

## Available Skills

### Foundations
| Skill | Description |
|-------|-------------|
| `/dpi-101` | What DPI is, the rails model, three-layer stack, global examples, governance |
| `/dpi-country-technical-recommendation` | CDPI guidance for countries adopting DPI — neutral tech selection, sequencing, procurement, governance |
| `/dpi-un-safeguards` | Universal DPI Safeguards Framework — 18 principles, 13 risks, 5 responsible authorities, 5 lifecycle stages, 300+ recommendations, 50-in-5, Accelerator |
| `/dpi-+1-approach` | The DPI +1 Approach — converting existing systems to DPI via small incremental additions (paper cert + QR = DPI); asynchronous adoption model, government-as-rails, country applications (Nepal, Bhutan) |

### DPI Layers
| Skill | Description |
|-------|-------------|
| `/dpi-digital-id` | Digital Identity — foundational/functional ID, eKYC, authentication protocols, privacy design |
| `/dpi-payments` | Payments DPI — instant payment rails, interoperability, ISO 20022, mobile money, G2P, CBDC |
| `/dpi-data-exchange` | Data Exchange — consent frameworks, DEPA, real-time vs async patterns, FHIR, open banking |
| `/dpi-data-sharing-models` | CDPI data sharing taxonomy — Verifiable Credentials, consent-based (DEPA), system-to-system APIs, anonymised open data |
| `/dpi-discovery-fulfilment` | Discovery & Fulfilment Networks — Beckn Protocol, ONDC, open commerce/mobility/health networks, platforms-to-protocols |
| `/dpi-trust-infra` | Trust Infrastructure — PKI, digital signatures, eIDAS, eSign, eConsent, HSMs, consent artefacts |

### Technology
| Skill | Description |
|-------|-------------|
| `/verifiable-credentials` | W3C VC Data Model, DIDs, OpenID4VC, SD-JWT, CDPI decision framework, selective disclosure |
| `/dpi-architect` | DPI architecture — API-first rails design, microservices, security, scalability, reference architectures |
| `/dpi-cloud` | Cloud DPI — CDPI cloud-agnostic strategy, Kubernetes, IaC, sovereign cloud, multi-region |
| `/dpi-registries` | DPI Registries — civil registration, CRVS, functional registries, Sunbird RC, OpenG2P |

### DPI-AI Framework
| Skill | Description |
|-------|-------------|
| `/dpi-ai-framework` | Overview of AI for DPI — governance, responsible AI, privacy-preserving AI |
| `/dpi-ai-blocks` | AI building blocks — perception, reasoning, generation, memory, safety layers for DPI |
| `/dpi-ai-workflows` | AI workflow patterns — sync/async orchestration, LangGraph, Temporal, DPI tool integration |
| `/dpi-public-agents` | Public sector AI agents — agent architecture, DPI tool use, governance, multi-agent patterns |
| `/dpi-mcp-dpg` | MCP-ready DPGs — exposing DPI systems as MCP servers, tool schemas, security, DPG registry |
| `/dpi-ai-use-cases` | AI use cases for DPI — health, financial inclusion, welfare, agriculture, governance |

### Digital Public Goods (DPGs)
| Skill | Description |
|-------|-------------|
| `/dpi-inji` | Inji Wallet, Inji Certify, Inji Verify — MOSIP VC ecosystem, OpenID4VCI, offline QR |
| `/dpi-e-signet` | e-Signet — OIDC gateway, eKYC, auth plugins, QR login, SP integration (MOSIP) |
| `/dpi-credebl` | CREDEBL — LF Decentralized Trust VC platform, multi-tenant, AnonCreds, OpenID4VC |
| `/dpi-x-road` | X-Road — Estonia's federated data exchange, Security Server, G2G/G2B interoperability |
| `/dpi-opencrvs` | OpenCRVS — open-source CRVS DPG, FHIR data model, birth/death/marriage registration, national ID integration |
| `/dpi-openfn` | OpenFn — DPG workflow automation for DPI data exchange (DHIS2, OpenMRS, registries) |
| `/dpi-n8n` | n8n — Visual AI agent workflows, 400+ nodes, self-hostable, LangChain-based agents |
| `/dpi-superapp` | DPI Superapp — unified citizen apps, mini-app architecture, service orchestration |

### Sectoral DPI
| Skill | Description |
|-------|-------------|
| `/dpi-health` | DPI for Healthcare — health identity, HIE, OpenHIE, FHIR R4, consent-based records, telemedicine, disease surveillance, open-source health DPGs |
| `/dpi-childhood` | DPI for Children — UNICEF framework, child identity safeguards, birth registration, biometric limitations, child-safe design principles, social protection |

### Use Cases
| Skill | Description |
|-------|-------------|
| `/dpi-use-cases` | End-to-end DPI use cases — health, financial inclusion, social protection, agriculture, education |


## Usage

Invoke any skill in Claude Code with its name:
```
/dpi-101
/dpi-architect
/dpi-mcp-dpg
/verifiable-credentials
```

## Skill Map

```
┌──────────────────────────────────────────────────────────┐
│                   DPI Skills Map                         │
│                                                          │
│  Foundations                                            │
│    /dpi-101  /dpi-country-technical-recommendation      │
│                                                          │
│  DPI Layers           Technology                        │
│    /dpi-digital-id      /verifiable-credentials         │
│    /dpi-payments        /dpi-architect                  │
│    /dpi-data-exchange   /dpi-cloud                      │
│    /dpi-data-sharing-models  /dpi-registries            │
│    /dpi-discovery-fulfilment                            │
│    /dpi-trust-infra                                     │
│                                                          │
│  DPI-AI Framework                                       │
│    /dpi-ai-framework    /dpi-ai-blocks                  │
│    /dpi-ai-workflows    /dpi-public-agents              │
│    /dpi-mcp-dpg         /dpi-ai-use-cases               │
│                                                          │
│  Digital Public Goods (DPGs)                            │
│    /dpi-inji   /dpi-e-signet   /dpi-credebl             │
│    /dpi-x-road /dpi-superapp                            │
│                                                          │
│  Use Cases                                              │
│    /dpi-use-cases                                       │
└──────────────────────────────────────────────────────────┘
```

## Key References

- **CDPI Documentation**: https://docs.cdpi.dev/
- **CDPI Cloud Guidance**: https://docs.cdpi.dev/technical-notes/cloud-agnostic-deployment-of-dpi-and-data-portability
- **CDPI VC Decision Framework**: https://docs.cdpi.dev/technical-notes/data-and-credentialing-infra/decision-framework-for-verifiable-credentials
- **DPI Architecture Principles**: https://docs.cdpi.dev/the-dpi-wiki/dpi-tech-architecture-principles
- **DPI-AI Framework**: https://digitalpublicinfrastructure.ai/
- **Inji Documentation**: https://docs.inji.io/
- **CREDEBL (LF Decentralized Trust)**: https://www.lfdecentralizedtrust.org/projects/credebl
- **X-Road**: https://x-road.global/
- **OpenFn (DPG workflow automation)**: https://openfn.org | https://docs.openfn.org
- **n8n (visual AI workflow automation)**: https://n8n.io | https://docs.n8n.io
