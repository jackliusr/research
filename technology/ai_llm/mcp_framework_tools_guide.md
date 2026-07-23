# The Model Context Protocol (MCP) — A Comprehensive Guide

> **An open standard by Anthropic (announced November 25, 2024) for connecting LLMs to external tools, data sources, and services. Donated to the Linux Foundation's Agentic AI Foundation (AAIF) in December 2025.**

---

## 1. What Is MCP?

The **Model Context Protocol (MCP)** is an open, standardised protocol that defines how AI applications — particularly Large Language Models (LLMs) — discover and interact with external tools, data sources, and services. It provides a universal, interoperable interface between LLM hosts and the systems they need to access.

### The USB-C Analogy

Just as USB-C standardises how hardware peripherals connect to computers — any USB-C device works with any USB-C port — MCP standardises how AI applications connect to tools and data. Instead of every tool requiring a custom integration (LangChain adapter, OpenAI function-call schema, bespoke API wrapper), MCP provides a single protocol that any MCP-compliant client can use with any MCP-compliant server.

| Hardware World | AI World |
|---|---|
| USB-C port (standard connector) | MCP Client (standard protocol consumer) |
| USB-C device (keyboard, monitor, drive) | MCP Server (tool, data source, service) |
| Hot-plug and discover | Dynamic tool/resource discovery |
| One cable, many device types | One protocol, many tool types |

### Key Characteristics

- **Open standard** — Not vendor-owned. Developed by Anthropic, donated to the Linux Foundation AAIF in 2025.
- **Protocol, not a library** — Defines a wire format (JSON-RPC 2.0) over configurable transports.
- **LLM-agnostic** — Works with Claude, GPT, Gemini, open-source models, and any LLM.
- **Bidirectional** — Clients invoke tools on servers; servers can request LLM sampling.

### Brief History

- **Nov 25, 2024** — Anthropic announces MCP, open-sources spec, Python/TypeScript SDKs, 7 reference servers.
- **Early 2025** — Major IDEs (Cursor, VS Code, Zed) and assistants (Continue.dev, Cody) adopt MCP.
- **Mid 2025** — Spec v2025-06-18 with refined auth, streaming, lifecycle management.
- **Dec 2025** — Donated to Linux Foundation's Agentic AI Foundation (open governance).
- **2026** — 57,000+ MCP servers on GitHub, 1,000+ clients, widespread enterprise adoption.

---

## 2. Why MCP Matters

### 2.1 Standardisation

Before MCP, every LLM-tool integration was custom:

- **LangChain tools** tied to a specific library and runtime.
- **OpenAI Function Calling** bound to OpenAI's API surface.
- **Custom APIs** required bespoke endpoint design, auth, and documentation per tool.

MCP provides a **single protocol** that any LLM client can use to interact with any MCP-compliant server. A tool author writes one server implementation; every MCP client can use it.

### 2.2 Security

MCP introduces a layered security model:

- **Server-managed policies** — The MCP server decides which tools and resources to expose and to whom.
- **Client-enforced boundaries** — The MCP client enforces which tools the LLM may actually invoke.
- **Standardised authentication** — Remote servers use OAuth 2.0 / OIDC; local servers rely on OS-level permissions.
- **Audit trail** — Every tool invocation can be logged with input parameters, result status, and timestamps.

### 2.3 Discoverability

MCP servers advertise their capabilities on connection:

```
Client connects → Server sends capabilities (tools, resources, prompts)
  → LLM sees available tools → Client invokes tool → Server returns results
```

LLM clients can dynamically discover what tools are available without hard-coded configuration. When a server adds a new tool, every connected client sees it immediately.

### 2.4 Composability

Multiple MCP servers can be connected to the same client simultaneously. A **filesystem server** provides local file access, a **PostgreSQL server** provides database querying, a **GitHub server** provides repository operations, and a **Slack server** provides messaging — all visible to the LLM as a unified tool surface.

### 2.5 Ecosystem

| Metric | Nov 2024 | Apr 2025 | Mid 2026 |
|---|---|---|---|
| MCP servers | 7 (official) | 5,800+ | 57,000+ |
| MCP clients | 2 | 300+ | 1,000+ |
| SDK downloads | — | 8M+/month | 50M+/month |

### 2.6 Portability

A tool built for MCP works with **any MCP-compliant client**: Claude Desktop, Cursor IDE, VS Code, Zed, Continue.dev, custom apps. Swapping the LLM backend (Claude → GPT-4o → local model) requires no changes to the MCP server.

---

## 3. Core MCP Concepts

### 3.1 MCP Server

A process that exposes capabilities to LLM clients. Each server implements the MCP protocol and advertises one or more of:

- **Tools** — Executable functions the LLM can invoke (name, description, JSON Schema input, output).
- **Resources** — Data sources the LLM can read (identified by URI, e.g., `file:///path`, `postgres:///users`).
- **Prompts** — Pre-defined prompt templates for reusable interaction patterns.

Servers manage their own security policies, access control, and error handling.

### 3.2 MCP Client

Connects to one or more servers, discovers capabilities, and mediates tool/resource access on behalf of an LLM. Responsibilities include establishing connections, discovering tools/resources/prompts, invoking tools and forwarding results, enforcing client-side security, and managing connection lifecycle. Examples: Claude Desktop's internal client, VS Code MCP extension, custom SDK-based applications.

### 3.3 MCP Host

The application that orchestrates the overall LLM interaction, hosting one or more MCP clients.

| Host | Client(s) | Use Case |
|---|---|---|
| Claude Desktop | Filesystem, GitHub, Postgres MCP clients | General AI assistant with tool access |
| VS Code (Copilot/Claude ext.) | GitHub, Filesystem, custom MCP clients | AI-assisted coding |
| Cursor IDE | Multiple MCP servers | AI code generation with context |
| Custom web app | Database MCP client, Search MCP client | Domain-specific AI tools |

### 3.4 Transports

How MCP clients and servers communicate:

| Transport | Direction | Use Case |
|---|---|---|
| **stdio** | Local (subprocess) | MCP server runs as child process. Fast, no network overhead. Best for local filesystem, local databases, CLI tools. |
| **SSE** (Server-Sent Events) | Remote (HTTP) | MCP server on separate host. Client connects via HTTP. Best for cloud APIs, remote databases, SaaS. |
| **WebSocket** | Bidirectional (planned) | Full-duplex for streaming and real-time updates. |

### 3.5 Protocol Flow

```
LLM → Client → Server

1. Client connects to server
2. Server sends capabilities (tools list, resources list)
3. Client forwards tool/resource list to LLM
4. LLM decides to invoke a tool → Client invokes tool on server
5. Server executes tool and returns result
6. Client forwards result to LLM
7. LLM incorporates result into response
```

### 3.6 Tool Definition (JSON Schema Example)

Each tool has a name, description, and `inputSchema` defined in JSON Schema:

```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "inputSchema": {
    "type": "object",
    "properties": {
      "location": { "type": "string", "description": "City name or coordinates" },
      "units": { "type": "string", "enum": ["celsius", "fahrenheit"], "default": "celsius" }
    },
    "required": ["location"]
  }
}
```

---

## 4. MCP Architecture Patterns

### 4.1 Local Server (stdio)

MCP server runs as **subprocess on same machine**.

**Use cases:** Local file access, SQLite queries, CLI tools.
**Pros:** Low latency, no network, OS-level security.
**Cons:** Local machine only, no horizontal scaling.

### 4.2 Remote Server (SSE)

MCP server runs on **separate host** via HTTP SSE.

**Use cases:** Cloud APIs, remote databases, SaaS services.
**Pros:** Centralised, scalable, multi-client access.
**Cons:** Network latency, auth required, TLS needed.

### 4.3 Gateway / Aggregator

Single gateway routes to multiple backend MCP servers.

```
Client → MCP Gateway (Auth/Routing/Logging) → Server A, Server B, Server C
```

**Use cases:** Enterprise deployments with centralised access control.
**Pros:** Single entry point, centralised security, audit logging.
**Cons:** Single point of failure (HA mitigates).

### 4.4 Embedded

MCP server embedded in-app, in-process communication.

**Use cases:** App-specific tools, single-binary deployments.
**Pros:** No IPC overhead, simple, tight security.
**Cons:** Not reusable by other clients.

### 4.5 Multi-Server Composition

Client connects to **multiple servers simultaneously**. Tools from all servers available to LLM. Cross-server resource references possible.

### 4.6 Decision Guide

| Scenario | Pattern | Why |
|---|---|---|
| Local files, dev tools | Local (stdio) | Fast, secure, zero-config |
| Cloud APIs, SaaS | Remote (SSE) | Accessible anywhere |
| Enterprise, multi-team | Gateway | Centralised auth, routing |
| Single app, internal | Embedded | Simplest, tightest security |
| Multi-domain workflows | Multi-server | Each domain = own server |

---

## 5. MCP Tools and Ecosystem

### 5.1 Official MCP Reference Servers

| Server | Capability |
|---|---|
| **Filesystem** | Read/write/search files and directories |
| **GitHub** | Repository ops, issues, PRs, commits, search |
| **PostgreSQL** | Query databases, schema inspection |
| **SQLite** | Local database operations |
| **Puppeteer** | Browser automation, web scraping |
| **Brave Search** | Web/local search via Brave API |
| **Slack** | Messaging, channels, search |
| **Google Drive** | File access and search |
| **Memory** | Persistent knowledge graph for LLM |
| **Sequential Thinking** | Structured multi-step reasoning |

### 5.2 Community-Built MCP Servers

| Category | Examples |
|---|---|
| **Dev & Code** | Git, GitLab, Jira, Linear, Docker, Kubernetes, Sentry |
| **Databases** | MySQL, MongoDB, Redis, Elasticsearch, Snowflake, BigQuery |
| **Cloud** | AWS (50+ svcs), GCP, Azure, Cloudflare, Vercel |
| **Productivity** | Google Workspace, M365, Notion, Confluence, Discord, Telegram |
| **AI & Data** | Hugging Face, Replicate, Pinecone, Chroma, Weaviate |
| **Finance** | Stripe, Shopify, Salesforce, HubSpot, Airtable |

### 5.3 MCP Clients

| Client | Type | Features |
|---|---|---|
| **Claude Desktop** | Desktop | Built-in MCP, config via `claude_desktop_config.json` |
| **Claude Code (CLI)** | Terminal | MCP via `.mcp.json`, stdio + SSE |
| **VS Code, Cursor, Zed** | IDEs | Native MCP support in settings |
| **Continue.dev, Cody** | OSS asst. | MCP via config.json |
| **Custom (SDK)** | Any app | Python or TS MCP client library |

### 5.4 MCP SDKs

| SDK | Maintainer | Features |
|---|---|---|
| **Python** (`fastmcp`/`mcp`) | Anthropic | Async, full client+server, decorator API |
| **TypeScript** (`@modelcontextprotocol/sdk`) | Anthropic | Full client+server, Transport abstraction |
| **Java, Go, Kotlin, Rust** | Community | Spring Boot, goroutines, coroutines, zero-cost |

### 5.5 MCP Services and Platforms

| Platform | Description |
|---|---|
| **Cloudflare Workers MCP** | Edge-deployed, auto-scaling MCP servers |
| **Glama.ai / Smithery.ai** | MCP server marketplaces with discovery, ratings, hosting |
| **Composio** | 250+ SaaS integrations via single MCP endpoint |
| **Nango** | Open-source OAuth for MCP servers |
| **MCPHub** | CLI tool for discovering/installing MCP servers |
| **Pipedream MCP** | Connect 2000+ apps as MCP tools |

---

## 6. MCP vs Other Tool Integration Approaches

### 6.1 Comparison Table

| Dimension | MCP | OpenAI FC | LangChain Tools | Custom APIs | Anthropic TU |
|---|---|---|---|---|---|
| **Type** | Open protocol | API feature | Library | Bespoke | API feature |
| **LLM Agnostic** | ✅ Yes | ❌ No | ⚠️ Partial | ⚠️ Depends | ❌ No |
| **Transport** | stdio, SSE, WS | HTTP | HTTP | Varies | HTTP |
| **Discoverability** | Built-in | None | None | None | None |
| **Security Model** | Explicit (srv+client) | API-key | App-level | App-level | API-key |
| **Composability** | Multi-server | Single call | Chain-based | None | Single call |
| **Ecosystem Size** | 57K+ servers | N/A | 700+ integs | N/A | N/A |
| **Open Source** | ✅ Linux Found. | ❌ | ✅ | Varies | ❌ |

### 6.2 Key Differences

**Protocol vs Library vs API Feature:**
- MCP is an **open protocol** (JSON-RPC wire format). Any implementation can interoperate.
- OpenAI Function Calling and Anthropic Tool Use are **API features** bound to their respective providers.
- LangChain Tools is a **library abstraction** bound to LangChain's runtime.

**Separation of Concerns:** MCP separates server implementation from client/LLM — one server works with any MCP client. Other approaches couple tool definitions to a specific client library or API surface.

**Security:** MCP has an explicit layered model — server controls access, client enforces boundaries, transport provides encryption. Other approaches rely on application-level security (often a single API key).

### 6.3 When to Use What

| Use Case | Best Approach |
|---|---|
| General-purpose AI assistant with many tools | **MCP** (composability, discoverability) |
| OpenAI-only, no plan to switch | OpenAI Function Calling (simpler, mature) |
| Existing LangChain codebase | LangChain tools |
| Single-purpose, tightly coupled integration | Custom API (simplest) |
| Enterprise with multiple LLM providers | **MCP** (LLM-agnostic) |
| Local/offline AI tools | **MCP with stdio** |

---

## 7. Implementing an MCP Server

### 7.1 Step 1 — Define Tools and Resources

Identify capabilities. For each tool, design name, description, and inputSchema (JSON Schema). For resources, define URI schemes (e.g., `weather://{location}/current`).

### 7.2 Step 2 — Choose Transport

- **stdio** for local/CLI tools
- **SSE** for remote/network services
- **WebSocket** for real-time bidirectional

Consider supporting both stdio and SSE for maximum compatibility.

### 7.3 Step 3 — Implement with SDK

#### Python (FastMCP)

```python
from fastmcp import FastMCP

mcp = FastMCP("Weather Service")

@mcp.tool()
def get_weather(location: str, units: str = "celsius") -> dict:
    """Get current weather for a location."""
    return {"location": location, "temperature": 22, "conditions": "Cloudy", "units": units}

@mcp.resource("weather://{location}/current")
def current_weather(location: str) -> str:
    return f"Weather for {location}: 22°C, Cloudy"

if __name__ == "__main__":
    mcp.run()  # stdio; for SSE: mcp.run(transport="sse", port=8000)
```

#### TypeScript

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { CallToolRequestSchema, ListToolsRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "weather-service", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{ name: "get_weather", description: "Current weather", inputSchema: {
    type: "object", properties: { location: { type: "string" }, units: { type: "string", enum: ["c", "f"] } }, required: ["location"]
  }}],
}));

server.setRequestHandler(CallToolRequestSchema, async (req) => {
  if (req.params.name !== "get_weather") throw new Error("Unknown tool");
  return { content: [{ type: "text", text: JSON.stringify({ temp: 22, conditions: "Cloudy" }) }] };
});

await server.connect(new StdioServerTransport());
```

### 7.4 Step 4 — Register Tools and Handle Errors

- Validate all inputs (required params, types, constraints).
- Handle errors gracefully with structured error messages.
- Never expose internal stack traces to the LLM.

```python
@mcp.tool()
def query_database(sql: str) -> list:
    if not sql.strip().upper().startswith("SELECT"):
        raise ValueError("Only SELECT queries are allowed")
    try:
        return execute_query(sql)
    except Exception as e:
        return {"error": f"Query failed: {str(e)}"}
```

### 7.5 Step 5 — Add Security

- **Authentication:** API keys, OAuth 2.0, OIDC, mTLS for remote servers.
- **Authorization:** Role-based access per tool/resource.
- **Input validation:** Sanitise all parameters (prevent injection).
- **Rate limiting:** Per-client or per-tool limits.
- **Audit logging:** Log every tool invocation with client ID, params, timestamp, result.

### 7.6 Step 6 — Test with MCP Client

**Claude Desktop** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "weather-service": {
      "command": "python",
      "args": ["-m", "weather_server"],
      "env": { "WEATHER_API_KEY": "your-api-key" }
    }
  }
}
```

**MCP Inspector** (debugging tool):
```bash
npx @modelcontextprotocol/inspector python -m weather_server
```

**Claude Code** (`.mcp.json`, per-project):
```json
{
  "mcpServers": {
    "weather-service": { "command": "uv", "args": ["run", "weather_server.py"] }
  }
}
```

### 7.7 Step 7 — Publish

- Document tools and resources with examples.
- Provide configuration snippets for Claude Desktop, VS Code, Cursor, etc.
- Publish to registries (Smithery.ai, Glama.ai, npm/PyPI).
- Provide a Docker image for easy deployment.
- Add health checks for monitoring.

---

## 8. MCP Security Considerations

### 8.1 Tool Access Control

| Layer | Responsibility | Mechanism |
|---|---|---|
| **Server** | Decides which tools/resources to expose | Capability advertisement, authentication |
| **Client** | Enforces which tools LLM can use | Tool allow/block-list, user approval |
| **Transport** | Encrypts communication | TLS 1.3 for remote (SSE/WebSocket) |
| **Host** | Manages user permissions | OS-level access control, user prompts |

### 8.2 Data Privacy

- **Server controls what data is sent to LLM** — sensitive fields can be filtered at server level.
- **Client can inspect tool calls and results** — redact sensitive data before forwarding to LLM.
- **Local-first design** — Even Claude Desktop runs MCP servers locally; no data sent to Anthropic servers.

### 8.3 Input Validation

Prevent injection attacks:

```python
@mcp.tool()
def search_logs(pattern: str, max_results: int = 100) -> list:
    safe_pattern = re.escape(pattern)       # prevent regex injection
    max_results = min(max_results, 1000)    # prevent resource exhaustion
    return log_database.search(safe_pattern, limit=max_results)
```

### 8.4 Output Filtering

Filter sensitive fields before returning to the LLM:

```python
SENSITIVE = {"password", "secret", "token", "api_key", "ssn"}

def sanitise(data):
    if isinstance(data, dict):
        return {k: sanitise(v) for k, v in data.items() if k.lower() not in SENSITIVE}
    if isinstance(data, list):
        return [sanitise(item) for item in data]
    return data
```

### 8.5 Rate Limiting

```python
from collections import defaultdict
import time

rl = RateLimiter(max_calls=60, window=60)
if not rl.check(client_id):
    raise PermissionError("Rate limit exceeded")
```

### 8.6 Audit Logging

```python
import logging, json
logger = logging.getLogger("mcp.audit")

@mcp.tool()
def sensitive_operation(data: dict) -> dict:
    logger.info(json.dumps({"event":"tool_call", "tool":"sensitive_operation", "params": data, "ts": time.time()}))
    return perform_operation(data)
```

### 8.7 MCP vs Direct API Security

| Aspect | MCP | Direct API |
|---|---|---|
| Attack surface | Narrow (protocol-defined) | Broad (any endpoint) |
| Security boundary | Server + client = 2 layers | Single layer |
| Auth changes | Centralised at server/gateway | Per-endpoint or per-service |
| Audit capability | Built-in (tool tracing) | Requires custom middleware |
| Least privilege | Per-tool permissions | Per-endpoint (if implemented) |

---

## 9. MCP for Enterprise and Banking

### 9.1 Enterprise Use Cases

**Secure Internal Data Access:** Connect LLMs to internal DBs without direct credentials. Filter PII at MCP server level.
**Controlled API Access:** MCP server wraps banking APIs with rate limits, auth, and audit logging. LLM never touches API directly.
**Compliance-Aware Execution:** Every tool call logged with caller identity, inputs, timestamp, result. Satisfies MAS, GDPR, SOX, PCI DSS.
**Data Residency:** MCP servers control data leaving the jurisdiction; local servers keep data on-premise.

### 9.2 Enterprise Architecture

```
LLM App → MCP Gateway (Auth/Routing/Logging)
  → Risk MCP (Credit risk)
  → Compliance MCP (Regulatory reporting)
  → Customer MCP (KYC, profile)
```

### 9.3 Deployment Patterns

| Pattern | Description | Best For |
|---|---|---|
| Internal network | MCP servers on LAN, stdio or internal SSE | Internal tools, sensitive data |
| DMZ deployment | Authenticated SSE in DMZ | External-facing tools, partners |
| Service mesh | Gateway with Istio/Linkerd, mTLS | Large enterprises, microservices |
| Air-gapped | Fully offline MCP servers, local LLM | Classified, strict data sovereignty |

### 9.4 Compliance Requirements

| Requirement | MCP Implementation |
|---|---|---|
| **Audit trail** | Log all tool calls (who, what, when, result) to immutable store |
| **Data residency** | Deploy MCP servers in-region; filter cross-border data at server level |
| **Access control** | Integrate with IAM (LDAP, SAML, OIDC, Azure AD, Okta) |
| **Least privilege** | Per-tool, per-resource RBAC at MCP server/gateway |
| **Encryption** | TLS 1.3 in transit; server-configurable encryption at rest |
| **Regulatory reporting** | Export audit logs to Syslog, Splunk, ELK |

### 9.5 Banking Use Case: KYC Service MCP Server

- `check_customer_status(id)` → risk tier, verification level
- `validate_document(id, type)` → authenticity score
- `search_sanctions_list(name, country)` → match/no-match

Security: OIDC + SSO. Read-only DB. All calls logged for audit.

### 9.6 Banking Use Case: Trade Surveillance MCP Server

- `check_trade_restrictions(instrument, counterparty, jurisdiction)` → allow/block
- `get_market_risk_data(instrument_id, range)` → VaR, Greeks
- `report_suspicious_activity(trade_id, reason)` → case ref

Security: Role-based access (trader vs compliance). Rate-limited. Outputs sanitised.

---

## 10. MCP Future Directions

### 10.1 Protocol Evolution

| Area | Current | Future |
|---|---|---|
| Context | Static per session | Deep multi-turn, session persistence |
| Streaming | Single-response calls | Streaming results, progress updates |
| Bidirectional | Client→Server invoke | Full duplex, server-initiated recommendations |
| Security | Basic auth + OAuth 2.0 | ABAC, zero-trust, signed tool requests |
| Lifecycle | Connect→Use→Disconnect | Long-lived sessions, hot-reload |
| Subscriptions | Polling | Real-time push via SSE/WS |
| Federation | Independent servers | Cross-server resource refs, server mesh |

### 10.2 Ecosystem Growth

- **Standardised marketplaces** — Official MCP registries with quality ratings and security audits.
- **Enterprise MCP platforms** — Managed gateway services with auth, monitoring, SLAs.
- **MCP-compliant SaaS** — SaaS providers ship MCP servers alongside REST APIs.
- **IoT/edge MCP** — Lightweight implementations for embedded devices.

### 10.3 MCP and Agents

MCP is becoming the standard protocol for agent-to-tool communication:

- **Single-agent tool use** — Agent calls MCP tools to accomplish tasks (current model).
- **Multi-agent coordination** — Agents discover and invoke tools exposed by other agents via MCP.
- **Agent-internal MCP** — Sub-agents use MCP to share tools and context.
- **Swarm architectures** — MCP as communication fabric for agent swarms (AutoGPT, CrewAI, Magentic-One).

### 10.4 MCP and Observability

- **Distributed tracing** — MCP tool invocations traced end-to-end (OpenTelemetry).
- **Metrics** — Tool call latency, error rates, throughput, saturation.
- **Performance monitoring** — Per-tool dashboards, anomaly detection.
- **Cost tracking** — Attributing LLM spend to specific tool calls and users.

### 10.5 MCP and Governance

- **Policy-as-code** — MCP gateways enforce organisation policies on tool access, data flow, rate limits.
- **Compliance frameworks** — Standardised MCP packages (SOC 2, ISO 27001, PCI DSS, MAS TRM).
- **Audit standards** — Canonical audit format for MCP tool invocations, SIEM-exportable.
- **Federated governance** — Cross-organisation MCP with contractual trust boundaries.

---

## 11. Getting Started Checklist

### □ Read the MCP Specification
- [modelcontextprotocol.io](https://modelcontextprotocol.io) — Official docs and spec

### □ Install an MCP SDK
```bash
pip install fastmcp       # Python
npm install @modelcontextprotocol/sdk  # Node.js
```

### □ Build a Simple MCP Server
1. **Filesystem** — Wrap `os`/`pathlib` as MCP tools
2. **API wrapper** — Call OpenWeather, GitHub REST via MCP
3. **Database** — Wrap SQLite/PostgreSQL queries
4. **Calculator** — Expose math operations

### □ Configure Claude Desktop
Edit `claude_desktop_config.json`:
```json
{"mcpServers": {"my-server": {"command": "python", "args": ["-m", "my_server"]}}}
```
(Mac: `~/Library/Application Support/Claude/`, Win: `%APPDATA%\Claude\`)

### □ Explore Community Servers
- [Smithery.ai](https://smithery.ai) — Registry and hosting
- [Glama.ai](https://glama.ai) — Marketplace
- [GitHub: modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)

### □ Test with MCP Inspector
```bash
npx @modelcontextprotocol/inspector python -m my_mcp_server
```

### □ Integrate MCP into Your Application
- **Python:** `mcp` client library
- **TypeScript:** `@modelcontextprotocol/sdk/client`
- **CLI:** Claude Code `.mcp.json`

### □ Consider MCP for Enterprise
- Evaluate gateways (APISIX MCP plugin, custom). Integrate with IAM. Plan audit logging.
- Start with one use case (secure DB access) and expand.

### □ Monitor the Ecosystem
- [modelcontextprotocol on GitHub](https://github.com/modelcontextprotocol)
- [Spec changelog](https://modelcontextprotocol.io/specification/latest)

---

## 12. References and Resources

### Official
- **Spec:** https://modelcontextprotocol.io
- **GitHub:** https://github.com/modelcontextprotocol/modelcontextprotocol
- **Servers:** https://github.com/modelcontextprotocol/servers
- **Python SDK:** https://pypi.org/project/fastmcp/
- **TypeScript SDK:** https://www.npmjs.com/package/@modelcontextprotocol/sdk

### Community & Platforms
- **Smithery.ai** (registry), **Glama.ai** (marketplace), **Composio** (250+ integs)
- **Cloudflare MCP:** https://developers.cloudflare.com/mcp/
- **Nango (OAuth):** https://nango.dev
- **MCPHub CLI:** https://github.com/JePlank/mcphub

### Learning
- **Announcement:** https://www.anthropic.com/news/model-context-protocol
- **Wikipedia:** https://en.wikipedia.org/wiki/Model_Context_Protocol
- **Enterprise Guide 2025:** https://guptadeepak.com/research/mcp-enterprise-guide-2025/

### Tools
- **Inspector:** `npx @modelcontextprotocol/inspector`
- **Pipedream:** https://pipedream.com/mcp

---

> **About this guide:** Written July 2026. MCP is under active development. For the latest specification, refer to [modelcontextprotocol.io](https://modelcontextprotocol.io).
>
> **Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB
