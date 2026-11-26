# momentfind

Looking for what was actually said in a podcast, without wading through full transcripts? momentfind is a local forwarding client for the hosted Egleze MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The service indexes attribution-verified public podcast moments and published episode summaries. You get Egleze-written headlines and short summaries with canonical links back to the source — never full transcripts or quotes. Browse what is new, filter by topic category, search by keyword and date range, pull an episode summary by show or episode title, or list the categories and shows the index covers.

## The tools

- **`search_moments`** — Search moments by `query` keyword, `category`, and a `since` / `until` date range, with a `limit` (1–50, default 10). No argument is required.
- **`get_latest`** — The newest moments, optionally filtered by `category`, with a `limit` (1–50, default 10).
- **`get_episode_summary`** — A published episode summary and its key points, matched by `show` name and/or `episode` title fragment, with a `limit` (1–10, default 3).
- **`list_categories`** — The topic categories the index covers, for use as filters in the search tools. Takes no arguments.
- **`list_shows`** — The podcasts represented in the index, with moment counts, with a `limit` (1–200, default 100).

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/momentfind mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "momentfind": {
      "command": "npx",
      "args": ["-y", "@reneturcios/momentfind", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/momentfind
docker run --rm ghcr.io/cloudcorprecords/momentfind search_moments '{"query":"AI safety"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
momentfind search_moments '{"query":"AI safety","limit":5}'
momentfind get_latest '{"category":"AI & Tech"}'
momentfind get_episode_summary '{"show":"The Joe Rogan Experience"}'
momentfind list_categories '{}'
momentfind list_shows '{"limit":20}'
momentfind --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `MOMENTFIND_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://egleze-mcp-server.onrender.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `momentfind error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 5 tools, deep-equal, with their `title` keys and with no `required` key on any tool, exactly as captured), exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
