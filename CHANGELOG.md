# Changelog

All notable changes to this repository are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the package uses [Semantic Versioning](https://semver.org/).

## [0.2.0] - 2026-11-01

### Added

- This repository: the PopNow developer MCP server, for coding agents and your own code with a PopNow API key.
  The consumer server for the ChatGPT and Claude apps (OAuth, website credits) stays in
  [popnow-mcp](https://github.com/popnow-ai/popnow-mcp).
- Setup guide and API key configs for Claude Code, Cursor, VS Code, Windsurf, Codex, Cline, Claude Desktop and generic clients.
- `server.json` for the MCP registry as `io.github.popnow-ai/popnow-api-mcp`, with a required `Authorization` header.

### Changed

- `@popnow/mcp` moved here from popnow-mcp. It now connects to `https://api.popnow.ai/api/v1/mcp` by default
  (it was `https://api.popnow.ai/mcp`, which accepts API keys until 31 January 2027) and its `mcpName` is
  `io.github.popnow-ai/popnow-api-mcp`.
- Billing is in USD from the API balance: every paid call returns a quote first and runs with `max_usd`.
