# @pipeworx/form5500

U.S. employee-benefit and pension plan annual returns (Form 5500 / 5500-SF): who sponsors a
plan, how many participants, how large the plan is, and, for the plans that file them, the
Schedule H (large plans) and Schedule I (small plans) financial line items.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1865+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

Sources: U.S. Department of Labor, Employee Benefits Security Administration (EBSA), EFAST2
Form 5500 Search (live, per request) and the U.S. Department of Labor, EBSA, Form 5500
Schedule H/I datasets (a stored copy, refreshed from the publisher; `data_as_of` is in every
Schedule response). Federal data, public domain.

## Tools

- `form5500_search(sponsor?, ein?, plan_name?, state?, plan_year?, ...)` — find filings.
  A sponsor name that matches nothing as written is retried with legal-form words removed
  ("Corp.", "Inc", "LLC"), then word-by-word; the response says which retry answered
  (`search_fallback`) or, when none did, lists near names in `did_you_mean`.
- `form5500_filing(ack_id)` — one filing plus its Schedule H / Schedule I line items under
  `schedule_h` / `schedule_i`. `form5500_filing(ein)` — that sponsor's full filing history.
- `form5500_schedule(ein, plan_number, plan_year?)` — Schedule H / I line items with no Ack ID.

## `schedule_status` is never null

`h`, `i` (data returned) · `not_filed_sf` (a Form 5500-SF filer: files neither schedule) ·
`full_form_no_schedule` (a full Form 5500 with neither schedule) · `not_loaded_year` (that plan
year is not in the stored copy; `schedule_loaded_plan_years` lists what is) · `not_in_dataset_yet`
(filed after the dataset's as-of date) · `unknown_plan_year` · `store_unavailable`.
`form5500_schedule` adds `no_schedule_found`.

Line items use friendly names (`employer_contributions`, `total_contributions`, `total_income`,
`total_benefits_paid`, `total_expenses`, `net_assets_eoy`, ...) plus every stored line under
`dol_fields` by its DOL column name. A line the filer left blank is listed in
`blank_line_items` and absent from `line_items`; a reported 0 stays 0.

## Refresh

`bash scripts/form5500/refresh.sh build|load|verify` (see the header of that script for the
one-time D1 database creation and binding). No deploy is involved in a data refresh.

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

Both URLs reach the same gateway and the same 1865+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/form5500_search \
  -H 'Content-Type: application/json' \
  -d '{"sponsor":"Apple Inc","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/form5500_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-form5500` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/form5500/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

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
