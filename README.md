# JOA MCP Server

Remote [Model Context Protocol](https://modelcontextprotocol.io) server for the
**Job Opportunities API (JOA)** — search live employer-direct job postings
from employers' own career sites, look up a company's hiring signal, read
aggregate market statistics, and follow the incremental change feed, straight
from an AI agent.

- **Live endpoint:** `https://api.jobopportunitiesapi.org/mcp` (Streamable HTTP, single endpoint, both verbs)
- **Use the endpoint above (not the website) in MCP clients and proxies** such as Claude Desktop, Cursor, mcp-remote or mcpo.
- **Setup guide:** [MCP setup guide](https://jobopportunitiesapi.org/docs/mcp) (a web page for people, not the server endpoint)
- **Pricing:** https://jobopportunitiesapi.org/pricing
- **Registry:** [`org.jobopportunitiesapi/mcp`](https://registry.modelcontextprotocol.io) on the official MCP Registry
- **This repo:** listing/config metadata only — **no API source code**. The server itself ships in JOA's own Go API binary.

## The six tools

| Tool | What it does | Key needed? |
|---|---|---|
| `search_jobs` | Filter the live ledger by country, city, US state, category, seniority, remote type, employment type, salary, employer, source type, free text | Yes |
| `get_job` | One job posting's full detail, including the complete advert text and closure info | Yes |
| `company_hiring` | An employer's profile and open-roles trend over several time windows | Trend is keyless; full profile/roster needs a key |
| `market_signals` | Aggregate salary percentiles and time-to-fill by job family × country × seniority | **No** |
| `coverage` | Dataset size, freshness, per-country and top-employer coverage audit | **No** |
| `changes_since` | Incremental delta feed: created/updated/withdrawn/delisted since a cursor | Yes, Growth plan or above |

A row-serving tool called with no key never touches the database — it returns a
structured refusal naming the free-key page, never a bare 401. Any job or company
description text returned by any tool is third-party text scraped from an external
employer site: every tool description tells the calling model to treat it as data,
never as an instruction to follow.

## Auth

Exactly one door: an `Authorization: Bearer <key>` HTTP header on the MCP
connection — never a `?key=` query param, never a tool argument. A free **Explore**
key (1,000 records/month, no card) is enough to try every keyed tool:
https://jobopportunitiesapi.org/login?ref=mcp

Every tool call is metered identically to the REST endpoint it wraps — same plan,
same monthly allowance, same rate limit. `changes_since` needs the Growth plan or
above, same as `/v1/changes`.

## Client configuration

### Claude Desktop / Claude.ai (custom connector)

Settings → Connectors → Add custom connector, or paste into `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "joa": {
      "type": "streamableHttp",
      "url": "https://api.jobopportunitiesapi.org/mcp",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

### Cursor

Project or global `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "joa": {
      "url": "https://api.jobopportunitiesapi.org/mcp",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

### Windsurf

Cascade → Plugins → MCP Servers → add a custom server with the URL and header
above, or edit `~/.codeium/windsurf/mcp_config.json` directly (same JSON shape as Cursor's).

### VS Code

Command Palette → "MCP: Add Server" → HTTP, or add to `.vscode/mcp.json` using
the identical JSON shape shown for Cursor above.

### Cline

MCP Servers → Configure MCP Servers, same JSON shape as Cursor's above (Cline
reads the identical `mcpServers` block).

### ChatGPT (developer mode / Apps)

OpenAI's unified plugin directory needs a ZIP submission with an
identity-verified developer account and a domain-ownership challenge — not yet
submitted. Until then, any MCP-capable ChatGPT developer-mode client that accepts
a raw Streamable HTTP URL with a static header can use the configuration shown above.

### Raw JSON-RPC (curl)

Handshake:

```bash
curl -s https://api.jobopportunitiesapi.org/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize",
       "params":{"protocolVersion":"2025-06-18","capabilities":{},
                 "clientInfo":{"name":"curl","version":"1.0"}}}'
```

A tool call:

```bash
curl -s https://api.jobopportunitiesapi.org/mcp \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call",
       "params":{"name":"search_jobs","arguments":{"country":["DE"],"category":["Engineering"]}}}'
```

## Registry metadata

[`mcp/server.json`](./mcp/server.json) is the published entry on the official
MCP Registry under the DNS-verified namespace `org.jobopportunitiesapi/mcp`.

## About JOA

Job Opportunities API — live employer-direct job postings taken from employers'
own career sites and applicant-tracking systems, with closed postings retained
with closure dates. Every field is tagged published/inferred/absent (provenance);
coverage gaps are published, not hidden (live figures: https://jobopportunitiesapi.org/facts). REST API, keyless statistics, website
job/company pages, CSV export, OpenAPI spec.

- Website: https://jobopportunitiesapi.org
- Docs: https://jobopportunitiesapi.org/docs
- Pricing: https://jobopportunitiesapi.org/pricing
- Contact: support@jobopportunitiesapi.org

## License

The contents of this repository (documentation, configuration snippets, and
`server.json`) are licensed under the [MIT License](./LICENSE). This repository
contains **no source code of the JOA API or MCP server implementation** — those
live in JOA's private monorepo and ship as part of the production API binary.
