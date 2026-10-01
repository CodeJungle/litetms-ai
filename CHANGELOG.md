# Changelog

## Unreleased

- Docs: read-only is the state today, not the plan. Tools that make changes are added later, step by
  step; the skill tells the model to follow such a tool's description and confirm with the user first.
- Docs: LiteTMS turns on every AI app with one module, "ChatGPT, Claude & MCP", instead of three
  (Claude, ChatGPT, MCP Server). README, skill and plugin description say so.

## 1.0.0 (2026-09-30)

First release.

- The LiteTMS MCP server address (`https://mcp.litetms.eu/mcp`) for Claude, Claude Code, ChatGPT,
  Codex and other MCP apps.
- The `litetms` skill: how to search drivers and employees, present the results and answer questions
  about document expiry.
- Two read-only tools on the server: `search_drivers` and `get_driver`.
