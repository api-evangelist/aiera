---
name: aiera-mcp-agent-access
description: Connect an agent to Aiera's hosted MCP server, choosing between OAuth and the api_key query parameter, and know which capabilities exist only there.
api: Aiera MCP Server
generated: '2026-09-14'
method: generated
source: https://rest.aiera.com/docs/mcp, https://rest.aiera.com/docs/mcp-platforms, https://rest.aiera.com/docs/mcp-sdks, https://rest.aiera.com/docs/mcp-tools, mcp/aiera-mcp.yml, mcp/aiera-tool-crosswalk.yml
operations:
  - tools/list
---

# Reach Aiera from an agent

Endpoint: `https://mcp-pub.aiera.com/`

## Pick the right connection mode

Aiera supports two, and they are for different situations.

**OAuth 2.1** is what the Claude and ChatGPT connector directories use. Search "Aiera" in the
connector store, click Connect, and sign in with existing Aiera dashboard credentials. Requires an
Admin/Owner role in the host platform and an active Aiera subscription. Discovery:

- `https://mcp-pub.aiera.com/.well-known/oauth-protected-resource` — resource metadata (RFC 9728)
- `https://mcp-pub.aiera.com/.well-known/oauth-authorization-server` — authorization server metadata
  (RFC 8414), PKCE `S256`, dynamic client registration at `/oauth/register`

Scopes are `openid`, `profile`, `email` — identity only. Content permissions come from the Aiera
account's entitlements, not from scope.

**API key in the query string** is what Aiera's own SDK examples use:

- OpenAI: `tools: [{type: "mcp", server_label: "Aiera", server_url: "https://mcp-pub.aiera.com/?api_key=..."}]`
- Anthropic: `mcp_servers: [{type: "url", name: "Aiera", url: "https://mcp-pub.aiera.com/?api_key=..."}]`

Simpler, but the key is in a URL — it will be logged by anything in the path. Prefer OAuth where the
host supports it.

**Locally over stdio**, `pip install git+https://github.com/aiera-inc/aiera-mcp.git` runs the same tool
surface with `AIERA_API_KEY` in the environment. Aiera marks this package experimental and subject to
breaking changes.

## Know what only exists here

Aiera's MCP surface is not a wrapper over its published REST contract. Roughly two dozen tools have no
operation in either published OpenAPI:

- **Financial statements and metrics** — `get_financials` (income statement / balance sheet / cash
  flow, as-reported or standardized), `get_ratios`, `get_kpis_and_segments`.
- **Indexes and watchlists** — `get_available_indexes`, `get_index_constituents`,
  `get_available_watchlists`, `get_watchlist_constituents`.
- **Semantic search** — `search_transcripts`, `search_filings`, `search_company_docs`,
  `search_thirdbridge`. These are embedding-based and paginate with an opaque `search_after` cursor,
  not page numbers. There is no REST equivalent anywhere.
- **Broker research** — the whole `find_research` / `get_research` / `search_research` family plus the
  taxonomy lookups.
- **`trusted_web_search`** — web search restricted to domains relevant to financial professionals,
  narrowable further with `allowed_domains`.

If you need any of the above, MCP is the only published way in.

## Know what is missing here

Chat sessions, Transcrippet creation and deletion, people lookup, company lookup, entitlement
introspection, the iCal feeds, transcript item detail and the HTML/PDF/CSV renditions are REST-only.
So is everything in Aiera's GitHub spec family — topics, AI summaries, corporate activity, news and
monitor stream matches. See `mcp/aiera-tool-crosswalk.yml` for the full mapping.

## Things that will bite you

- **Tool schemas are gated.** An anonymous `tools/list` returns HTTP 401 with an OAuth challenge. The
  tool inventory in `mcp/aiera-mcp.yml` is transcribed from Aiera's published documentation; exact
  JSON Schema types need an authenticated introspection call.
- **`page_size` ceilings disagree.** Aiera's tool reference says default 25, max 25; the Python
  package defaults to 50 with a max of 100.
- **No write tools exist.** Every MCP tool reads. There is nothing here to be idempotent about and
  nothing to reverse — which is fortunate, because the REST API has no idempotency mechanism at all.
