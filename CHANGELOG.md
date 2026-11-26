# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Egleze MCP server. The 5 captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search_moments`, `get_latest`, `get_episode_summary`, `list_categories`, `list_shows`. Each tool carries its captured `title` key; no tool carries a `required` key, exactly as captured.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `momentfind error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `MOMENTFIND_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
