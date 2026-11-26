# BUILD REPORT — momentfind

Clean-room rebuild of `com.egleze/mcp` v1.1.0
(remote-only MCP server at
`https://egleze-mcp-server.onrender.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/momentfind` 0.1.0, registry name
`io.github.CloudCorpRecords/momentfind`.

## Surface built

Full surface: **5 of 5 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `search_moments` | (none) |
| `get_latest` | (none) |
| `get_episode_summary` | (none) |
| `list_categories` | (none) |
| `list_shows` | (none) |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured: the capture
carries a `title` key on every tool and it is served verbatim,
and no tool carries a `required` key. The tool literals were
generated programmatically from
`harness/fixtures/egleze-tools.json` by the script at
`/tmp/b51-momentfind-scratch/gen.py` (parse + re-serialise; the
generator's round-trip assertions confirmed the emitted literal
parses back to the fixture's tool objects) — never retyped — and
the registration-integrity test suite plus the parity harness
confirmed the served literals deep-equal the fixture's tool
objects. Tools are registered via the MCP SDK low-level Server
API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 5 tool names,
their titles and descriptions, the absence of any `required`
key on every tool, and the property shapes (including
`list_categories` with empty properties and no
`additionalProperties` key, served verbatim).

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned:
  `https://egleze-mcp-server.onrender.com/mcp`.
- Env override `MOMENTFIND_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`MOMENTFIND_API_KEY`, `API_KEY`, `AUTHORIZATION`)
  set — and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `momentfind error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests.

## Deviations

**None.** (The validation chain ran setsid-detached per the
standing write-guard playbook, invoking the direct
`tsc`/`tsx` binaries where noted; all outputs were verified
from disk afterwards. That is the documented normal route,
not a deviation.)

## Lockfile

- Route: **Route 3 (offline)** succeeded:
  `npm install --package-lock-only --ignore-scripts --offline`,
  generated clean in the build dir with no `node_modules`
  present.
- Parses as JSON; `name` == `@reneturcios/momentfind`.
- Package entries: **128**.
- sha256: `08986e178af22cd45ca04f8b1cf834e4509148bb4941f5c7495a5377b92732e1`
  (re-verified unchanged after the subsequent full `npm install`).

## server.json

- Description exactly: "Search attribution-verified public podcast moments and episode summaries with canonical links." —
  **94 characters** (verified programmatically from the file).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/momentfind:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/momentfind"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/momentfind`
  and its smoke test asserts the server name `momentfind`.

## Verification (all run 2026-10-28)

1. `tsc --noEmit` (direct binary, detached chain) → exit 0.
2. `tsc` build (`npm run build`) → exit 0; real `dist/` produced
   (cli, forward, index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. Tests (`tsx --test test/*.test.ts`, fake-fetch only) →
   **53 tests, 53 pass, 0 fail** (registration-integrity suite
   deep-equals the served tools/list against the fixture's 5
   tools, including no-`required`-key exactness and the
   `title`-key assertions, plus session handling/reuse, env
   override, the no-Authorization / no-X-API-Key assertions
   under any environment, verbatim argument forwarding and
   result pass-through including structuredContent and upstream
   isError, SSE parsing, error rendering for both the
   JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including working subcommands
   for all 5 tools against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/egleze-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/momentfind/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/momentfind/parity-out` → **verdict PASS, parity 100.0**
   (5/5 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
