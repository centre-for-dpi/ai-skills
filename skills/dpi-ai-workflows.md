---
name: dpi-ai-workflows
description: DPI Workflows — YAML-specified locus of control orchestrating AI Blocks and DPI rail APIs. Use when designing DPI service automation, choosing workflow tools, or integrating AI with DPI.
---

# DPI AI Workflows

> **Reference**: https://digitalpublicinfrastructure.ai/dpi-workflows.php  
> **DPI-AI Framework**: https://digitalpublicinfrastructure.ai/

## DPI-AI Framework: What is a DPI Workflow?

> *"A DPI Workflow is the recipe that coordinates AI Blocks with DPI systems, policy rules, data flows, and human oversight. The workflow is the locus of control — AI operates within it, never outside it."* — DPI-AI Framework

DPI Workflows are the **second element** of the DPI-AI Framework. They sit between Public Agents (citizen-facing) and AI Blocks (AI capabilities):

```
Public Agent  (runs on SLM or LLM via declared adapter)
(interprets intent, activates workflow)
        ↓
DPI Workflow  ←── THE LOCUS OF CONTROL
(orchestrates AI Blocks + DPI rail calls + human oversight)
        ↓
Foundational AI Blocks          Sector-Specific AI Blocks
(translate / STT / OCR /        (eligibility_verify /
 summarise / classify)           credential.issue /
                                 registry.search /
                                 symptom_triage)
        +
DPI Rail APIs (identity / payments / registries / credentials)
```

**Why workflows are the locus of control**: The workflow defines what AI can do, in what order, with what data, subject to what consent, with what human checkpoints. AI operates *within* the workflow boundary — it cannot reach outside it.

### Key Properties of DPI Workflows

| Property | Meaning |
|---------|---------|
| **Governed** | Each workflow is pre-approved; Public Agents can only activate approved workflows |
| **Reproducible** | Same input + same workflow = same output; auditable |
| **Reusable** | Workflow templates shared across governments and agencies |
| **Embedded safeguards** | No AI output directly modifies a DPI record without validation |
| **Human oversight** | Every workflow has a defined escalation path |
| **Consent-bound** | Data access steps carry explicit consent scope |

### Workflow Specification (YAML)

DPI Workflows can be specified in YAML (then executed by a workflow engine) or in code. The YAML is both the **governance document** and the **deployment spec** — the deployment must be an exact execution of this document.

```yaml
# DPI Workflow: Benefit Eligibility Assessment
workflow:
  id: benefit_eligibility_v1
  name: "Social Benefit Eligibility Check"
  version: "1.0"
  approved_by: "Ministry of Social Protection"
  requires_consent: ["income_data", "household_data"]

  # Model adapter declaration — swap model without changing workflow steps
  model:
    adapter: ollama             # "ollama" | "openai" | "vllm"
    model_id: llama3-8b-welfare-ft
    hosting: on_premise         # sovereign cloud | on_prem | external_api
    data_residency: in_country
    replaceability: "any model passing evaluation suite v3"
  
  steps:
    # --- Foundational AI Block: voice/text input normalisation ---
    - id: parse_input
      type: ai_block
      block_type: foundational
      action: speech_to_text    # if voice channel; no-op for text
      params:
        language: auto_detect

    - id: translate_input
      type: ai_block
      block_type: foundational
      action: translate
      params:
        target_lang: en         # normalise to processing language

    # --- DPI Rail: identity ---
    - id: authenticate
      type: dpi_rail
      action: identity.verify
      params:
        auth_mode: OTP
      on_failure: return_auth_failed

    # --- DPI Rail: data exchange ---
    - id: fetch_income
      type: dpi_rail
      action: data_exchange.fetch
      params:
        data_type: income_statement
        consent_scope: benefit_eligibility
      requires_consent: true

    # --- Sector-Specific AI Block: domain reasoning ---
    - id: assess_eligibility
      type: ai_block
      block_type: sector_specific
      action: eligibility_verify
      params:
        program_id: social_benefit_v3
        output_schema: EligibilityDecision
      confidence_threshold: 0.85
      on_low_confidence: escalate_to_human

    # --- Human checkpoint ---
    - id: human_review
      type: human_checkpoint
      trigger: on_escalation
      assignee: case_worker
      timeout_hours: 48

    # --- DPI Rail: registry write (irreversible — requires confirmation) ---
    - id: enroll
      type: dpi_rail
      action: registry.enroll
      requires_confirmation: true
      condition: "assess_eligibility.eligible == true"

    # --- Foundational AI Block: response generation in citizen's language ---
    - id: generate_response
      type: ai_block
      block_type: foundational
      action: translate
      params:
        target_lang: "{{parse_input.detected_lang}}"

    - id: speak_response
      type: ai_block
      block_type: foundational
      action: text_to_speech
      params:
        language: "{{parse_input.detected_lang}}"
      condition: "channel == voice"
  
  audit:
    log_all_steps: true
    retain_days: 2555  # 7 years
```

### Invoking AI Blocks in Workflows: Foundational vs Sector-Specific

DPI Workflows call both types of AI Block, each with a distinct role:

| Step role | Block type | Examples | When in the workflow |
|-----------|-----------|---------|---------------------|
| Input normalisation | **Foundational** | `speech_to_text`, `translate`, `ocr_extract` | First — before any domain logic |
| Intent classification | **Foundational** | `classify` | After input normalisation |
| Domain reasoning | **Sector-Specific** | `eligibility_verify`, `symptom_triage`, `crop_risk_score` | Core decision step |
| Record actions | **DPI Rail** | `registry.enroll`, `credential.issue`, `payment.initiate` | After AI decision |
| Output delivery | **Foundational** | `translate`, `text_to_speech`, `summarise` | Last — before returning to citizen |

**Design rule**: Foundational blocks handle the *edges* of the workflow (input → normalise, output → localise). Sector-specific blocks handle the *core* domain decision. DPI Rail calls handle the *authoritative action*.

A workflow that skips foundational blocks at the edges will fail multilingual and low-literacy users. A workflow that uses only foundational blocks and no sector-specific blocks will not encode the domain policy correctly.

### Model Adapter Declaration and Workflow Portability

The model adapter is declared **per workflow** (or per agent), not per step. This means:
- Every step in the workflow uses the same declared model
- Swapping from a cloud LLM to a sovereign SLM requires changing one line in the YAML — no workflow logic changes
- The workflow spec is the governance document; the adapter is the deployment detail

```yaml
# Pilot phase — cloud LLM
model:
  adapter: openai
  model_id: gpt-4o
  hosting: external_api
  data_residency: varies     # ← flag for review before production

# Production phase — sovereign SLM (same workflow, adapter swapped)
model:
  adapter: ollama
  model_id: llama3-8b-welfare-ft
  hosting: on_premise
  data_residency: in_country
```

**Progressive migration** — the workflow does not change between phases:

```
Phase 1: Pilot    →  adapter: openai   (validate logic, collect data)
Phase 2: Hybrid   →  adapter: vllm     (self-hosted open-weight, partial sovereignty)
Phase 3: Sovereign →  adapter: ollama  (fine-tuned SLM, full data residency)
```

> See the **dpi-public-agents** skill for the full LLM vs SLM decision framework and the 9-dimension comparison table.

### Reusable Workflow Templates

The DPI-AI Framework promotes sharing workflow templates across governments:

| Template | Use Case | Shared By |
|---------|---------|---------|
| G2P Eligibility Check | Social benefit eligibility | CDPI template library |
| Birth Registration → Enrollment | CRVS to health/social protection | OpenCRVS + OpenG2P |
| KYC for Financial Inclusion | Bank account onboarding | Mojaloop + MOSIP |
| Immunisation Reminder | Child health follow-up | DHIS2 + OpenFn |
| Farmer Credential Issuance | Agricultural benefit access | CDPI Bootcamp template |

### Multi-Agent Orchestration in DPI Workflows

For complex services spanning multiple DPI domains, workflows can orchestrate multiple agents:

> *"Multiple agents collaborating by sharing context, coordinating across workflows, and dynamically decomposing complex tasks — DPI provides the shared digital rails: common protocols, registries, and governance frameworks."*

```
Citizen Request: "I need maternal health support"
        ↓
Orchestrator Workflow
  ├── Identity Sub-Workflow  → verifies identity via MOSIP
  ├── Health Sub-Workflow    → fetches FHIR health record
  ├── Eligibility Sub-Workflow → checks maternity benefit criteria
  └── G2P Sub-Workflow       → triggers payment if eligible
        ↓
Each sub-workflow is independently governed, logged, and auditable
```

### Federated Deployment

DPI Workflows must be designed for federated deployment:
- Each ministry/agency deploys and governs its own workflows
- Shared workflow specs (YAML) ensure interoperability without centralization
- A citizen-facing workflow spans agency boundaries while each agency retains data ownership

## What is a DPI Workflow? (Technical Definition)

A DPI workflow is an orchestrated sequence of steps that connects DPI building blocks (identity, payments, data exchange, registries, credentials) to deliver an end-to-end citizen or government service. Each step in a workflow typically combines AI building blocks (perception, reasoning, generation, memory, safety) — see the **dpi-ai-blocks** skill for the composition of those primitives. Workflows can be:

- **Fully automated**: Triggered by events, execute without human intervention
- **Human-in-the-loop**: AI handles routine cases; humans handle exceptions
- **AI-assisted**: LLM reasoning guides routing, assessment, or response generation
- **Data exchange pipelines**: Move data between DPI systems on schedule or trigger

**Key principle**: AI provides the intelligence; DPI provides the authority. AI agents should call DPI APIs for all authoritative actions (identity verification, payments, registry updates) — never bypass DPI rails.

## Workflow Platform Categories

DPI workflows span a spectrum from code-first to no-code:

```
Code-First                                         No-Code/Low-Code
─────────────────────────────────────────────────────────────────
  Python/    LangGraph   Temporal   Apache      OpenFn      n8n
  Custom                            Airflow
  
  Maximum    Stateful    Long-      Batch/      DPI-        Visual +
  control    AI agents   running    scheduled   specific    AI agents
                         durable    pipelines   DPG         
                         workflows              connectors  
```

### When to Use Each Approach

| Scenario | Recommended Platform |
|---------|---------------------|
| AI agent orchestrating DPI tools | LangGraph (code) or n8n (visual) |
| Long-running enrollment (days/weeks) | Temporal |
| Batch G2P disbursement | Apache Airflow |
| DHIS2 ↔ OpenMRS ↔ Registry sync | OpenFn (DPG-specific adaptors) |
| Citizen chatbot with DPI tool access | n8n AI Agent or LangGraph |
| G2G data exchange pipeline | OpenFn or X-Road |
| Quick prototype/integration | n8n (visual, 400+ nodes) |
| Full sovereignty, custom logic | Python/custom code |

## What is a DPI AI Workflow?

A DPI AI Workflow specifically adds AI reasoning to the workflow — an orchestrated sequence of AI reasoning steps and DPI rail API calls that delivers an intelligent citizen service. The AI layer provides intelligence (understanding, reasoning, generation); the DPI layer provides the authoritative actions (verify identity, move money, share consented data).

## Workflow Patterns

### Pattern 1: Linear (Sequential) Workflow

Steps execute in order; each step's output feeds the next:

```python
async def benefit_application_workflow(citizen_input: str) -> WorkflowResult:
    # Step 1: Understand intent
    intent = await ai.understand(citizen_input)
    assert intent.type == "benefit_application"
    
    # Step 2: Authenticate identity via DPI rail
    auth_result = await dpi.identity.authenticate(
        citizen_id=intent.citizen_id,
        factor="otp",
        otp=intent.otp
    )
    if not auth_result.verified:
        return WorkflowResult(status="auth_failed")
    
    # Step 3: Fetch eligibility data via data exchange
    income_data = await dpi.data_exchange.fetch(
        subject=auth_result.verified_id,
        data_type="income_statement",
        consent_token=await dpi.consent.get(citizen_id, "benefit_eligibility")
    )
    
    # Step 4: AI reasons about eligibility
    eligibility = await ai.reason_eligibility(income_data, benefit_type=intent.benefit)
    
    # Step 5: If eligible, register in beneficiary registry
    if eligibility.eligible:
        enrollment = await dpi.registry.enroll(
            citizen_id=auth_result.verified_id,
            program=intent.benefit
        )
    
    # Step 6: Generate personalized response
    response = await ai.generate_response(eligibility, enrollment, language=intent.language)
    
    return WorkflowResult(status="complete", message=response)
```

### Pattern 2: Conditional / Branching Workflow

Different paths based on conditions:

```python
async def kyc_workflow(application: LoanApplication) -> KYCResult:
    # Verify identity
    identity = await dpi.identity.authenticate(application.citizen_id)
    
    # Branch on available data
    if await dpi.registry.has_credit_history(identity.id):
        # Fast path: existing credit history
        credit_data = await dpi.data_exchange.fetch(identity.id, "credit_report")
        decision = await ai.credit_decision(credit_data)
    else:
        # Alternative scoring path: use transaction + income data
        income = await dpi.data_exchange.fetch(identity.id, "income_statement")
        transactions = await dpi.data_exchange.fetch(identity.id, "bank_transactions")
        decision = await ai.alternative_credit_decision(income, transactions)
    
    # High-risk path: require additional verification
    if decision.risk_level == "HIGH":
        biometric = await dpi.identity.verify_biometric(identity.id)
        decision = await ai.final_decision(decision, biometric_confirmed=biometric.verified)
    
    return KYCResult(approved=decision.approved, limit=decision.credit_limit)
```

### Pattern 3: Parallel Workflow

Independent steps run concurrently for speed:

```python
async def onboarding_workflow(citizen_id: str) -> OnboardingResult:
    # Run all data fetches in parallel
    identity, income, existing_accounts, credit = await asyncio.gather(
        dpi.identity.get_profile(citizen_id),
        dpi.data_exchange.fetch(citizen_id, "income"),
        dpi.data_exchange.fetch(citizen_id, "accounts"),
        dpi.data_exchange.fetch(citizen_id, "credit_score")
    )
    
    # AI synthesizes all data simultaneously fetched
    risk_profile = await ai.build_risk_profile(identity, income, existing_accounts, credit)
    
    return OnboardingResult(profile=risk_profile)
```

### Pattern 4: Human-in-the-Loop Workflow

AI handles routine cases; humans handle exceptions:

```python
async def benefit_review_workflow(application_id: str) -> ReviewResult:
    application = await registry.get_application(application_id)
    
    # AI pre-assessment
    ai_assessment = await ai.assess_application(application)
    
    # Auto-approve high-confidence, clearly eligible cases
    if ai_assessment.confidence > 0.95 and ai_assessment.eligible:
        return await auto_approve(application)
    
    # Auto-reject high-confidence, clearly ineligible cases (with appeal path)
    if ai_assessment.confidence > 0.95 and not ai_assessment.eligible:
        return await auto_reject(application, reason=ai_assessment.reason)
    
    # Escalate ambiguous cases to human reviewer with AI pre-filled assessment
    return await escalate_to_human(
        application,
        ai_summary=ai_assessment.summary,
        ai_recommendation=ai_assessment.recommendation,
        flagged_concerns=ai_assessment.concerns
    )
```

### Pattern 5: Event-Driven Workflow

Triggered by events from DPI systems:

```python
# Triggered by DPI payment system webhook
@event_handler("payment.completed")
async def on_payment_completed(event: PaymentEvent):
    if event.purpose_code == "G2P_WELFARE":
        # AI generates personalized usage guidance
        citizen_context = await dpi.registry.get_citizen(event.recipient_id)
        guidance = await ai.generate_welfare_guidance(
            amount=event.amount,
            program=event.program,
            household_context=citizen_context
        )
        await dpi.notify(event.recipient_id, guidance, channel="sms")

# Triggered by consent expiry notification
@event_handler("consent.expiring")
async def on_consent_expiring(event: ConsentEvent):
    # AI generates personalized renewal reminder
    renewal_message = await ai.generate_consent_renewal(event.consent_details)
    await dpi.notify(event.citizen_id, renewal_message)
```

## Workflow Orchestration Frameworks

### LangGraph (Python) — Recommended for Complex Workflows

```python
from langgraph.graph import StateGraph, END

class BenefitWorkflowState(TypedDict):
    citizen_id: str
    auth_token: Optional[str]
    income_data: Optional[dict]
    eligibility: Optional[dict]
    enrollment: Optional[dict]
    response: Optional[str]

workflow = StateGraph(BenefitWorkflowState)

workflow.add_node("authenticate", authenticate_citizen)
workflow.add_node("fetch_income", fetch_income_data)
workflow.add_node("assess_eligibility", assess_eligibility)
workflow.add_node("enroll", enroll_in_program)
workflow.add_node("generate_response", generate_response)

workflow.add_edge("authenticate", "fetch_income")
workflow.add_conditional_edges("assess_eligibility", 
    lambda state: "enroll" if state["eligibility"]["eligible"] else "generate_response")
workflow.add_edge("enroll", "generate_response")
workflow.add_edge("generate_response", END)

workflow.set_entry_point("authenticate")
app = workflow.compile()
```

### Temporal (Go/Python/Java) — Recommended for Long-Running Workflows

```python
# Temporal workflow for multi-day enrollment process
@workflow.defn
class EnrollmentWorkflow:
    @workflow.run
    async def run(self, citizen_id: str) -> str:
        # Step 1: Document collection (may take days)
        docs = await workflow.execute_activity(
            collect_documents,
            citizen_id,
            start_to_close_timeout=timedelta(days=7)
        )
        
        # Step 2: AI verification of documents
        verified = await workflow.execute_activity(
            verify_documents_ai,
            docs,
            start_to_close_timeout=timedelta(minutes=5)
        )
        
        # Step 3: DPI enrollment
        if verified:
            enrollment_id = await workflow.execute_activity(
                dpi_enroll,
                citizen_id,
                start_to_close_timeout=timedelta(minutes=1)
            )
            return enrollment_id
```

### Apache Airflow — Recommended for Batch DPI Workflows

```python
# Batch G2P disbursement workflow
with DAG("g2p_disbursement", schedule="0 9 * * *") as dag:
    
    fetch_beneficiaries = PythonOperator(
        task_id="fetch_active_beneficiaries",
        python_callable=lambda: dpi.registry.list_active_beneficiaries()
    )
    
    ai_fraud_screen = PythonOperator(
        task_id="ai_fraud_screening",
        python_callable=lambda: ai.batch_fraud_screen(beneficiaries)
    )
    
    initiate_payments = PythonOperator(
        task_id="initiate_g2p_payments",
        python_callable=lambda: dpi.payments.batch_disburse(screened_beneficiaries)
    )
    
    fetch_beneficiaries >> ai_fraud_screen >> initiate_payments
```

## DPI API Integration Patterns

### Tool Definitions for LLM Agents

```python
# Define DPI APIs as LLM tools
dpi_tools = [
    {
        "name": "verify_identity",
        "description": "Verify a citizen's identity using their ID number and OTP. Returns verified or not.",
        "parameters": {
            "citizen_id": "string - the citizen's national ID number",
            "otp": "string - the one-time password sent to their phone"
        }
    },
    {
        "name": "fetch_income_data",
        "description": "Fetch verified income data for a citizen. Requires valid consent token.",
        "parameters": {
            "citizen_id": "string",
            "consent_token": "string - obtained from consent management API"
        }
    },
    {
        "name": "initiate_payment",
        "description": "Initiate a payment to a citizen's registered account. For G2P use only.",
        "parameters": {
            "recipient_id": "string",
            "amount": "number",
            "currency": "string",
            "purpose_code": "string - G2P purpose code"
        }
    },
    {
        "name": "issue_credential",
        "description": "Issue a verifiable credential to a citizen after successful verification.",
        "parameters": {
            "citizen_id": "string",
            "credential_type": "string",
            "attributes": "object - credential attributes"
        }
    }
]
```

### Safe Tool Execution Pattern

```python
async def safe_dpi_tool_call(tool_name: str, params: dict, context: dict) -> dict:
    # Pre-execution checks
    if not await auth.has_permission(context["service_id"], tool_name):
        raise PermissionError(f"Service not authorized to call {tool_name}")
    
    if tool_name in HIGH_STAKES_TOOLS:  # payments, enrollment, credential issuance
        await audit_log.pre_execution(tool_name, params, context)
        result = await dpi_tools[tool_name].execute(params)
        await audit_log.post_execution(tool_name, params, result, context)
    else:
        result = await dpi_tools[tool_name].execute(params)
    
    return result
```

## Error Handling in DPI Workflows

### Error Categories and Handling

```python
class DPIWorkflowError(Exception): pass
class AuthenticationError(DPIWorkflowError): pass
class ConsentError(DPIWorkflowError): pass
class DataUnavailableError(DPIWorkflowError): pass
class RateLimitError(DPIWorkflowError): pass

async def robust_dpi_call(api_call, retries=3, backoff=2.0):
    for attempt in range(retries):
        try:
            return await api_call()
        except RateLimitError:
            await asyncio.sleep(backoff ** attempt)  # Exponential backoff
        except DataUnavailableError as e:
            if e.is_temporary:
                await asyncio.sleep(5)
            else:
                raise  # Permanent unavailability; don't retry
        except AuthenticationError:
            raise  # Never retry auth failures; security risk
    raise DPIWorkflowError("Max retries exceeded")
```

### Graceful Degradation

```python
async def workflow_with_fallback(citizen_id: str):
    # Try primary data source
    try:
        income = await dpi.data_exchange.fetch(citizen_id, "formal_income")
    except DataUnavailableError:
        # Fallback to alternative data
        try:
            income = await dpi.data_exchange.fetch(citizen_id, "mobile_money_history")
            income["source"] = "alternative"  # Flag for different scoring model
        except DataUnavailableError:
            income = None  # Proceed without income data; use conservative estimate
    
    # Adjust AI reasoning based on available data
    return await ai.assess(citizen_id, income, data_quality=income["source"] if income else "none")
```

## Testing DPI AI Workflows

```python
# Unit test with mocked DPI APIs
async def test_benefit_workflow_eligible():
    with mock_dpi() as dpi_mock:
        dpi_mock.identity.authenticate.return_value = AuthResult(verified=True, id="citizen_123")
        dpi_mock.data_exchange.fetch.return_value = IncomeData(annual=12000, currency="USD")
        
        result = await benefit_application_workflow("Apply for housing benefit, ID 123, OTP 456789")
        
        assert result.status == "complete"
        assert "approved" in result.message.lower()
        dpi_mock.registry.enroll.assert_called_once_with("citizen_123", program="housing_benefit")
```

## Observability for DPI AI Workflows

```python
# Structured logging for every workflow step
@trace_workflow_step
async def step_assess_eligibility(state: WorkflowState) -> WorkflowState:
    span = tracer.start_span("eligibility_assessment")
    span.set_attribute("citizen_id", anonymize(state.citizen_id))
    span.set_attribute("benefit_type", state.benefit_type)
    
    result = await ai.reason_eligibility(state.income_data, state.benefit_type)
    
    span.set_attribute("eligible", result.eligible)
    span.set_attribute("confidence", result.confidence)
    span.set_attribute("model_version", result.model_version)
    span.end()
    
    return state | {"eligibility": result}
```

## No-Code / Low-Code Workflow Tools for DPI

Not all DPI workflows require writing code. Visual workflow platforms lower the barrier for government teams and allow non-developers to build and maintain DPI integrations.

### OpenFn — DPG-Specific Integration

> **See**: dpi-openfn skill for full details | https://openfn.org | https://docs.openfn.org

OpenFn is a **registered Digital Public Good** built specifically for DPI data exchange. Use it when you need pre-built adaptors for DPGs (DHIS2, OpenMRS, Kobo, Primero, CommCare):

```
Typical OpenFn DPI workflow:
[Webhook: New birth in CRVS]
       ↓
[Job 1: Validate record]
       ↓
[Job 2: Call identity API → generate National ID]
       ↓
[Job 3: Enroll in DHIS2 immunization tracker]
       ↓
[Job 4: Send SMS to mother]
```

**Best for**: G2G data exchange, DPG-to-DPG integration, scheduled data syncs, health information exchange, government programs using DHIS2/OpenMRS ecosystem.

**Deployment**: Cloud (app.openfn.org) or fully self-hosted (AGPLv3, PostgreSQL only dependency).

### n8n — Visual AI Agents + Broad Integration

> **See**: dpi-n8n skill for full details | https://n8n.io | https://docs.n8n.io

n8n is a **fair-code visual automation platform** with native AI Agent nodes (LangChain-based). Best for citizen-facing automation where LLMs need to reason and call DPI APIs:

```
n8n DPI AI workflow (visual):
[Webhook Trigger]
       ↓
[AI Agent Node]
  - LLM: Claude / Ollama (local)
  - Tools: DPI Identity, Registry, Payments (HTTP nodes)
  - Memory: PostgreSQL
       ↓
[HTTP Request → DPI Identity API]
       ↓
[HTTP Request → Registry Query]
       ↓
[Send Notification (SMS/WhatsApp)]
```

**Best for**: AI-powered citizen service bots, benefit eligibility automation, complaint routing, reconciliation with AI exception handling.

**Deployment**: n8n.cloud or fully self-hosted (Docker/Kubernetes, Sustainable Use License). 400+ nodes, 900+ community templates.

### OpenFn + n8n Together

These tools are complementary — use both in the same DPI program:

```
OpenFn: DHIS2 ↔ OpenMRS ↔ Registry (DPG data exchange, specialist tool)
                    ↕
n8n:    Citizen chat → AI reasoning → DPI API calls (citizen-facing, AI layer)
                    ↕
Both call the same DPI HTTP APIs (identity, payments, registries)
```

### Platform Selection Summary

| Requirement | Use |
|-------------|-----|
| DHIS2, OpenMRS, Kobo, CommCare integration | **OpenFn** |
| AI agent with LLM reasoning | **n8n** |
| DPG registered, open-source first | **OpenFn** (AGPLv3) |
| Visual editor, low code barrier | **n8n** |
| Kafka triggers for event streams | **OpenFn** |
| 400+ integrations needed | **n8n** |
| Long-running (days/weeks) durable workflows | **Temporal** (code) |
| Batch scheduled pipelines (ETL) | **Apache Airflow** (code) |
| Maximum control, custom logic | **Python + LangGraph** (code) |
| Quick prototype | **n8n** or **OpenFn** |

## Key Workflow Design Principles

1. **DPI APIs are the source of truth** — AI reasons over DPI data; it doesn't generate authoritative facts
2. **Reversibility** — Before calling irreversible DPI APIs (payments, enrollment), confirm with a safety check
3. **Auditability** — Every AI decision that touches a DPI action must be logged with reasoning
4. **Graceful degradation** — Design for partial DPI availability; define behavior when services are down
5. **Human escalation** — Every automated decision pathway must have a human escalation path
6. **Idempotency** — DPI API calls from workflows must be idempotent (retry-safe)
7. **Privacy minimization** — Request only the DPI data actually needed for the workflow step
8. **Sovereignty** — Prefer self-hosted workflow platforms when DPI workflows process citizen PII
9. **Choose the right abstraction** — Visual tools (OpenFn, n8n) for operational teams; code-first (LangGraph, Temporal) for engineering teams building complex agents
