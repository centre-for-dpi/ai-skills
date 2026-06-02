---
name: dpi-public-agents
description: Public sector AI agents — constrained citizen interfaces (chat/voice/USSD), model selection (LLM vs SLM), adapter pattern, governance. Use when designing AI-enabled government services.
---

# DPI Public Agents

> **Reference**: https://digitalpublicinfrastructure.ai/public-agents.php  
> **DPI-AI Framework**: https://digitalpublicinfrastructure.ai/

## What is a Public Agent? (DPI-AI Framework Definition)

> *"A Public Agent is an AI-enabled assistant — software-based, AI-assisted human, or a hybrid of both — that interacts with citizens or officials within defined rights-based boundaries, and activates DPI Workflows to complete tasks."* — DPI-AI Framework

**A Public Agent is NOT autonomous.** It is a constrained interface that activates governed workflows:

| What a Public Agent CAN do | What it CANNOT do |
|--------------------------|------------------|
| Understand intent in any language, any channel | Access data without citizen consent |
| Activate pre-approved DPI Workflows | Modify records directly |
| Interpret requests and route to the right workflow | Bypass human oversight |
| Offer human escalation at any point | Make irreversible decisions autonomously |
| Operate via WhatsApp, USSD, voice, or web | Act outside defined workflow boundaries |

**The key principle**: *Government AI does not need to be brilliant — it needs to be reliable, bounded, and governable.*

Public Agents are **accountable extensions of government capacity**, not autonomous AI systems. They interpret citizen intent and trigger the right combination of AI Blocks and DPI Workflows — but the workflows provide the authority, not the agent.

## The DPI-AI Framework: Three Elements

Public Agents are one element of the three-element DPI-AI Framework:

```
┌─────────────────────────────────────────────────────────────┐
│                    DPI-AI Framework                          │
│                                                             │
│  ┌─────────────┐   ┌─────────────────┐   ┌───────────────┐ │
│  │  AI Blocks  │   │  DPI Workflows  │   │ Public Agents │ │
│  │  (modular   │   │  (orchestrated  │   │ (citizen-     │ │
│  │   AI caps)  │   │   sequences)    │   │  facing UI)   │ │
│  └─────────────┘   └─────────────────┘   └───────────────┘ │
│         ↑                  ↑                     ↑           │
│         └──────────────────┴─────────────────────┘          │
│                     DPI Rails                               │
│         Identity · Payments · Data Exchange · Registries    │
└─────────────────────────────────────────────────────────────┘
```

- **AI Blocks** provide modular AI capabilities (perception, reasoning, generation, memory, safety)
- **DPI Workflows** orchestrate AI Blocks with DPI rail calls, policy rules, and human oversight
- **Public Agents** are the citizen-facing interface that interprets intent and activates workflows

> See the **dpi-ai-blocks** and **dpi-ai-workflows** skills for the other two elements.

## Model Sizing and Sovereignty

The DPI-AI Framework makes a strong recommendation on model choice:

> *"A 3–8 billion parameter model fine-tuned on that specific task, running on sovereign infrastructure, is more appropriate than a frontier LLM accessed via a third-party API — and it will be cheaper, faster, and more controllable at the 10 million interactions that matter."*

| Approach | Pros | Cons | Best for |
|----------|------|------|---------|
| **3–8B SLM, fine-tuned, on-premise** | Sovereign, fast, controllable, cheap at scale | Requires fine-tuning effort | Production, PII-sensitive, 10M+ interactions |
| **Frontier LLM (API)** | High capability, zero setup | Data sovereignty risk, cost at scale, API dependency | Prototyping, low-volume pilots |
| **Hybrid: SLM + frontier fallback** | Best of both | More complex | Most production deployments |

**When to use sovereign SLM**:
- PII-sensitive workflows (citizen data must not leave the country)
- Low-connectivity contexts (need edge/local deployment)
- Local dialect/language support
- National-scale production deployment

## Channel Architecture

A single Public Agent logic deployed across all citizen channels:

```
WhatsApp Business API
        ↓
SMS / USSD (feature phones)          →  Channel Adapter Layer
        ↓                                      ↓
IVR / Voice (IVR gateway)            →  Public Agent Core
        ↓                                 (SLM + Workflow Router)
Web Chat / Mobile App                          ↓
        ↓                                DPI Workflows
Government Portal                              ↓
                                         DPI Rails
```

**Channel considerations**:
- **WhatsApp**: Rich media, 2B+ users, works on low-end smartphones
- **USSD**: No internet required, works on any phone, session-based (180s timeout)
- **IVR/Voice**: Critical for low-literacy populations; ASR in local languages
- **Web/App**: Richest experience; full document upload and display

## Model Selection: LLMs vs SLMs

> *"Public Agents can run on any language model — from large frontier APIs to small on-device models. The choice is not just technical; it has direct implications for sovereignty, cost, latency, and inclusion. Neither type is universally better. Each has a role."* — DPI-AI Framework

**Model selection is a governance decision, not just a technical one.** Every Public Agent spec must declare the model it uses, where it runs, and under what conditions it can be replaced.

### LLMs vs SLMs: Comparison for Government

| Dimension | Large LLMs (frontier APIs) | Small LLMs / SLMs (on-premise / on-device) |
|-----------|--------------------------|-------------------------------------------|
| **Capability** | Highest reasoning, broad knowledge | Narrower but sufficient for bounded tasks |
| **Sovereignty** | Data leaves the country (API call) | Full data sovereignty, runs in-country |
| **Cost at scale** | High — per-token pricing at 10M interactions | 10–30× cheaper at scale |
| **Latency** | Several seconds (network + inference) | Sub-second (local inference) |
| **Connectivity** | Requires internet | Works offline / low-connectivity |
| **Inclusion** | Fails in low-connectivity contexts | Reaches district offices, rural areas |
| **Control** | Vendor controls model updates | Government controls model version |
| **Fine-tuning** | Hard to fine-tune (API-only) | Full fine-tuning on local languages/policy |
| **Best for** | Pilots, complex reasoning, prototyping | Production at national scale, PII workflows |

### The Role of Each Model Type

**Large frontier LLMs** are best for:
- Early pilots where infrastructure doesn't exist yet
- Complex, open-ended reasoning that requires broad world knowledge
- Low-volume use cases where per-token cost is acceptable
- Tasks where fine-tuning capacity is unavailable

**Small Language Models (SLMs, 1–8B parameters)** are best for:
- Production deployment at national scale (10M+ interactions)
- Workflows that handle citizen PII (data must not leave the country)
- Low-connectivity environments (district offices, rural health workers)
- Tasks with a well-defined scope — a fine-tuned 3B SLM outperforms a frontier LLM on a specific government task
- Edge deployment on commodity hardware

> *"A 3–8 billion parameter model fine-tuned on that specific task, running on sovereign infrastructure, is more appropriate than a frontier LLM accessed via a third-party API — and it will be cheaper, faster, and more controllable at the 10 million interactions that matter."*

### The Adapter Pattern: Swap Models Without Rebuilding Workflows

The DPI-AI Framework uses an **adapter-based design** so the workflow never changes when the model changes. Only the adapter does:

```yaml
# Public Agent model declaration (in workflow spec)
agent:
  id: welfare_eligibility_agent_v1
  model:
    adapter: ollama          # swap to "openai" or "vllm" without changing workflow
    model_id: llama3-8b-welfare-ft   # fine-tuned on welfare policy
    hosting: on_premise      # sovereign cloud | on_prem | external_api
    data_residency: in_country
    replaceability: "any model that passes evaluation suite v3"
```

```python
# Adapter interface — all models implement the same contract
class ModelAdapter:
    async def complete(self, prompt: str, tools: list, context: dict) -> ModelResponse:
        ...

class OllamaAdapter(ModelAdapter):   # local sovereign SLM
    async def complete(self, prompt, tools, context):
        return await ollama.chat(model=self.model_id, messages=prompt, tools=tools)

class OpenAIAdapter(ModelAdapter):   # frontier LLM for pilots
    async def complete(self, prompt, tools, context):
        return await openai.chat.completions.create(model=self.model_id, messages=prompt, tools=tools)

class VLLMAdapter(ModelAdapter):     # self-hosted open-weight model
    async def complete(self, prompt, tools, context):
        return await vllm_client.generate(prompt=prompt, tools=tools)
```

### Progressive Migration Path

Governments do not have to start with sovereign infrastructure. The framework supports a pragmatic progression:

```
Phase 1: Pilot
  Cloud LLM (OpenAI / Gemini / Claude API)
  → validate workflow logic, gather data, prove impact
  → DPI Workflows unchanged; only adapter differs
        ↓
Phase 2: Hybrid
  Sensitive/high-volume steps → sovereign SLM on-prem
  Complex/low-volume steps → cloud LLM fallback
  → fine-tune SLM on task data collected in Phase 1
        ↓
Phase 3: Sovereign at Scale
  Fine-tuned SLM (3–8B) on sovereign infra
  → sub-second latency, full data residency, 10–30× cheaper
  → DPI Workflows unchanged; adapter swapped
```

> *"Start with what you can deploy today — even a cloud LLM for a pilot — and migrate to a sovereign SLM as capacity grows. The workflow does not change. Only the adapter does."*

### Country Language Model Examples

Several governments have built sovereign language models tailored to local context:

| Country / Region | Model | Capability |
|-----------------|-------|-----------|
| Singapore | SEA-LION | Southeast Asian languages |
| Malaysia | ILMU AI | Outperforms global models on Malay benchmarks |
| India | Multiple initiatives | 22 scheduled languages (IndiaAI Mission) |
| UAE | Falcon | Arabic-first, open-weight |
| France | Mistral | European sovereign model |

---

## Governance Constraints (Non-Negotiable)

The DPI-AI Framework defines four mandatory governance properties for every Public Agent:

### 1. Constrained to Approved Workflows Only
A Public Agent can only activate pre-approved, pre-tested DPI Workflows. It cannot improvise actions outside the defined workflow set.

### 2. Always Offers Human Recourse
Every interaction must offer a clear path to a human officer. No dead ends. Citizens must always be able to escalate.

### 3. Consent Before Data Access
No agent may fetch citizen data from DPI rails without a valid consent artefact scoped to the specific purpose and interaction.

### 4. Federated Deployment
Public Agents must be designed for federated deployment across multiple government agencies, remaining independently deployable while interoperating through shared standards.

```python
# Governance constraint enforcement (illustrative)
class PublicAgentGovernance:
    
    async def activate_workflow(self, workflow_id: str, context: AgentContext):
        # 1. Check workflow is pre-approved
        if workflow_id not in self.approved_workflows:
            raise WorkflowNotApprovedError(f"{workflow_id} is not in approved set")
        
        # 2. Check consent for any data access
        if self.approved_workflows[workflow_id].requires_data:
            consent = await self.consent_manager.verify(
                citizen_id=context.citizen_id,
                purpose=workflow_id
            )
            if not consent.valid:
                return self.request_consent(context)
        
        # 3. Activate workflow with full audit
        return await self.workflow_engine.run(workflow_id, context)
    
    async def handle_any_input(self, input: str, context: AgentContext):
        # Always offer human escalation
        if self.detect_distress_or_complexity(input):
            return self.escalate_to_human(context)
        
        intent = await self.llm.parse_intent(input)
        return await self.activate_workflow(intent.workflow_id, context)
```

## Agent Architecture

```
┌──────────────────────────────────────────────────────────┐
│                  DPI Public Agent                        │
│                                                          │
│  ┌─────────────┐   ┌──────────────┐   ┌─────────────┐  │
│  │  Perception │   │   Reasoning  │   │  Generation │  │
│  │  (NLU/ASR)  │   │  (LLM + RAG) │   │  (Response) │  │
│  └─────────────┘   └──────────────┘   └─────────────┘  │
│                          ↕                               │
│             ┌────────────────────────┐                   │
│             │   Tool Orchestrator    │                   │
│             │  (decides which DPI    │                   │
│             │   APIs to call)        │                   │
│             └────────────────────────┘                   │
│                          ↕                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │              DPI Tool Layer                       │   │
│  │  identity_verify │ payment_initiate │ data_fetch  │   │
│  │  registry_query  │ credential_issue │ notify_send │   │
│  └──────────────────────────────────────────────────┘   │
│                          ↕                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Safety & Governance Layer               │   │
│  │  Consent check │ Rate limit │ Audit log           │   │
│  │  Human escalation │ Bias monitor │ PII redaction  │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
          ↕                              ↕
   DPI Rails (APIs)              Human Supervisor
```

## Public Agent Types

### Type 1: Citizen Service Agent

Handles citizen-facing service requests over text or voice:

```python
class CitizenServiceAgent:
    
    async def handle(self, citizen_input: str, channel: str) -> AgentResponse:
        # Parse intent
        intent = await self.nlu.parse(citizen_input)
        
        # Route to appropriate handler
        match intent.category:
            case "benefit_inquiry":
                return await self.handle_benefit_inquiry(intent)
            case "document_request":
                return await self.handle_document_request(intent)
            case "payment_inquiry":
                return await self.handle_payment_inquiry(intent)
            case "complaint":
                return await self.escalate_to_human(intent)
            case _:
                return await self.handle_general(intent)
    
    async def handle_benefit_inquiry(self, intent: Intent) -> AgentResponse:
        # Verify citizen identity (tool call)
        auth = await self.tools.verify_identity(intent.citizen_id, intent.otp)
        if not auth.verified:
            return AgentResponse("Identity verification failed. Please try again.")
        
        # Fetch benefit status (tool call)
        status = await self.tools.query_registry(auth.citizen_id, "benefit_status")
        
        # Generate personalized response
        return AgentResponse(
            await self.llm.generate(f"Status: {status}", language=intent.language)
        )
```

### Type 2: Government Staff Agent

Assists case workers and civil servants with complex decisions:

```python
class CaseWorkerAgent:
    
    async def assist_case_review(self, case_id: str, worker_query: str) -> AgentResponse:
        # Fetch case data from DPI registry
        case = await self.tools.registry_query("cases", case_id)
        
        # Fetch supporting data with consent
        documents = await self.tools.data_fetch(case.citizen_id, "supporting_documents")
        
        # AI summarizes case with relevant policy context (RAG)
        summary = await self.llm.summarize(
            case=case,
            documents=documents,
            policy_context=await self.rag.fetch(worker_query),
            prompt="Summarize this case and highlight key eligibility factors"
        )
        
        return AgentResponse(
            summary=summary,
            recommendation=await self.llm.recommend(summary),
            supporting_data=documents
        )
```

### Type 3: Proactive Welfare Agent

Agents that reach out to citizens proactively (push, not pull):

```python
class ProactiveWelfareAgent:
    
    @scheduled("0 8 * * *")  # Daily at 8am
    async def identify_eligible_citizens(self):
        # Query registry for citizens who may be newly eligible
        candidates = await self.tools.registry_query(
            "citizens",
            filter={"life_event": "new_birth", "days_since": 90}
        )
        
        for citizen_id in candidates:
            # Check if already enrolled
            if not await self.tools.registry_query("enrollments", citizen_id):
                # AI generates personalized outreach
                message = await self.llm.generate_outreach(
                    citizen_id=citizen_id,
                    program="maternity_benefit",
                    channel="sms"
                )
                await self.tools.notify_send(citizen_id, message, channel="sms")
```

### Type 4: Audit and Oversight Agent

Agents that monitor DPI system health and detect anomalies:

```python
class AuditAgent:
    
    @event_handler("payment.batch.completed")
    async def audit_disbursement(self, batch: PaymentBatch):
        # Statistical analysis of batch
        anomalies = await self.ai.detect_anomalies(batch)
        
        if anomalies:
            report = await self.ai.generate_audit_report(anomalies)
            await self.notify_auditor(report, urgency="high")
    
    @scheduled("0 2 * * 1")  # Monday 2am
    async def weekly_bias_check(self):
        decisions = await self.tools.data_fetch("ai_decisions", last_days=7)
        bias_report = await self.ai.bias_analysis(decisions, protected_attributes=["gender", "region", "age_group"])
        await self.tools.notify_send("audit_team", bias_report)
```

## DPI Tool Definitions (OpenAI-Compatible Format)

```json
[
  {
    "type": "function",
    "function": {
      "name": "identity_verify",
      "description": "Verify a citizen's identity. Returns verified=true/false and a verified_id token.",
      "parameters": {
        "type": "object",
        "required": ["citizen_id", "auth_factor"],
        "properties": {
          "citizen_id": { "type": "string", "description": "The citizen's national identifier" },
          "auth_factor": { "type": "string", "enum": ["otp", "biometric", "pin"] },
          "otp_value": { "type": "string", "description": "Required if auth_factor=otp" }
        }
      }
    }
  },
  {
    "type": "function",
    "function": {
      "name": "registry_query",
      "description": "Query an authorized DPI registry. Returns structured registry data.",
      "parameters": {
        "type": "object",
        "required": ["registry_name", "query_params"],
        "properties": {
          "registry_name": { "type": "string", "enum": ["benefits", "health_facilities", "businesses", "land"] },
          "query_params": { "type": "object" }
        }
      }
    }
  },
  {
    "type": "function",
    "function": {
      "name": "payment_initiate",
      "description": "Initiate a G2P payment. CAUTION: This is irreversible. Confirm with user before calling.",
      "parameters": {
        "type": "object",
        "required": ["recipient_id", "amount", "currency", "purpose_code", "authorization_token"],
        "properties": {
          "recipient_id": { "type": "string" },
          "amount": { "type": "number" },
          "currency": { "type": "string" },
          "purpose_code": { "type": "string" },
          "authorization_token": { "type": "string", "description": "Token from verified identity step" }
        }
      }
    }
  }
]
```

## Governance Constraints for Public Agents

### Mandatory Constraints

**1. Consent Gate**
No agent may call a DPI data exchange API without a valid, in-scope consent token:
```python
async def data_fetch_with_consent_gate(citizen_id: str, data_type: str, purpose: str):
    consent = await consent_manager.check(citizen_id, data_type, purpose)
    if not consent.valid:
        raise ConsentRequired(f"Citizen consent required for {data_type} for {purpose}")
    return await dpi.data_exchange.fetch(citizen_id, data_type, consent.token)
```

**2. Confirmation Before Irreversible Actions**
Any action that moves money, updates a registry record, or issues a credential must request explicit confirmation before execution:
```python
IRREVERSIBLE_TOOLS = {"payment_initiate", "registry_enroll", "credential_issue", "registry_deregister"}

async def agent_tool_call(tool_name: str, params: dict, context: AgentContext):
    if tool_name in IRREVERSIBLE_TOOLS:
        confirmation = await context.request_confirmation(
            f"About to: {describe_action(tool_name, params)}\nConfirm? [yes/no]"
        )
        if not confirmation:
            return {"status": "cancelled", "reason": "user_declined"}
    return await execute_tool(tool_name, params)
```

**3. Human Escalation**
Every agent must have an escalation path to a human operator:
```python
ESCALATION_TRIGGERS = [
    "confidence < 0.7",           # Low confidence AI decision
    "amount > HIGH_VALUE_THRESHOLD",  # High-value transaction
    "complaint or grievance",      # Formal complaint
    "data_breach_suspected",       # Security concern
    "protected_group_decision",    # Decisions affecting protected groups
]
```

**4. Audit Trail (Mandatory)**
Every agent action is logged:
```python
@audit_logged
async def agent_action(tool_name: str, params: dict, result: dict, context: dict):
    # audit_logged decorator writes:
    # { timestamp, session_id, agent_id, citizen_id_hash, tool, params_sanitized, result_summary, model_version }
    pass
```

**5. Rate Limits**
Prevent abuse of DPI rails via agent:
```python
rate_limits = {
    "identity_verify": { "per_session": 5, "per_citizen_per_day": 10 },
    "payment_initiate": { "per_session": 1, "per_citizen_per_day": 3 },
    "data_fetch": { "per_session": 20, "per_citizen_per_day": 100 }
}
```

## Multi-Agent Architecture for Complex DPI Services

For services that span multiple DPI domains, use a multi-agent architecture:

```
Citizen Request
      ↓
Orchestrator Agent (routes to specialist agents)
  ├── Identity Agent (handles authentication, KYC)
  ├── Welfare Agent (handles benefit applications)
  ├── Health Agent (handles health services)
  ├── Payment Agent (handles disbursements)
  └── Grievance Agent (handles complaints)
```

```python
class DPIOrchestratorAgent:
    
    agents = {
        "identity": IdentityAgent(),
        "welfare": WelfareAgent(),
        "health": HealthAgent(),
        "payment": PaymentAgent(),
        "grievance": GrievanceAgent()
    }
    
    async def route(self, request: CitizenRequest) -> AgentResponse:
        domain = await self.llm.classify_domain(request.text)
        
        # Authenticated session is shared across agents
        if not request.session.authenticated:
            auth = await self.agents["identity"].authenticate(request)
            request.session.auth_token = auth.token
        
        return await self.agents[domain].handle(request)
```

## Deployment Patterns

### Stateful Agent Session (for multi-turn conversations)
```python
# Redis-backed session state for stateful agents
class AgentSession:
    def __init__(self, session_id: str):
        self.id = session_id
        self.auth_token: Optional[str] = None
        self.conversation_history: list = []
        self.pending_confirmations: list = []
        self.ttl = timedelta(minutes=30)  # Session expires after 30 min inactivity
    
    async def save(self):
        await redis.setex(f"session:{self.id}", self.ttl, self.serialize())
```

### Channel Adapters
Same agent, multiple channels:
```python
channels = {
    "web": WebChatAdapter(agent=dpi_agent),       # Browser chat widget
    "sms": SMSAdapter(agent=dpi_agent),            # SMS/USSD for feature phones
    "whatsapp": WhatsAppAdapter(agent=dpi_agent),  # WhatsApp Business API
    "voice": IVRAdapter(agent=dpi_agent),          # IVR for voice calls
    "app": MobileAppAdapter(agent=dpi_agent)       # Mobile app SDK
}
```

## Key Standards and Frameworks

- **OpenAI Tool Use / Function Calling**: Standard API for LLM tool definitions
- **Anthropic Tool Use (Claude)**: Tool use with explicit confirmation patterns
- **LangGraph**: Stateful multi-agent workflows
- **AutoGen (Microsoft)**: Multi-agent conversation frameworks
- **Agency Swarm**: Role-based multi-agent coordination
- **Model Context Protocol (MCP)**: Standard for connecting agents to data sources — see dpi-mcp-dpg skill
