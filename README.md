# recompetewatch

Which federal contracts are about to expire, and who is most likely to win them when they are recompeted? recompetewatch is a local forwarding client for the hosted federal recompete-prediction MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The predictions are probabilistic estimates computed from public USAspending data, and they arrive with the base rates used, the sample size, and an honest confidence band. That is market intelligence about companies and contract lanes — not insider information, not a verdict about any person, and not a gate on anything — and this client treats it that way: it forwards the question and passes the answer through untouched.

All 3 captured tools are built; there are no omissions.

## The tools

- **`predict_recompete`** — A winner prediction for one expiring federal contract recompete. Identify the contract either by its USAspending generated internal id (`gid`) or by a `naics` plus `incumbent` pair, optionally sharpened with `agency`; `topN` sets how many competitors come back (default 5). The result gives the incumbent with an adjusted incumbent-advantage probability, the top likely competitors ranked by recent award velocity in the lane, the base rates behind the estimate, the sample size, and a confidence band. No parameter is marked required in the captured surface — the identifier alternatives live in the parameter descriptions.
- **`expiring_recompetes`** — The recompete pipeline feed: federal contracts whose period of performance ends inside a horizon window, each with a winner prediction. The window runs from `minDays` (default 90) to `maxDays` (default 180); results can be filtered by `naics` and `agency`, floored by `minAmount` (default 1,000,000), and capped with `limit` (default 20). No required parameters.
- **`lane_competitors`** — The firms with the most recent award velocity in a NAICS lane, optionally narrowed to one `agency`: in other words, the likely bidders on any recompete in that lane, ranked by recent wins and dollars. `naics` is required; `limit` defaults to 15.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/recompetewatch mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "recompetewatch": {
      "command": "npx",
      "args": ["-y", "@reneturcios/recompetewatch", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/recompetewatch
docker run --rm ghcr.io/cloudcorprecords/recompetewatch expiring_recompetes '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
recompetewatch predict_recompete '{"naics":"541512","incumbent":"Example Systems Inc"}'
recompetewatch expiring_recompetes '{"minDays":60,"maxDays":120,"limit":10}'
recompetewatch lane_competitors '{"naics":"541512"}'
recompetewatch --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `GOVCONRECOMPETE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://recompete.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `recompetewatch error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 3 tools, deep-equal, in order, with no titles invented and no `required` key added to the two tools that carry none), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
