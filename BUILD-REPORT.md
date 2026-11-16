# BUILD REPORT — recompetewatch

Clean-room rebuild of `com.dropwatchhq/govcon-recompete`
v1.0.0 (remote-only MCP server at `https://recompete.dropwatchhq.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/recompetewatch` 0.1.0, registry name
`io.github.CloudCorpRecords/recompetewatch`.

## Surface built

Full surface: **all 3 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `predict_recompete` | NO `required` key in the capture — served verbatim, none invented |
| `expiring_recompetes` | NO `required` key in the capture — served verbatim, none invented |
| `lane_competitors` | `naics` |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/govconrecompete-tools-original.json` by the script at
`/tmp/b47-recompetewatch-scratch/gen_forward.py` (parse +
re-serialise into `src/forward.ts`) — never retyped — and a
post-generation check (the generator's own round-trip assertion,
the registration-integrity test suite, plus the parity harness)
confirmed the served literals deep-equal the fixture's 3 tool
objects, in order. Tools are registered via the MCP SDK low-level
Server API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 3 tools' names,
descriptions, required fields (including the two tools with no
`required` key: `predict_recompete`, `expiring_recompetes`) and
property shapes. The spec's note that the capture carries no
`title` keys matches the fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://recompete.dropwatchhq.com/mcp`.
- Env override `GOVCONRECOMPETE_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`GOVCONRECOMPETE_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `recompetewatch error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.
- Prediction judgment (from the spec, recorded by the coordinator
  on the firsthand capture read): the predictions are
  probabilistic estimates computed from PUBLIC USAspending data,
  returned WITH the base rates used, sample size and an honest
  confidence band (per the captured `predict_recompete`
  description: "Probabilistic estimate from PUBLIC USAspending
  data — not insider info"). This is the awardsignal
  predictive-market-intel family: market intelligence about
  companies and contract lanes — no person is scored, no
  exclusion list is checked, nothing is gated. No payment,
  metering, contact, signup or send tool exists in the surface.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/recompetewatch`.
- Package entries: **128**.
- sha256: `8ced31757cb6f1ba049036e89b9e191a6bd0a352b10a7265bcfbc3c959e0895b`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Federal recompete predictions from public
  USAspending data, with base rates and confidence." —
  **91 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/recompetewatch:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/recompetewatch"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/recompetewatch`
  and its smoke test asserts the server name `recompetewatch`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **56 tests, 56 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 3
   tools, including the no-title and no-required assertions, plus
   session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour including a per-tool CLI call
   against a local stub upstream).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/govconrecompete-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/recompetewatch/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/recompetewatch/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
