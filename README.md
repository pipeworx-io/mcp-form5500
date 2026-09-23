# mcp-form5500

Form 5500 MCP — wraps the U.S. DOL/EBSA EFAST2 Form 5500 Search service

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1663+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `form5500_search` | Search U.S. Department of Labor Form 5500 / 5500-SF filings (employee benefit & pension plans, EFAST2) by plan sponsor name, employer EIN, plan name, state, and/or plan year. Returns matching filings with sponsor, plan name/number, participant counts, plan assets, plan type, and a link to the filing PDF. Provide at least one of sponsor, ein, plan_name, or state. |
| `form5500_filing` | Get Form 5500 filing detail from the DOL EFAST2 index. Pass ack_id for a single filing (sponsor, plan name/number, participant counts, begin/end-of-year plan assets, plan type, filing date, and full filing PDF URL). Pass ein instead to get a plan sponsor's full filing history (all Form 5500 filings for that EIN, newest first). Note: line-item detail such as Schedule C service providers is not in the search index — the returned pdf_url links to the complete filing. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "form5500": {
      "url": "https://gateway.pipeworx.io/form5500/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/form5500/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1663+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/form5500_search \
  -H 'Content-Type: application/json' \
  -d '{"sponsor":"Apple Inc","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/form5500_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "form5500": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-form5500"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-form5500
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Form5500 data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
