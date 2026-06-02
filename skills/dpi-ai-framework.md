---
name: dpi-ai-framework
description: CDPI DPI-AI Framework (2026) — AI as external interoperable capabilities for DPI. Covers AI Blocks, DPI Workflows, Public Agents, 9-step implementation, and AI governance.
---

# DPI-AI Framework

> **Framework Home**: https://digitalpublicinfrastructure.ai/  
> **Vision Paper**: https://digitalpublicinfrastructure.ai/paper.php  
> **Framework Overview**: https://digitalpublicinfrastructure.ai/framework.php  
> **Implementation Guide**: https://digitalpublicinfrastructure.ai/implementation.php  
> **DPGs as AI Blocks**: https://digitalpublicinfrastructure.ai/dpgs.php  
> **Authors**: Daniel Abadie & Antony Muriithi, CDPI (2026)

## The DPI-AI Framework: Core Thesis

> *"The aim is not to promote AI adoption for its own sake, but to provide a clear and practical way to align emerging AI capabilities with the foundational rails of DPI towards development outcomes."* — DPI-AI Framework

> *"The value of AI emerges not from its autonomy but from how it integrates dynamically with the foundational rails of DPI to create adaptive, intelligent, and publicly governed digital infrastructure."*

### The Problem the Framework Solves

Governments face a dual challenge: rapid AI advances and persistent fragmentation in public digital systems.

> *"Without interoperable and publicly governed AI capabilities, ministries duplicate investments, procurement becomes fragmented, and crisis responses slow down."*

### AI's Relationship to DPI (Critical Positioning)

> *"The DPI-AI Framework positions AI not as a new layer within DPI, but as an external and interoperable set of capabilities that connect to DPI through shared standards, governance mechanisms, and safeguards."*

| Old framing | DPI-AI Framework framing |
|------------|------------------------|
| AI as the "fourth DPI layer" | AI as external + interoperable capabilities |
| AI integrated inside DPI systems | AI Blocks callable from outside via open standards |
| Vertically integrated AI + DPI | Modular, replaceable, governed AI components |
| Vendor AI on DPI data | Sovereign, governable AI on sovereign infrastructure |

## The Three-Element Framework

Three complementary, interoperable elements:

```
Public Agent  (citizen-facing, constrained interface)
      ↓
DPI Workflow  (locus of control — YAML-specified, pre-approved, audited)
      ↓
AI Block  +  DPI Rail API
(AI capability)  (authoritative action)
```

| Element | Role | Key Constraint |
|---------|------|---------------|
| **AI Blocks** | Modular AI capabilities; DPGs exposed via API/MCP | Open-source preferred, replaceable, must declare training data provenance |
| **DPI Workflows** | Orchestrate AI Blocks + DPI rail calls + consent + human review | Pre-approved; AI operates within, never outside |
| **Public Agents** | Citizen-facing interface via WhatsApp/USSD/voice/web | Not autonomous; cannot access data without consent; always offers human escalation |

> **See**: `dpi-ai-blocks`, `dpi-ai-workflows`, `dpi-public-agents` for detailed coverage of each element.

## Implementation Guide: 9 Steps

> **Reference**: https://digitalpublicinfrastructure.ai/implementation.php

**Governance comes first — before any code.**

### Phase 1: Governance Readiness (Steps 1–3)

**Step 1: Define the governance baseline**

Policy and legal teams decide before any engineering begins:
- Who owns the service legally?
- What confidence threshold triggers human review?
- How do citizens escalate if the AI is wrong?
- How is PII handled in logs?
- Where is consent captured and stored?

**Step 2: Select a high-impact, low-risk pilot use case**

| Good pilot candidates | Avoid for pilots |
|---------------------|----------------|
| Benefit inquiry chatbot | Automated criminal justice decisions |
| Document verification | High-value irreversible payments |
| FAQ and service routing | Medical diagnosis |
| Immunisation reminder | Asylum determinations |

**Step 3: Map existing DPI rails**

Identify available DPI APIs: identity verification, data exchange, payment rail, consent management, notification channels.

### Phase 2: Build (Steps 4–6)

**Step 4: Specify the DPI Workflow in YAML (governance document first)**

```yaml
# YAML is both the governance document AND the deployment spec
workflow:
  id: maternal_benefit_eligibility_v1
  approved_by: Ministry of Social Protection
  legal_basis: Social Welfare Act 2024, Section 12
  
  governance:
    confidence_threshold_auto_approve: 0.90
    confidence_threshold_human_review: 0.70
    human_escalation_sla_hours: 48
    pii_redaction_in_logs: true
    consent_scope: maternal_benefit_eligibility
    audit_retention_days: 2555
  
  steps:
    - authenticate → fetch_income → assess_eligibility → [human_review?] → enroll
```

**Step 5: Select and configure AI Blocks** — choose DPGs as AI Blocks where possible (MOSIP, OpenFn, Sunbird RC, DHIS2).

**Step 6: Deploy on workflow engine**

| Team type | Approach |
|----------|---------|
| Code-first | Run YAML spec on OpenFn, n8n, or Prefect |
| No-code | Build workflow visually in n8n or OpenFn |
| Mixed | Policy team writes YAML; engineers implement |

> *"The deployment must match the governance spec — the deployment is an execution of the document, not a departure from it."*

### Phase 3: Test and Pilot (Steps 7–8)

**Step 7: Structured testing** — happy path, edge cases, failure modes, all languages and channels; validate escalation and audit log completeness.

**Step 8: Narrow pilot** — specific region or user group before national rollout; monitor escalation rate, accuracy, citizen satisfaction.

### Phase 4: Scale and Govern (Step 9)

**Step 9: Continuous governance** — quarterly bias audits, drift detection, grievance mechanisms, public transparency report, shared workflow templates across agencies and governments.

---

## AI as an Enabler and Risk for DPI

AI transforms DPI both as a capability multiplier and as a governance challenge:

**AI amplifies DPI by:**
- Making DPI services accessible via natural language (voice/text bots)
- Automating document processing and fraud detection at scale
- Enabling predictive service delivery
- Reducing language/literacy barriers (multimodal AI)

**AI introduces new risks:**
- Algorithmic discrimination in identity verification, credit, welfare
- Privacy violations through inference and re-identification
- Opacity ("black box") in high-stakes decisions
- Concentration of AI power contradicting DPI's open principles
- Bias amplification affecting marginalized populations

## The DPI-AI Stack

```
┌─────────────────────────────────────────────┐
│           AI Applications Layer              │
│  (Chatbots, fraud detection, analytics)      │
├─────────────────────────────────────────────┤
│           AI Infrastructure Layer            │
│  (Models, compute, data pipelines)           │
├─────────────────────────────────────────────┤
│              DPI Layers                      │
│  Identity | Payments | Data Exchange         │
└─────────────────────────────────────────────┘
```

AI can operate at each DPI layer:
- **On top of DPI**: AI services built using DPI APIs
- **Within DPI**: AI improving DPI operations (fraud detection, deduplication)
- **Alongside DPI**: AI as a new fourth DPI layer (AI commons)

## DPI-AI Framework: Key Themes

The framework covers:
- **Three-element model**: AI Blocks + DPI Workflows + Public Agents as the practical vocabulary for AI in public sector
- **Governance-first deployment**: Policy and legal decisions before engineering; YAML governance specs
- **Sovereignty**: 3–8B SLM on sovereign infrastructure over frontier LLM APIs for production
- **DPGs as AI Blocks**: Open-source DPGs (MOSIP, OpenFn, Mojaloop, DHIS2) callable via API/MCP
- **Reusable workflow templates**: Shared across governments; country-configurable
- **9-step implementation playbook**: From governance baseline to continuous monitoring
- **Multi-agent orchestration**: Multiple agents collaborating via shared DPI rails

## AI Governance for DPI

### Core Principles (aligned with OECD AI Principles, EU AI Act)

1. **Human oversight**: High-stakes AI decisions must be reviewable by humans
2. **Explainability**: Decisions affecting rights must be explainable
3. **Non-discrimination**: AI must not discriminate on protected characteristics
4. **Privacy**: AI must not enable surveillance beyond its purpose
5. **Accountability**: Clear responsibility chains for AI decisions
6. **Robustness**: AI must be reliable and secure
7. **Transparency**: Public disclosure of AI use in government services

### EU AI Act Risk Categories (relevant to DPI)

| Risk Level | DPI Examples | Requirements |
|------------|-------------|-------------|
| **Unacceptable** | Real-time biometric surveillance in public spaces, social scoring | Banned |
| **High Risk** | Biometric ID, welfare eligibility, credit scoring, law enforcement | Conformity assessment, logs, human oversight |
| **Limited Risk** | Chatbots, document readers | Transparency disclosure |
| **Minimal Risk** | Anti-spam, route optimization | Self-governance |

### India's DPI AI Approach
- **DPDP Act 2023**: Data protection covering AI processing of personal data
- **India AI Mission**: ₹10,372 Cr ($1.25B) for AI infrastructure
- **MOSIP AI Vision**: AI for deduplication quality, fraud detection in ID
- **IndiaAI**: Government AI compute/dataset/startup initiative

## AI Use Cases Across DPI Layers

### Identity + AI

**Biometric Quality Enhancement**
- AI-based image quality assessment at enrollment
- Super-resolution for low-quality captures
- Liveness detection (anti-spoofing)
- Face reconstruction for masked/aged faces

**Identity Fraud Detection**
- Synthetic identity detection
- Document authenticity verification
- Anomaly detection in authentication patterns
- Velocity and behavioral biometrics

**Accessible Enrollment**
- OCR for document processing
- Voice-guided enrollment for illiterate populations
- Multi-language support (100+ Indian languages for MOSIP)
- Exception handling for biometric failures

### Payments + AI

**Fraud Prevention**
- Real-time transaction scoring (milliseconds)
- Graph analysis for mule account networks
- Anomaly detection in UPI/instant payment flows
- Consortium fraud models (shared across banks)

**Financial Inclusion**
- Alternative credit scoring (telco data, utility payments)
- Cash flow analysis for thin-file borrowers
- Micro-insurance risk assessment
- Agricultural loan underwriting from satellite + weather data

**Reconciliation and Disputes**
- Automated dispute resolution
- Reconciliation anomaly detection
- G2P payment failure prediction

### Data Exchange + AI

**Consent Interface**
- Natural language consent explanations
- Risk-based consent prompting
- Anomaly detection in consent patterns

**Data Quality**
- Automated data validation
- Schema mapping and normalization
- Missing data imputation
- Duplicate detection across datasets

**Analytics on Consented Data**
- Federated learning (model training without centralizing data)
- Differential privacy for aggregate analytics
- Synthetic data generation for testing

## Conversational AI for DPI Services

### Multilingual Government Chatbots
Key design requirements:
- **Language**: Support regional languages, code-switching (mixing languages)
- **Literacy**: Voice-first for low-literacy users
- **Accessibility**: Works on feature phones (IVR + basic SMS)
- **Domain knowledge**: DPI-specific fine-tuning (welfare schemes, ID processes)
- **Fallback**: Clear escalation to human agent
- **Privacy**: Minimal PII retention; clear data handling

### Architecture Pattern
```
User (Voice/Text) → ASR (if voice) → NLU/LLM → Intent Resolution
                                         ↓
                               DPI API Calls (ID Auth, Payments, Data)
                                         ↓
                               Response Generation → TTS (if voice) → User
```

### LLM Integration with DPI APIs
```python
# Pattern: LLM as orchestrator, DPI as tool
# (tool names are illustrative — adapt to your country's DPI APIs)
tools = [
    {"name": "verify_identity", "description": "Verify user via national identity OTP/biometric"},
    {"name": "check_balance", "description": "Check beneficiary balance on social registry"},
    {"name": "initiate_payment", "description": "Initiate G2P payment via payment rail"},
    {"name": "fetch_health_record", "description": "Fetch health record with consent via health DPI"}
]
# LLM decides which DPI API to call based on user intent
```

## Privacy-Preserving AI Techniques

### Federated Learning
- Train AI models across distributed data (hospitals, banks) without centralizing data
- Each node trains locally; only model updates (gradients) are shared
- Used for: fraud detection consortiums, health AI across hospitals

### Differential Privacy
- Add calibrated noise to query results so individuals cannot be re-identified
- Used for: aggregate statistics on sensitive DPI data
- Key parameter: ε (epsilon) — privacy budget; lower = more private, less accurate

### Homomorphic Encryption
- Computation on encrypted data without decryption
- Still largely impractical for large-scale ML, but useful for specific operations
- Used for: privacy-preserving credit scoring prototypes

### Secure Multi-Party Computation (MPC)
- Multiple parties compute a joint function without revealing inputs
- Used for: cross-institutional fraud detection without data sharing

## AI Infrastructure for DPI

### DPI AI Commons (Proposed)
Countries building shared AI infrastructure as public goods:
- **Shared compute**: GPU clusters for government AI workloads
- **Shared datasets**: Privacy-preserving public datasets
- **Shared models**: Foundation models fine-tuned on local languages/contexts
- **Model hub**: Government-vetted models for high-stakes use cases

### India AI Mission Components
- **Compute**: 10,000 GPU compute capacity for startups and research
- **Datasets**: Non-personal/anonymized government datasets for AI training
- **Startup**: AI startup funding and acceleration
- **Research**: AI research institutes (IITs, IISc)
- **Talent**: AI skilling programs

### Cloud and Compute Considerations
- Sovereign AI: Training on national data, hosted in-country
- Energy costs: AI compute is energy-intensive — sustainability matters
- Open-weight models: LLaMA, Mistral, Gemma for on-premise deployment
- Inference optimization: Quantization, distillation for edge deployment

## Responsible AI Checklist for DPI Projects

### Pre-Deployment
- [ ] Bias audit across demographic groups (gender, caste, income, geography)
- [ ] Privacy Impact Assessment (PIA)
- [ ] Data lineage documented
- [ ] Model explainability mechanism in place
- [ ] Human oversight process defined for edge cases
- [ ] Adversarial robustness tested
- [ ] Legal basis for AI processing established (DPDP/GDPR/etc.)

### Deployment
- [ ] Model monitoring in production
- [ ] Drift detection and retraining triggers
- [ ] Grievance redressal for AI-affected decisions
- [ ] Audit log of AI decisions (immutable)
- [ ] Kill switch / fallback to non-AI processing

### Post-Deployment
- [ ] Regular bias re-audits (quarterly)
- [ ] Outcome monitoring (did AI actually improve service delivery?)
- [ ] Public transparency report
- [ ] Continuous improvement feedback loop

## Key Frameworks & Standards

| Framework | Scope |
|-----------|-------|
| **EU AI Act** | Binding regulation; risk-based approach |
| **OECD AI Principles** | Non-binding; adopted by 50+ countries |
| **UNESCO AI Ethics Recommendation** | Global soft law; 193 member states |
| **NIST AI RMF** | Risk management framework for AI |
| **ISO/IEC 42001** | AI management system standard |
| **India DPDP Act 2023** | Data protection including AI |
| **ITU AI for Good** | Global AI for SDG alignment |
| **G7 Hiroshima AI Process** | Responsible AI for frontier models |

## DPI + Generative AI Considerations

### Opportunities
- Auto-fill government forms from documents
- Multilingual service delivery at scale
- Intelligent routing and triage of citizen queries
- Policy summarization and accessibility

### Risks
- Hallucination in high-stakes government information
- Deepfake threats to biometric identity systems
- LLM training on sensitive government data
- Prompt injection attacks in citizen-facing chatbots

### Mitigation
- RAG (Retrieval-Augmented Generation) over authoritative DPI documentation
- Structured output formats (JSON) rather than free text for DPI API calls
- Sandboxed execution for LLM tool use
- Human-in-the-loop for irreversible actions (payments, enrollment)
