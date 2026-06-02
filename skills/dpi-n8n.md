---
name: dpi-n8n
description: n8n — self-hostable visual workflow automation with 400+ nodes and AI agent capabilities. Use when building DPI data workflows, automating government processes, or designing AI pipelines.
---

# n8n — AI-Native Workflow Automation for DPI

## What is n8n?

n8n is a **fair-code workflow automation platform** that combines a visual node-based editor with full code extensibility and native AI agent capabilities. It is particularly suited for DPI programs that need to orchestrate AI models alongside DPI API calls — the bridge between LLM intelligence and DPI authority.

> **Official positioning**: "The flexibility of code with the speed of no-code" — 400+ integrations, native AI agents, fully self-hostable.

**Key links:**
- **Cloud**: https://n8n.cloud
- **Documentation**: https://docs.n8n.io
- **GitHub**: https://github.com/n8n-io/n8n (190,000+ stars, TypeScript)
- **Templates**: 900+ community workflow templates
- **License**: Sustainable Use License (fair-code) — self-host internally for free; cannot resell as SaaS
- **Quick start**: `npx n8n` or Docker

## Why n8n for DPI?

n8n addresses the gap between DPI-specific integration tools (like OpenFn) and AI-powered orchestration:

```
OpenFn:  DPI-to-DPI data exchange (DHIS2 ↔ MOSIP ↔ Registry)
n8n:     AI agent orchestration + DPI API calls + broad integrations

Combined:
  OpenFn handles DPG-to-DPG data flow
  n8n handles AI reasoning + citizen-facing automation
       ↕
  Both call DPI HTTP APIs
```

For DPI programs needing:
- **LLM-powered citizen service automation** → n8n
- **AI triage + DPI action** → n8n
- **Broad integration ecosystem** (400+ nodes) → n8n
- **Visual workflow design for non-developers** → n8n
- **Compliance in regulated environments** (zero data leaves your infra) → n8n self-hosted

## Core Concepts

### Nodes

Each node is a configurable building block representing one step in a workflow:

```
Trigger Node → Processing Nodes → Action Nodes

[Webhook]  →  [AI Agent]  →  [HTTP Request]  →  [Send SMS]
              (LLM decides      (calls DPI       (notifies
               what to do)       identity API)    citizen)
```

**Node categories:**
- **Trigger nodes**: Start workflows (Webhook, Cron/Schedule, Form, HTTP)
- **Data transformation**: Set, Merge, Split, Filter, Code
- **AI nodes**: AI Agent, LLM Chat, Embeddings, Vector Store, Memory
- **Integration nodes**: 400+ built-in (HTTP Request, PostgreSQL, Google Sheets, Slack, etc.)
- **Control flow**: If/Else, Switch, Loop, Error Trigger

### Workflows

Visual JSON-backed workflow definition — design in the UI, stored as JSON:

```json
{
  "name": "DPI Citizen Service Bot",
  "nodes": [
    {
      "id": "webhook-trigger",
      "type": "n8n-nodes-base.webhook",
      "parameters": { "path": "citizen-service", "httpMethod": "POST" }
    },
    {
      "id": "ai-agent",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "parameters": {
        "model": "claude-sonnet-4-6",
        "systemMessage": "You are a citizen service agent with access to DPI tools...",
        "tools": ["dpi-identity-verify", "dpi-registry-query", "dpi-payment-status"]
      }
    }
  ],
  "connections": {
    "webhook-trigger": { "main": [[ { "node": "ai-agent", "type": "main", "index": 0 } ]] }
  }
}
```

### Triggers

| Trigger | Description | DPI Example |
|---------|-------------|-------------|
| **Webhook** | HTTP POST to unique URL | Citizen submits form → workflow starts |
| **Schedule (Cron)** | Time-based execution | Daily G2P reconciliation |
| **Chat Trigger** | Conversational interface | Citizen types message → AI responds |
| **HTTP Request** | Poll external API | Check payment status every 5 min |
| **Form Trigger** | n8n-hosted web form | Complaint submission form |

### Credentials

Secure storage of secrets — API keys, OAuth tokens, database passwords:
```
n8n Credentials Store (encrypted at rest)
  ├── DPI Identity API Key
  ├── DPI Payment API OAuth Token
  ├── DHIS2 API credentials
  ├── Anthropic API key (for Claude)
  └── PostgreSQL connection string
```

## AI Agent Capabilities

n8n's AI Agent node is the primary differentiator for DPI use:

### LangChain-Based Agents

n8n uses LangChain under the hood for its AI Agent node, supporting:
- **ReAct agents**: Reason → Act → Observe loop (decides which tools to call)
- **Plan-and-execute agents**: Plan full approach then execute step by step
- **Conversational agents**: Multi-turn conversation with memory

### Supported LLM Providers

```
Anthropic    → Claude Sonnet, Opus, Haiku
OpenAI       → GPT-4o, o1, o3-mini
Google       → Gemini Pro, Flash
Mistral      → Mistral Large, Small
Cohere       → Command R+
Ollama       → Any local open-weight model (LLaMA, Mistral, Qwen, etc.)
Hugging Face → Inference API
```

**For DPI sovereignty**: Use **Ollama** to run open-weight models locally — no citizen data leaves your infrastructure.

### Memory Options

| Memory Type | Description | DPI Use Case |
|-------------|-------------|-------------|
| **Buffer Memory** | In-session conversation history | Multi-turn citizen service chat |
| **Windowed Memory** | Last N messages | Token-efficient conversation |
| **PostgreSQL Memory** | Persistent across sessions | Case history, citizen preferences |
| **Vector Store Memory** | Semantic search over past interactions | "Have I talked to this citizen before about X?" |

### DPI Tool Definitions for AI Agent

Define DPI APIs as tools the AI agent can invoke:

```javascript
// Custom tool: DPI Identity Verification
const identityVerifyTool = {
  name: "verify_identity",
  description: "Verify a citizen's identity using their ID number and OTP. Returns verified=true/false. Only call after getting explicit consent from the citizen.",
  schema: {
    type: "object",
    required: ["citizen_id", "otp"],
    properties: {
      citizen_id: { type: "string", description: "National ID number" },
      otp: { type: "string", description: "6-digit OTP sent to registered phone" }
    }
  }
};

// The AI agent will call this tool when appropriate
// n8n routes the call to an HTTP Request node that calls the DPI identity API
```

### RAG (Retrieval-Augmented Generation)

n8n supports full RAG pipelines for grounding AI responses in DPI policy documents:

```
Policy PDFs/Docs
      ↓
[Embeddings Node] → vector representations
      ↓
[Vector Store Node] → Qdrant/Pinecone/pgvector
      ↓
[AI Agent] ← retrieves relevant policy context at query time
      ↓
Grounded response (no hallucination of policy details)
```

## Deployment

### Self-Hosted (Recommended for DPI)

**Docker (quickest start):**
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
# Editor at: http://localhost:5678
```

**Docker Compose (production-ready):**
```yaml
version: '3.8'
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - WEBHOOK_URL=https://n8n.yourdomain.gov
      - N8N_PAYLOAD_SIZE_MAX=64  # 64MB max payload for large DPI responses
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on: [postgres]

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: n8n
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

volumes:
  n8n_data:
```

**Kubernetes (Helm):**
```bash
helm install n8n oci://ghcr.io/n8n-io/n8n-helm-chart/n8n \
  --version 1.0.0 \
  -f n8n-values.yaml
```

**Production scaling — Queue Mode:**
For high-volume DPI workflows (50,000+ monthly executions):
```yaml
# Enable queue mode: separate workers from the main process
N8N_EXECUTIONS_MODE: queue
QUEUE_BULL_REDIS_HOST: redis
# Workers scale horizontally independently of the editor
```

### n8n Cloud (SaaS)
Managed at n8n.cloud — fastest to start, no infrastructure management. Less suitable for DPI workloads requiring data sovereignty.

## DPI Workflow Patterns in n8n

### Pattern 1: AI-Powered Citizen Service Chat

```
[Chat Trigger / Webhook]
         ↓
[AI Agent Node]
  - System prompt: "You are a citizen service agent for [Country] government..."
  - Tools: verify_identity, check_benefit_status, query_registry, initiate_grievance
  - Memory: PostgreSQL (persistent session)
  - LLM: Claude (Anthropic) or Ollama (local)
         ↓
[AI decides to call DPI tools based on citizen request]
         ↓
[HTTP Request → DPI Identity API]  [HTTP Request → Registry API]
         ↓
[AI generates personalized response in citizen's language]
         ↓
[Send SMS / WhatsApp / Email]
```

### Pattern 2: Automated Benefits Eligibility Check

```
[Schedule Trigger: Daily 6am]
         ↓
[PostgreSQL: Get pending applications]
         ↓
[Loop over each application]
         ↓
  [HTTP: DPI Identity Verify]
         ↓
  [HTTP: Fetch income data (AA/consent)]
         ↓
  [AI Agent: Assess eligibility against policy (RAG)]
         ↓
  [If: eligible?]
    YES → [HTTP: Enroll in registry] → [Send approval SMS]
    NO  → [Send rejection SMS with reason]
         ↓
[PostgreSQL: Update application status]
```

### Pattern 3: Payment Reconciliation with Exception Handling

```
[Schedule Trigger: After G2P batch]
         ↓
[HTTP: Fetch payment batch results from payment rail]
         ↓
[Split: Process each transaction]
         ↓
  [If: Payment status = FAILED]
    ↓
  [AI Agent: Diagnose failure reason]
    - "Account closed" → update registry, notify
    - "Insufficient routing info" → fetch updated account from identity system
    - "Technical error" → add to retry queue
         ↓
  [HTTP: Initiate retry OR update registry]
         ↓
[Merge: Consolidate results]
         ↓
[Generate reconciliation report]
         ↓
[Send report to finance officer (email)]
```

### Pattern 4: Document-Triggered Credential Issuance

```
[Webhook: Document verified in registry]
         ↓
[HTTP: Fetch citizen details from identity API]
         ↓
[HTTP: Issue Verifiable Credential (Inji Certify / CREDEBL)]
         ↓
[HTTP: Store credential reference in registry]
         ↓
[Send notification: "Your [document] is now available in your digital wallet"]
```

## n8n Code Node (When Visual Isn't Enough)

For complex transformations, use the Code node (JavaScript/Python):

```javascript
// Code node: Transform DPI payment batch for reconciliation
const payments = items[0].json.paymentBatch;

// Group by status, calculate totals
const summary = payments.reduce((acc, payment) => {
  const status = payment.status || 'UNKNOWN';
  if (!acc[status]) acc[status] = { count: 0, totalAmount: 0 };
  acc[status].count++;
  acc[status].totalAmount += payment.amount;
  return acc;
}, {});

// Flag high-value failures for immediate escalation
const highValueFailures = payments.filter(p => 
  p.status === 'FAILED' && p.amount > 10000
);

return [{
  json: {
    summary,
    highValueFailures,
    totalProcessed: payments.length,
    failureRate: payments.filter(p => p.status === 'FAILED').length / payments.length
  }
}];
```

## HTTP Request Node for DPI APIs

The HTTP Request node connects to any DPI API without custom code:

```
HTTP Request Node Configuration:
  Method: POST
  URL: https://identity.example.gov/v1/auth/otp
  Authentication: Bearer Token (from Credentials)
  Headers:
    Content-Type: application/json
    X-Transaction-ID: {{ $execution.id }}
  Body:
    {
      "citizen_id": "{{ $json.citizenId }}",
      "otp": "{{ $json.otpValue }}",
      "txn_id": "{{ $execution.id }}"
    }
  Response mapping:
    verified → {{ $json.status === 'Y' }}
```

## Security for DPI Deployments

```bash
# Environment variables for secure n8n DPI deployment
N8N_ENCRYPTION_KEY=64-char-random-key-for-credentials-encryption
N8N_USER_MANAGEMENT_JWT_SECRET=jwt-secret-key
N8N_AUTH_EXCLUDE_ENDPOINTS=webhook  # Webhooks don't require auth by default
DB_POSTGRESDB_SSL_ENABLED=true      # Encrypt DB connection

# Network: n8n behind reverse proxy (nginx/traefik) with TLS
# Webhooks: Use signed webhook secrets to validate inbound DPI events
```

## n8n vs OpenFn for DPI

| Dimension | n8n | OpenFn |
|-----------|-----|--------|
| **AI agent capability** | Native, powerful (LangChain-based) | Limited |
| **DPG-specific adaptors** | Via HTTP node (generic) | 70+ pre-built DPI adaptors |
| **Visual editor** | Rich, node-based | Yes, but more code-oriented |
| **License** | Fair-code (Sustainable Use) | AGPLv3 (open source) |
| **DPG status** | Not a registered DPG | Registered Digital Public Good |
| **Best for** | AI workflows, citizen automation | G2G data exchange, DPG-to-DPG |
| **Kafka triggers** | No (use webhook adapter) | Yes |
| **Community** | 190K GitHub stars, 900+ templates | Smaller, government-focused |
| **Data sovereignty** | Full self-host | Full self-host |
| **Complementary?** | Yes — use together | Yes — use together |

**Recommended combination**: OpenFn for DPG-to-DPG data exchange (DHIS2, OpenMRS, identity systems) + n8n for AI-powered citizen service automation and broader integration workflows.

## Key Templates for DPI (Community)

Browse https://n8n.io/workflows/ for:
- "AI Customer Service Agent" — adapt for citizen service
- "Webhook + HTTP Request + Database" — DPI API integration pattern
- "AI Agent with Memory" — persistent citizen sessions
- "RAG Q&A Pipeline" — government policy chatbot
- "Scheduled Data Sync" — DPI registry synchronization
