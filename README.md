# @pipeworx/uk-flood-monitoring

England's official flood warnings and river-gauge telemetry, from the
Environment Agency's real-time flood-monitoring API.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `ukflood_warnings(min_severity?, county?, limit?)` — flood warnings and alerts
  currently in force, with severity, named flood area, river or sea, and the EA
  message text. Answers "is there a flood warning near X".
- `ukflood_stations_near(latitude, longitude, radius_km?, parameter?, river?, limit?)` —
  monitoring stations near a point and the measures each reports. This is where
  you get the station id for the next tool.
- `ukflood_station_readings(station_id, since?, limit?)` — water level (m), river
  flow (m³/s) or rainfall (mm) from one station, latest per measure by default,
  or a time series with `since`.
- `ukflood_areas(county?, search?, latitude?, longitude?, radius_km?, limit?)` —
  the named flood areas warnings are issued against, so a warning code resolves
  to a place.

## Auth

Keyless. Open Government Licence v3.

## Data sources

- <https://environment.data.gov.uk/flood-monitoring/id/floods> — warnings in force.
- <https://environment.data.gov.uk/flood-monitoring/id/stations> — station registry,
  filterable by `lat`/`long`/`dist` (km), `parameter`, `riverName`.
- <https://environment.data.gov.uk/flood-monitoring/id/stations/{id}/readings> —
  telemetry; `?latest` for the newest reading per measure, `?since=` for a series.
- <https://environment.data.gov.uk/flood-monitoring/id/floodAreas> — flood-area registry.
- Reference: <https://environment.data.gov.uk/flood-monitoring/doc/reference>

Things worth knowing:

- **England only.** Wales (Natural Resources Wales), Scotland (SEPA) and Northern
  Ireland publish separately. A Cardiff or Glasgow query legitimately returns nothing.
- **Zero warnings is a normal state, not a failure.** On a dry day `/id/floods`
  returns `items: []`, and that matched the Environment Agency's own public page
  ("No flood alerts") when this pack was built. `ukflood_warnings` therefore
  returns severity counts and an explicit all-clear `status` alongside the empty
  list, so a caller can tell "nothing is happening" from "the call broke".
- The API returns JSON-LD. `@id` fields are URIs; the pack hands back the last
  path segment, which is the notation callers actually pass back in.
- `label` is occasionally an array rather than a string.
- `floodAreas` has no text-search parameter; `search` is applied client-side
  after over-fetching, so combine it with `county` on large counties.
- Different source from the `flood` pack, which is Open-Meteo's global modelled
  river discharge. This one is measured gauges and official warnings.

## Verified

2026-09-17 — stations near London, readings from 5380TH (Walthamstow, Low Hall,
River Lee) and Lincolnshire flood areas all returned rows; warnings returned a
cross-checked genuine all-clear.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "uk-flood-monitoring": {
      "url": "https://gateway.pipeworx.io/uk-flood-monitoring/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/uk-flood-monitoring/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ukflood_warnings \
  -H 'Content-Type: application/json' \
  -d '{"min_severity":4}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ukflood_warnings`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-uk-flood-monitoring` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/uk-flood-monitoring/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "uk-flood-monitoring": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-uk-flood-monitoring"]
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
ask_pipeworx({ question: "your question about Uk Flood Monitoring data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
