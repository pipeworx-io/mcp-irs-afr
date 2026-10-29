# @pipeworx/irs-afr

IRS Applicable Federal Rates (short, mid and long term, every compounding period, 110/120/130% variants, adjusted AFR) and the Section 7520 rate, for any month from the monthly IRS revenue ruling.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1858+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `irs_afr(month?, term?, compounding?, percent?)` — the AFR tables for a month: short-term (up to 3 years), mid-term (over 3 to 9) and long-term (over 9) at annual, semiannual, quarterly and monthly compounding, with the 110%, 120% and 130% variants (150% and 175% for mid-term) and the adjusted AFRs. Filters return a single rate. Answers "what is the November 2026 mid-term AFR, annual compounding?".
- `irs_7520_rate(month?)` — the Section 7520 rate for valuing annuities, life estates, terms of years, remainders and reversions, plus the same ruling's section 382 long-term tax-exempt rate and low-income housing credit percentages.

Omit `month` for the newest published month. The IRS issues each month's rates in the preceding month (typically mid-month), so next month's rates are usually available from about the 15th. A month that has not been issued returns `status: "not_yet_published"` with the expected timing, never an empty result.

Every response carries `ruling` (e.g. "Rev. Rul. 2026-21"), `data_as_of` (the date the IRS published the ruling) and `source`.

## Auth

Keyless.

## Data sources

- IRS, Applicable Federal Rates revenue ruling — the monthly Internal Revenue Service revenue ruling that prescribes the AFR, adjusted AFR, section 382, low-income housing credit and Section 7520 rates. US federal government work, public domain. Each request reads the ruling live; nothing is copied.

Notes for maintainers: ruling numbers are not in month order (the month is read from the ruling's own text), and the IRS listing page lags the newest ruling and mislabels months, so it is used only as a hint and later numbers are probed directly.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "irs-afr": {
      "url": "https://gateway.pipeworx.io/irs-afr/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/irs-afr/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1858+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/irs_afr \
  -H 'Content-Type: application/json' \
  -d '{"month":"2026-11","term":"mid","compounding":"annual","percent":100}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/irs_afr`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-irs-afr` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/irs-afr/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "irs-afr": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-irs-afr"]
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
ask_pipeworx({ question: "your question about Irs Afr data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
