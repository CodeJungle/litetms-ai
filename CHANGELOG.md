# Changelog

## Unreleased

- Docs: when the "ChatGPT, Claude & MCP" module is off, the LiteTMS Allow page now offers the way
  forward: an administrator turns it on there (**Turn on and continue**), anyone else asks the people
  who can (**Ask them to turn it on**). The page also warns when the user's role cannot see an area
  yet. README and skill say so.
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
