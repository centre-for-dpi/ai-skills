---
name: dpi-mcp-dpg
description: DPGs as AI Blocks via MCP and REST APIs — 14 DPGs (MOSIP, Mojaloop, OpenFn, DHIS2, Sunbird RC). Use when connecting AI to DPI infrastructure or designing MCP-based DPG integrations.
---

# Digital Public Goods as AI Blocks — MCP & API Readiness

> **Reference**: https://digitalpublicinfrastructure.ai/dpgs.php  
> **DPI-AI Framework**: https://digitalpublicinfrastructure.ai/

## The Core Vision: DPGs as Callable AI Blocks

> *"DPGs and AI Blocks function as interoperable, callable components that can be dynamically assembled by Public Agents and orchestrated through DPI Workflows, becoming infrastructure-native services that are discoverable, composable, and adaptable across jurisdictions."* — DPI-AI Framework

The DPI-AI Framework treats **Digital Public Goods as the primary source of AI Blocks** for DPI Workflows. Instead of building custom AI integrations, governments connect existing open-source DPGs to AI Workflows via their APIs or MCP servers.

```
Public Agent (citizen interface)
        ↓
DPI Workflow (locus of control)
        ↓
DPG as AI Block  ←── open-source, callable, governed
  MOSIP          → identity_verify(), get_kyc()
  Mojaloop       → initiate_payment(), check_status()
  OpenFn         → trigger_workflow()          [MCP native ✓]
  Sunbird RC     → registry_search(), attest()
  DHIS2          → health_record_fetch()
  OpenG2P        → eligibility_check(), enroll()
        ↓
DPI Rail (authoritative action recorded)
```

## DPG Catalogue: AI Block & MCP Readiness

The DPI-AI Framework catalogues **14 DPGs** as callable AI Blocks across 12 sectors. Each DPG is assessed for API readiness (can be called by a DPI Workflow) and MCP readiness (exposes a native MCP server for direct AI agent invocation).

| DPG | Sector | AI Block Capabilities | API Ready | MCP Ready |
|-----|--------|----------------------|-----------|-----------|
| **OpenFn** | Workflow / Integration | Trigger workflows, inspect logs, query adaptor schemas, DPG-to-DPG exchange | ✓ | ✓ Native (June 2025) — first DPG |
| **MOSIP** | Identity | identity_verify, biometric_match, send_otp, get_kyc, deduplication | ✓ | In progress |
| **Mojaloop** | Payments | initiate_payment, check_status, financial_address_lookup, G2P disburse | ✓ | In progress |
| **DHIS2** | Health data | health_record_fetch, program_enroll, aggregate_report, tracker_query | ✓ | In progress |
| **Sunbird RC** | Registries | registry_search, registry_write, credential_issue, verify_attestation | ✓ | In progress |
| **OpenG2P** | Social protection | eligibility_check, beneficiary_enroll, benefit_disburse, grievance_log | ✓ | In progress |
| **OpenMRS** | Clinical health | patient_record_fetch, encounter_create, clinical_data_query (FHIR) | ✓ | In progress |
| **OpenCRVS** | Civil registration | birth_record_get, death_record_get, registration_status | ✓ | In progress |
| **X-Road** | Data exchange | service_query, data_exchange_route, list_services (G2G/G2B) | ✓ | In progress |
| **Diia** | Digital government | document_verify, credential_present, citizen_auth | ✓ | In progress |
| **QuarkID** | Verifiable credentials | credential_issue, credential_verify, ZKP_present | ✓ | In progress |
| **OpenSPP** | Social protection | Integrates natively with MOSIP (identity), Mojaloop (payments), OpenCRVS (civil records) | ✓ | In progress |
| **e-Signet** | Authentication | OIDC_authenticate, get_userinfo, ekyc_share | ✓ | In progress |
| **Inji** | VC wallet / issuance | credential_issue (OpenID4VCI), credential_verify, QR_offline_present | ✓ | In progress |

### OpenFn: First DPG with Native MCP (June 2025)

OpenFn is the most advanced DPG for AI agent integration:

- **Native MCP server**: AI agents invoke OpenFn workflows directly via MCP tool calls
- **AI agents can**: trigger workflows, inspect run logs, query adaptor schemas — all via MCP
- **AI Assistant**: plain-language workflow generation ("describe a workflow, OpenFn generates it")
- **Pre-built adaptors**: OpenCRVS, DHIS2, Mojaloop, Kobo, CommCare, Salesforce, OpenMRS, and 60+ more
- **Effect**: AI agent → OpenFn MCP → DHIS2 / OpenMRS / Mojaloop / identity system (chained DPG calls in one agent turn)

### Sunbird RC: Auto-Generated APIs

Sunbird RC auto-generates CRUD REST APIs from schema configuration — no custom backend required. Any registry schema defined in Sunbird RC instantly becomes a callable AI Block with no additional engineering.

## DPG Qualification Criteria as AI Blocks

For a DPG to qualify as an AI Block in the DPI-AI Framework it must:

| Criterion | Requirement |
|-----------|-------------|
| **Open source** | Licensed under an OSI-approved licence |
| **Training data provenance** | Any AI components declare their training data under an open licence |
| **Privacy and consent controls** | Embeds consent verification before data access |
| **Replaceability** | Designed for modularity — no lock-in; any conforming implementation can replace it |
| **SDG alignment** | Demonstrably helps attain one or more Sustainable Development Goals |

## The DPG AI Block Catalogue: 30 Use Cases × 12 Sectors

The DPI-AI Framework's service architect maps **30 use cases** across **12 sectors**:

| Sector | Example Use Cases |
|--------|-----------------|
| **Social Services** | Benefit eligibility, cash transfer, grievance resolution |
| **Civil Registration** | Birth/death registration, ID linkage, vital statistics |
| **Livelihoods** | Farmer credential, agricultural input access, microfinance |
| **Education** | School enrollment, learning credential, scholarship |
| **Health** | Immunisation reminder, maternal care, disease surveillance |
| **Crisis Response** | Emergency benefit, displaced person registration, aid distribution |
| **Financial Inclusion** | Account onboarding, alternative credit, G2P disbursement |
| **Land & Property** | Title registration, land record query, property credential |
| **Justice** | Legal aid eligibility, case status, victim support |
| **Migration** | Permit verification, cross-border credential, refugee registration |
| **Tax & Revenue** | Compliance check, refund processing, SME registration |
| **Digital Identity** | ID verification, eKYC, progressive credential issuance |

The catalogue also covers **18 LLM/SLM models** to choose from when configuring the Public Agent for a given workflow.

## What is MCP and Why Does It Matter for DPI?

**Model Context Protocol (MCP)** is an open standard by Anthropic for connecting AI models (LLMs) to external tools, data sources, and services. It defines a standard interface that any LLM host (Claude, GPT, open models) can use to discover and invoke capabilities.

**Why MCP + DPI is significant:**

When DPI systems are exposed as MCP servers, any AI agent or LLM assistant that speaks MCP can instantly use DPI rails as tools — without custom integration code. This creates a standard layer where:

```
Any AI Agent (MCP Client)
        ↕ MCP Protocol
DPI System (MCP Server)
        ↕ DPI APIs
DPI Infrastructure (Identity / Payments / Data / Credentials)
```

This is the "MCP layer" of the DPI-AI stack — a standardized bridge between AI intelligence and DPI authority.

## MCP Architecture

### MCP Server Components

```
MCP Server
  ├── Tools        (callable functions — DPI actions)
  ├── Resources    (readable data — DPI registries, policy docs)
  └── Prompts      (pre-built prompt templates for DPI workflows)
```

### Transport Options
- **stdio**: Local process communication (Claude Code, local LLMs)
- **HTTP + SSE**: Remote server communication (production DPI services)
- **WebSocket**: Bidirectional real-time (streaming DPI responses)

## Designing a DPI MCP Server

### Example: Identity DPI MCP Server

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const identityMCPServer = new Server({
  name: "dpi-identity",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {},
    resources: {}
  }
});

// Tool: Verify identity
identityMCPServer.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "verify_citizen_identity",
      description: "Verify a citizen's identity using OTP or biometric. Returns verified status and a session token.",
      inputSchema: {
        type: "object",
        required: ["citizen_id", "auth_type"],
        properties: {
          citizen_id: {
            type: "string",
            description: "The citizen's national identifier"
          },
          auth_type: {
            type: "string",
            enum: ["otp", "biometric"],
            description: "Authentication method"
          },
          otp_value: {
            type: "string",
            description: "Required when auth_type=otp"
          }
        }
      }
    },
    {
      name: "get_kyc_attributes",
      description: "Retrieve verified KYC attributes for an authenticated citizen. Requires prior identity verification.",
      inputSchema: {
        type: "object",
        required: ["session_token", "requested_attributes"],
        properties: {
          session_token: {
            type: "string",
            description: "Token from verify_citizen_identity"
          },
          requested_attributes: {
            type: "array",
            items: { type: "string" },
            description: "List of attributes: name, dob, address, phone, email"
          }
        }
      }
    }
  ]
}));

// Handle tool execution
identityMCPServer.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  
  switch (name) {
    case "verify_citizen_identity": {
      const result = await dpiIdentityAPI.authenticate(args.citizen_id, args.auth_type, args.otp_value);
      return {
        content: [{
          type: "text",
          text: JSON.stringify({
            verified: result.verified,
            session_token: result.token,
            message: result.verified ? "Identity verified" : "Verification failed"
          })
        }]
      };
    }
    
    case "get_kyc_attributes": {
      const attrs = await dpiIdentityAPI.getKYC(args.session_token, args.requested_attributes);
      return {
        content: [{ type: "text", text: JSON.stringify(attrs) }]
      };
    }
  }
});
```

### Example: Payment DPI MCP Server

```typescript
const paymentMCPServer = new Server({ name: "dpi-payments", version: "1.0.0" }, { capabilities: { tools: {} } });

// Payment tools
const paymentTools = [
  {
    name: "get_payment_status",
    description: "Check the status of a payment by transaction ID.",
    inputSchema: {
      type: "object",
      required: ["transaction_id"],
      properties: {
        transaction_id: { type: "string" }
      }
    }
  },
  {
    name: "initiate_g2p_payment",
    description: "IMPORTANT: Initiates an irreversible government-to-person payment. Only call after explicit user confirmation.",
    inputSchema: {
      type: "object",
      required: ["recipient_id", "amount_minor_units", "currency_code", "purpose_code", "auth_token"],
      properties: {
        recipient_id: { type: "string", description: "Recipient's national identifier" },
        amount_minor_units: { type: "integer", description: "Amount in minor currency units (e.g., cents)" },
        currency_code: { type: "string", description: "ISO 4217 currency code" },
        purpose_code: { type: "string", description: "G2P purpose code from approved list" },
        auth_token: { type: "string", description: "Authorization token from identity verification" }
      }
    }
  }
];
```

### Example: Credential DPI MCP Server

```typescript
const credentialMCPServer = new Server({ name: "dpi-credentials", version: "1.0.0" }, { capabilities: { tools: {}, resources: {} } });

const credentialTools = [
  {
    name: "verify_credential",
    description: "Verify a presented Verifiable Credential (VC). Returns validity, issuer, and attributes.",
    inputSchema: {
      type: "object",
      required: ["credential_jwt_or_json"],
      properties: {
        credential_jwt_or_json: {
          type: "string",
          description: "The VC in JWT or JSON-LD format"
        },
        expected_issuer_did: {
          type: "string",
          description: "Optional: expected issuer DID to verify against"
        }
      }
    }
  },
  {
    name: "issue_credential",
    description: "Issue a verifiable credential to a citizen. Requires authorization.",
    inputSchema: {
      type: "object",
      required: ["recipient_id", "credential_type", "attributes", "auth_token"],
      properties: {
        recipient_id: { type: "string" },
        credential_type: { type: "string" },
        attributes: { type: "object" },
        auth_token: { type: "string" }
      }
    }
  }
];

// Resource: Trust Registry
credentialMCPServer.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: "dpi://trust-registry/issuers",
      name: "Trusted Credential Issuers",
      description: "List of all trusted credential issuers and their DID documents",
      mimeType: "application/json"
    },
    {
      uri: "dpi://trust-registry/schemas",
      name: "Credential Schemas",
      description: "All published credential schemas",
      mimeType: "application/json"
    }
  ]
}));
```

## DPG MCP Server Registry

A vision for DPI: a **public registry of DPG-backed MCP servers** that any AI agent can discover and connect to:

```json
// DPG MCP Registry entry
{
  "dpg_name": "MOSIP",
  "mcp_servers": [
    {
      "name": "mosip-identity",
      "transport": "http+sse",
      "endpoint": "https://mcp.mosip.example.gov/",
      "authentication": "oauth2",
      "tools": ["verify_citizen_identity", "get_kyc_attributes", "send_otp"],
      "regions": ["PH", "ET", "MA"],
      "compliance": ["GDPR-equivalent", "ISO27001"]
    }
  ]
}
```

### DPGs as MCP Servers

| DPG | MCP Server Role | Key Tools |
|-----|----------------|----------|
| **OpenFn** | Workflow orchestration (native MCP ✓) | trigger_workflow, get_run_log, query_adaptor_schema |
| **MOSIP** | Identity verification | verify_identity, get_kyc, send_otp, biometric_match |
| **e-Signet** | OIDC authentication | authenticate, get_userinfo, ekyc_share |
| **Inji Certify** | Credential issuance | issue_credential, get_credential_status |
| **Inji Verify** | Credential verification | verify_credential, check_revocation |
| **OpenG2P** | Social protection | check_eligibility, enroll_beneficiary, disburse |
| **Sunbird RC** | Registry queries (auto-API) | registry_lookup, registry_write, verify_attestation |
| **OpenCRVS** | Civil registration | get_birth_record, get_death_record, registration_status |
| **DHIS2** | Health data | fetch_health_record, enroll_program, aggregate_report |
| **Mojaloop** | Payments | initiate_payment, check_status, financial_address_lookup |
| **X-Road** | Data exchange routing | query_service, list_services, data_exchange_route |
| **CREDEBL** | VC platform (multi-tenant) | issue_credential, verify_presentation, revoke |
| **QuarkID** | ZKP credentials | issue_credential, zkp_present, verify |
| **Diia** | Digital government (Ukraine) | document_verify, citizen_auth, credential_present |

## Security for DPI MCP Servers

### Authentication Patterns

**OAuth 2.0 + PKCE for MCP Servers**:
```typescript
// MCP server with OAuth 2.0 authentication
const server = new Server({ name: "dpi-identity" }, {
  capabilities: { tools: {} }
});

// Middleware: validate OAuth bearer token before any tool call
server.use(async (request, next) => {
  const token = extractBearerToken(request.headers.authorization);
  if (!token) throw new McpError(ErrorCode.InvalidRequest, "Missing authorization");
  
  const claims = await oauth.introspect(token);
  if (!claims.active) throw new McpError(ErrorCode.InvalidRequest, "Invalid token");
  
  request.context.claims = claims;
  return next(request);
});
```

**Scoped Permissions**:
```typescript
const TOOL_SCOPES = {
  "verify_citizen_identity": "identity:authenticate",
  "get_kyc_attributes": "identity:kyc:read",
  "initiate_g2p_payment": "payments:g2p:write",
  "issue_credential": "credentials:issue:write"
};

function assertScope(claims: TokenClaims, tool: string) {
  const requiredScope = TOOL_SCOPES[tool];
  if (!claims.scopes.includes(requiredScope)) {
    throw new McpError(ErrorCode.InvalidRequest, `Missing scope: ${requiredScope}`);
  }
}
```

### Audit Logging for MCP Tool Calls
```typescript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  
  // Log before execution
  const auditId = await auditLog.start({
    tool: name,
    args: sanitize(args),  // Remove PII before logging
    caller: request.context.claims.sub,
    timestamp: new Date().toISOString()
  });
  
  try {
    const result = await executeTool(name, args, request.context);
    await auditLog.complete(auditId, { success: true });
    return result;
  } catch (error) {
    await auditLog.complete(auditId, { success: false, error: error.message });
    throw error;
  }
});
```

## Claude Code Integration with DPI MCP Servers

Configure DPI MCP servers in `.claude/settings.json`:

```json
{
  "mcpServers": {
    "dpi-identity": {
      "type": "http",
      "url": "https://mcp.identity.example.gov/",
      "headers": {
        "Authorization": "Bearer ${DPI_IDENTITY_TOKEN}"
      }
    },
    "dpi-payments": {
      "type": "http",
      "url": "https://mcp.payments.example.gov/",
      "headers": {
        "Authorization": "Bearer ${DPI_PAYMENT_TOKEN}"
      }
    },
    "dpi-credentials": {
      "type": "http",
      "url": "https://mcp.credentials.example.gov/"
    }
  }
}
```

Once configured, Claude can:
```
User: "Verify that citizen ID 12345 is eligible for the housing benefit"
Claude: [calls dpi-identity:verify_citizen_identity, then dpi-registry:check_eligibility]
→ "Citizen ID 12345 has been verified. They are eligible for the housing benefit based on their income of..."
```

## Building a DPG as an MCP-Ready System

### Design Checklist for MCP-Ready DPGs

- [ ] All public APIs described in OpenAPI 3.x
- [ ] Authentication via OAuth 2.0 with scoped tokens
- [ ] Tool-friendly error messages (structured JSON, actionable descriptions)
- [ ] Idempotency keys supported on write operations
- [ ] Sandbox environment available for agent testing
- [ ] Rate limits documented and communicated in response headers
- [ ] Tool descriptions are precise about side effects and irreversibility
- [ ] PII handling documented (what data leaves the system in responses)
- [ ] Audit trail for all tool invocations
- [ ] MCP server wrapper published as open source

## MCP SDK Resources

- **MCP Specification**: https://modelcontextprotocol.io/
- **TypeScript SDK**: https://github.com/modelcontextprotocol/typescript-sdk
- **Python SDK**: https://github.com/modelcontextprotocol/python-sdk
- **MCP Inspector** (testing tool): https://github.com/modelcontextprotocol/inspector
- **Claude Code MCP Docs**: https://docs.anthropic.com/en/docs/claude-code/mcp
