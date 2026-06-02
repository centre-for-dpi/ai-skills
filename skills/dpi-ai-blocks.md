---
name: dpi-ai-blocks
description: AI Blocks for DPI — Foundational (translate/OCR/speech) and Sector-Specific (eligibility/credential/registry). Use when selecting or designing AI capabilities for DPI services.
---

# DPI AI Building Blocks

> **Reference**: https://digitalpublicinfrastructure.ai/ai-blocks.php  
> **DPGs as AI Blocks**: https://digitalpublicinfrastructure.ai/dpgs.php  
> **DPI-AI Framework**: https://digitalpublicinfrastructure.ai/

## DPI-AI Framework: What is an AI Block?

> *"AI Blocks are modular units of AI capability that can be invoked as callable functions. They are designed to perform both sector-specific functions — such as identity verification, registry creation, credential issuance, or decision support — and foundational, bounded tasks such as classification, summarisation, and translation."* — DPI-AI Framework

AI Blocks are the **second element** of the DPI-AI Framework (alongside DPI Workflows and Public Agents):

```
Public Agent  →  interprets citizen intent
      ↓
DPI Workflow  →  orchestrates the sequence (the locus of control)
      ↓
AI Block      →  provides the AI capability (foundational or sector-specific)
      ↓
DPI Rail API  →  executes the authoritative action (identity / payment / registry)
```

**Key distinction**: AI Blocks provide intelligence. DPI Rails provide authority. An AI Block reasons about eligibility — the DPI registry records it.

---

## The Two Types of AI Block

> *"Foundational and Sector-Specific blocks are complementary. Foundational blocks provide the universal primitives that ensure accessibility and reuse, while Sector-Specific blocks extend them to meet the needs of specific programs and workflows."*

### Type 1: Foundational AI Blocks

**Foundational AI Blocks** are general-purpose capabilities reusable across many sectors and services. They provide the baseline multimodal and local-language primitives that are prerequisites for an inclusive, AI-ready nation.

| Foundational Block | Function | Signature |
|-------------------|---------|----------|
| **Translation / Transliteration** | Convert text between languages and scripts | `translate(text, from_lang, to_lang)` |
| **Speech-to-Text (ASR)** | Voice input in local languages and dialects | `speech_to_text(audio, language)` |
| **OCR / Document Extraction** | Extract structured data from scanned documents | `ocr_extract(image, schema)` |
| **Summarisation** | Condense long policy documents or case files | `summarise(text, max_length, audience)` |
| **Classification** | Route intents, triage cases, categorise inputs | `classify(text, taxonomy)` |
| **Text-to-Speech (TTS)** | Speak responses in local languages | `text_to_speech(text, language, voice)` |
| **Named Entity Recognition** | Extract names, IDs, dates, amounts from text | `extract_entities(text, entity_types)` |

**Design principle**: Foundational blocks are **sector-agnostic** — the same `translate()` block serves a health chatbot, a welfare eligibility workflow, and a land registry query. Build once, reuse everywhere.

**Examples in practice**:
- A health chatbot uses `speech_to_text()` + `translate()` to accept voice input in a regional dialect
- A welfare eligibility workflow uses `ocr_extract()` to read a scanned income certificate
- A farmer advisory service uses `text_to_speech()` to deliver responses in a local language over IVR

### Type 2: Sector-Specific AI Blocks

**Sector-Specific AI Blocks** are tailored to a domain or workflow. They embed the policy and operational logic of a particular sector into callable functions that serve specific public needs.

| Sector | Block | Function | Signature |
|--------|-------|---------|----------|
| **Identity** | Eligibility Verify | Check if citizen meets criteria for a program | `eligibility_verify(citizen_id, program_id, data_context)` |
| **Identity** | Document Verify | Authenticate a document's genuineness | `document_verify(document_image, document_type)` |
| **Credentials** | Credential Issue | Issue a verifiable credential to a citizen | `credential.issue(citizen_id, credential_type, attributes)` |
| **Registries** | Registry Search | Query a functional registry for a citizen record | `registry.search(registry_name, query_params)` |
| **Health** | Symptom Triage | AI triage of reported symptoms → severity + referral | `symptom_triage(symptoms, patient_context)` |
| **Health** | Immunisation Due | Determine which vaccines are due for a child | `immunisation_due(child_id, immunisation_record)` |
| **Agriculture** | Crop Risk Score | Satellite + weather → crop failure risk for insurance | `crop_risk_score(location, crop_type, season)` |
| **Finance** | Alternative Credit | Thin-file credit score from transaction + utility data | `alt_credit_score(citizen_id, data_sources)` |
| **Social Protection** | Ghost Beneficiary | Flag deceased or duplicate beneficiaries in a registry | `ghost_beneficiary_detect(beneficiary_list, crvs_feed)` |
| **Payments** | Fraud Screen | Real-time transaction anomaly detection | `fraud_screen(transaction, account_context)` |

**Design principle**: Sector-specific blocks **embed domain knowledge** — the eligibility logic for a maternity benefit is encoded into the block, not scattered across the workflow. Change the block, not the workflow.

### Foundational vs Sector-Specific: Comparison

| Dimension | Foundational | Sector-Specific |
|-----------|-------------|----------------|
| **Reusability** | High — used across all sectors | Narrow — built for one program/domain |
| **Domain knowledge** | None — pure AI primitive | High — policy + operational logic embedded |
| **Governance** | Lighter — general-purpose tool | Heavier — must reflect current policy version |
| **Example** | `translate()`, `ocr_extract()` | `eligibility_verify()`, `crop_risk_score()` |
| **Ownership** | Shared national AI infrastructure | Sector ministry / program owner |
| **Update trigger** | Model improvement | Policy change or new evidence base |

### Composing Both Types

Most DPI Workflows use both types together:

```
Citizen speaks in local dialect (voice)
        ↓
[Foundational: speech_to_text(audio, "swahili")]
        ↓
[Foundational: translate(text, "swahili", "english")]
        ↓
[Foundational: classify(text, welfare_intents)]
        → intent: "maternity_benefit_application"
        ↓
[Sector-Specific: eligibility_verify(citizen_id, "maternity_benefit")]
        ↓
[Sector-Specific: credential.issue(citizen_id, "maternity_benefit_vc")]
        ↓
[Foundational: translate(response, "english", "swahili")]
        ↓
[Foundational: text_to_speech(response, "swahili")]
        ↓
Citizen hears confirmation in their language
```

---

### AI Blocks Can Qualify as Digital Public Goods

An AI Block qualifies as a DPG when it:
- Is open-source or governed by open standards
- Declares its training data provenance
- Embeds privacy and consent controls
- Is designed for replaceability (no lock-in)

## DPGs as AI Blocks (MCP / API Readiness)

> **Reference**: https://digitalpublicinfrastructure.ai/dpgs.php

The DPI-AI Framework treats **Digital Public Goods as callable AI Blocks** — exposing DPG capabilities as tools that AI agents and DPI Workflows can invoke via API or MCP.

14+ DPGs are being made available as callable AI Blocks:

| DPG | AI Block Capability | Protocol |
|-----|-------------------|---------|
| **OpenFn** | Workflow orchestration, DPG-to-DPG data exchange | MCP (native, first DPG, June 2025) + API |
| **MOSIP** | Identity verification, biometric matching, eKYC | API |
| **Mojaloop** | Payment initiation, G2P disbursement, financial address lookup | API |
| **DHIS2** | Health data query, program enrollment, aggregate reporting | API |
| **Sunbird RC** | Registry read/write, credential issuance, eligibility check | API |
| **OpenG2P** | Beneficiary enrollment, social protection workflow | API |
| **OpenMRS** | Patient record query, clinical data exchange | API (FHIR) |
| **X-Road** | Federated government data exchange, G2G queries | API |
| **Diia** | Digital document verification, citizen portal | API |
| **QuarkID** | Verifiable credential issuance and verification (ZKP) | API + MCP |

### OpenFn: First DPG with Native MCP Server (June 2025)

OpenFn launched the first native MCP server interface for a DPG in June 2025:
- AI agents can **trigger OpenFn workflows** via MCP tool calls
- AI agents can **inspect run logs** and query adaptor schemas directly via MCP
- OpenFn also launched an **AI Assistant for plain-language workflow generation** — describe a workflow in natural language, OpenFn generates it
- Enables: AI agent → OpenFn (MCP) → DHIS2 / OpenMRS / identity system / payment rail

### Using DPGs as AI Block Tools

```python
# DPG AI Blocks exposed as LLM tools (MCP or function calling)
dpg_ai_blocks = [
    {
        "name": "mosip_verify_identity",
        "description": "Verify citizen identity via MOSIP. Returns verified=true/false and a token.",
        "parameters": {
            "citizen_id": "string",
            "auth_mode": "enum: OTP | biometric | face | QR"
        }
    },
    {
        "name": "sunbird_rc_query_registry",
        "description": "Query a Sunbird RC registry. Returns matching records.",
        "parameters": {
            "registry_name": "string",
            "query": "object - filter criteria"
        }
    },
    {
        "name": "mojaloop_initiate_payment",
        "description": "Initiate a G2P payment via Mojaloop. IRREVERSIBLE. Confirm before calling.",
        "parameters": {
            "recipient_financial_address": "string",
            "amount": "number",
            "currency": "string",
            "purpose_code": "string"
        }
    },
    {
        "name": "openfn_trigger_workflow",
        "description": "Trigger an OpenFn workflow by name. Returns run ID for tracking.",
        "parameters": {
            "workflow_name": "string",
            "input_data": "object"
        }
    }
]
```

## The AI Layer Model

AI capabilities can be organized as building blocks that sit on top of the DPI stack, composing DPI rail APIs with AI primitives to create intelligent public services:

```
┌──────────────────────────────────────────────────────────┐
│               AI-Powered DPI Services                    │
│  (Intelligent welfare routing, health triage, etc.)      │
├──────────────────────────────────────────────────────────┤
│               AI Orchestration Layer                     │
│  (Agents, workflows, tool use, multi-step reasoning)     │
├──────────┬───────────┬───────────┬───────────────────────┤
│ Perceive │  Reason   │ Generate  │  Remember             │
│ (Input)  │ (Process) │ (Output)  │  (State/Memory)       │
├──────────┴───────────┴───────────┴───────────────────────┤
│               DPI Rails                                  │
│  Identity API │ Payment API │ Data Exchange API          │
│  Registries │ Credential APIs │ Notification APIs        │
└──────────────────────────────────────────────────────────┘
```

## Core AI Building Blocks

### Block 1: Perception (Understanding Inputs)

AI blocks that convert raw inputs (text, image, audio, documents) into structured data that DPI systems can process.

**Natural Language Understanding (NLU)**
- Intent detection: "I want to apply for housing benefit" → `{ intent: "benefit_application", type: "housing" }`
- Entity extraction: Named entities, dates, amounts, IDs from free text
- Multilingual support: Translate and understand 100+ languages
- Code-switching: Handle mixed-language inputs (common in multilingual countries)

**Document Understanding (OCR + Structure)**
- Extract structured data from scanned documents (ID cards, certificates, forms)
- Handwriting recognition for legacy paper forms
- Table extraction from financial statements, medical reports
- Document classification (ID card vs. passport vs. utility bill)

**Speech Recognition (ASR)**
- Voice-to-text in local languages and dialects
- IVR (Interactive Voice Response) understanding
- Critical for low-literacy populations using DPI via voice

**Computer Vision**
- Liveness detection for biometric verification
- Document authenticity checks (holograms, microprint)
- Field boundary detection for agricultural land records
- Facility inspection from satellite/drone imagery

**Example — Document to DPI Record:**
```python
# Perception block: Extract from ID scan → DPI enrollment
async def perception_block(document_image: bytes) -> dict:
    # OCR to extract text
    extracted_text = await ocr_service.extract(document_image)
    # NLU to structure data
    structured = await nlp_service.structure(extracted_text, schema="id_card")
    # Return structured data ready for DPI registration API
    return {
        "name": structured["full_name"],
        "dob": structured["date_of_birth"],
        "id_number": structured["document_number"],
        "confidence": structured["confidence_score"]
    }
```

### Block 2: Reasoning (Making Decisions)

AI blocks that analyze structured data and make decisions or recommendations.

**Rule-Based + ML Hybrid Classifiers**
- Eligibility assessment: Is this person eligible for benefit X?
- Risk scoring: What is the fraud risk of this transaction?
- Routing decisions: Which government service does this request need?
- Anomaly detection: Is this authentication attempt unusual?

**Large Language Model (LLM) Reasoning**
- Multi-step reasoning over policy documents
- Legal text interpretation for eligibility decisions
- Complex query answering over government databases (RAG-based)
- Structured output generation (JSON decision records)

**Graph Reasoning**
- Relationship analysis (family networks for benefit eligibility)
- Fraud network detection (connected accounts, suspicious patterns)
- Supply chain verification (is this product from a registered supplier?)

**Predictive Models**
- Crop failure prediction (satellite + weather → insurance trigger)
- Disease outbreak forecasting (health data + mobility patterns)
- Cash flow prediction for thin-file loan underwriting

**Example — Eligibility Reasoning:**
```python
# Reasoning block: Assess benefit eligibility
async def eligibility_reasoning(applicant_id: str, benefit_type: str) -> dict:
    # Fetch verified data from DPI data exchange rail
    income_data = await data_exchange.fetch(applicant_id, "income", consent_token)
    household_data = await registry.get_household(applicant_id)
    
    # LLM reasoning against policy document (RAG)
    decision = await llm.reason(
        context=f"Income: {income_data}, Household: {household_data}",
        policy=await rag.fetch(f"eligibility_policy_{benefit_type}"),
        prompt="Determine eligibility and explain the decision in plain language"
    )
    
    return { "eligible": decision.eligible, "reason": decision.explanation, "confidence": decision.confidence }
```

### Block 3: Generation (Producing Outputs)

AI blocks that generate human-readable or machine-readable outputs.

**Text Generation**
- Personalized notification drafting ("Your application was approved because...")
- Policy explanation in plain language (multiple literacy levels)
- Multilingual communication generation
- Form pre-filling from structured data

**Structured Data Generation**
- JSON/XML document generation for DPI API calls
- Credential content generation (VC subject data)
- Report generation from aggregated DPI data
- API response summarization

**Voice / Speech Synthesis (TTS)**
- Speak notifications and instructions in local languages
- IVR menu and response generation
- Accessibility: read out government communications to blind citizens

**Visual Generation**
- Certificate and document rendering (from structured data → PDF/image)
- QR code and barcode generation for verifiable credentials
- Dashboard and report visualization

**Example — Personalized Notification:**
```python
# Generation block: Notify citizen of benefit status
async def generate_notification(decision: dict, citizen: dict) -> str:
    language = citizen.preferred_language
    literacy_level = citizen.literacy_level  # "basic" | "standard" | "advanced"
    
    prompt = f"""
    Generate a {literacy_level}-literacy notification in {language}.
    Decision: {decision}
    Citizen name: {citizen.name}
    Keep under 160 characters for SMS channel.
    """
    
    return await llm.generate(prompt, max_tokens=50)
```

### Block 4: Memory (State and Context)

AI blocks that maintain context across interactions and sessions.

**Session Memory**
- Multi-turn conversation state (what has the citizen already told us in this session?)
- Partial form completion state (resume abandoned applications)
- Authentication context (what was verified in this session?)

**Semantic Memory (Vector Store)**
- Knowledge base of government policies, schemes, eligibility criteria
- Retrieval-Augmented Generation (RAG): find relevant policy context for a query
- FAQ and known-issue tracking
- Historical interaction patterns

**Episodic Memory (Transaction History)**
- Past interactions with DPI services
- Complaint and grievance history
- Preference and channel preferences

**Example — RAG Policy Memory:**
```python
# Memory block: Retrieve relevant policy for a citizen query
async def policy_memory(query: str, jurisdiction: str) -> list[str]:
    # Embed the query
    query_embedding = await embedder.embed(query)
    
    # Search vector store for relevant policy chunks
    relevant_policies = await vector_store.search(
        embedding=query_embedding,
        filter={"jurisdiction": jurisdiction, "active": True},
        top_k=5
    )
    
    return [p.content for p in relevant_policies]
```

### Block 5: Safety and Alignment

AI blocks that ensure AI outputs are safe, accurate, and appropriate for DPI use.

**Hallucination Prevention**
- Ground all responses in verified DPI data (not model knowledge)
- Citation: every claim attributed to a source
- Confidence thresholds: low-confidence outputs escalate to human review
- Structured output enforcement: JSON schema validation for DPI API calls

**Bias and Fairness Monitoring**
- Demographic parity checks across population groups
- Disparate impact analysis for eligibility decisions
- Periodic bias audits with disaggregated metrics
- Human review triggers for high-stakes decisions affecting protected groups

**Output Filtering**
- PII redaction before logging
- Profanity and harmful content filtering in generated communications
- Legal compliance check (does output comply with local data protection law?)

**Audit Trail**
- Every AI decision logged with: input, model version, reasoning, output, confidence
- Immutable audit log linked to DPI transaction ID
- Human review trail for escalated cases

```python
# Safety block: Validate and audit AI decision
async def safety_check(ai_decision: dict, context: dict) -> dict:
    # Check confidence threshold
    if ai_decision["confidence"] < 0.85:
        return { "action": "escalate_to_human", "reason": "low_confidence" }
    
    # Check for bias signals
    if await bias_detector.check(ai_decision, context["demographics"]):
        return { "action": "flag_for_review", "reason": "potential_bias" }
    
    # Log to audit trail
    await audit_log.write({
        "dpi_transaction_id": context["txn_id"],
        "model_version": ai_decision["model_version"],
        "decision": ai_decision["result"],
        "confidence": ai_decision["confidence"],
        "timestamp": utcnow()
    })
    
    return { "action": "approve", "decision": ai_decision["result"] }
```

## Composing Blocks into DPI Services

Building blocks are composed into complete AI-powered DPI services:

```
Citizen Voice Input
        ↓
[ASR Block] → Text
        ↓
[NLU Block] → Intent + Entities
        ↓
[Memory Block] → Context + Relevant Policy
        ↓
[Reasoning Block] → Decision / Response
        ↓
[Safety Block] → Audit + Validate
        ↓
[DPI Rail Call] → Execute action (payment, enrollment, query)
        ↓
[Generation Block] → Response text
        ↓
[TTS Block] → Audio response
        ↓
Citizen hears response
```

## Key Technology Options (Technology-Neutral)

| Block | Open-Source Options | Cloud Options |
|-------|-------------------|---------------|
| LLM | LLaMA 3, Mistral, Qwen, Gemma | GPT-4o, Gemini, Claude |
| ASR | Whisper (OpenAI), Vosk, wav2vec | Azure Speech, Google STT |
| TTS | Coqui TTS, pyttsx3, eSpeak | AWS Polly, Azure TTS |
| OCR | Tesseract, EasyOCR, PaddleOCR | Azure Form Recognizer, AWS Textract |
| Vector Store | Weaviate, Qdrant, ChromaDB, pgvector | Pinecone, Azure AI Search |
| Embeddings | BGE, E5, nomic-embed | OpenAI ada, Cohere |
| Orchestration | LangChain, LlamaIndex, Haystack | AWS Bedrock Agents, Azure AI |

For DPI deployments, prefer **open-source models that can run on-premise** to avoid data sovereignty concerns with cloud AI APIs handling citizen PII.

> **Orchestrating these blocks at scale**: See the **dpi-ai-workflows** skill for how to wire building blocks into production workflows using LangGraph, Temporal, OpenFn, or n8n — including error handling, human-in-the-loop, and DPI rail integration patterns.
