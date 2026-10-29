# mcp-db-rail

DB Rail MCP — German + European rail and public transport, keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

| Tool | Description |
|------|-------------|
| `dbrail_locations` | Search German / European rail stations, stops, and places by name (e.g. "Berlin Hbf", "München", "Zürich HB"). Returns matches with a stop id, coordinates, and country. Keyless. You do NOT have to call this first — dbrail_departures, dbrail_arrivals and dbrail_journeys all accept a plain station name and resolve it themselves; use this when you want to disambiguate between similarly-named stations or need coordinates. |
| `dbrail_departures` | Live departures board for a German / European station. Pass a station NAME ("Berlin Hbf") or a stop id from dbrail_locations. Returns upcoming departures with line/train number, destination, scheduled and real-time departure, delay in minutes, platform/track, and operator. Keyless, real-time where the operator publishes it. |
| `dbrail_arrivals` | Live arrivals board for a German / European station. Pass a station NAME ("München Hbf") or a stop id from dbrail_locations. Returns upcoming arrivals with line/train number, origin, scheduled and real-time arrival, delay in minutes, platform/track, and operator. Keyless, real-time where the operator publishes it. |
| `dbrail_journeys` | Plan a multi-leg rail / public-transport journey between two places in Europe. Pass station NAMES ("Berlin Hbf" -> "München Hbf") or stop ids. Returns route options, each with total duration, number of transfers, and every leg (mode, line, from/to with platforms, scheduled + real-time times). Keyless. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "db-rail": {
      "url": "https://gateway.pipeworx.io/db-rail/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/db-rail/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/dbrail_locations \
  -H 'Content-Type: application/json' \
  -d '{"query":"Berlin Hbf"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/dbrail_locations`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-db-rail` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/db-rail/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "db-rail": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-db-rail"]
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
ask_pipeworx({ question: "your question about Db Rail data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
