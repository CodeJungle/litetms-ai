# Changelog

## 1.1.0 (2026-10-02)

Update to get the new skill (`claude plugin update litetms@litetms`; Codex and Gemini CLI update the
plugin the same way they installed it). The server address is unchanged.

- Four reminder tools on the server: `list_reminders`, `create_reminder`, `update_reminder` and
  `delete_reminder`. They reach only the reminders the signed-in user created. Everything else stays
  read-only.
- The skill has a Reminders section: turn "on Friday" into a date from the user's own `today`, find a
  reminder before changing it, carry out a clear request and report what was saved, ask first when
  more than one reminder fits, before deleting and before making a reminder company-wide.
- New `skills/litetms/references/reminders.md`: inputs, outputs, hints and the server's sentences.
- The permission sentence is now "You do not have permission to do this in LiteTMS." for every tool.
- Descriptions, the ChatGPT and Codex listing text, its capabilities and example prompts, and the
  README say what the connection does now. Anyone who connected before connects again once, so the
  Allow page can ask for the two reminder areas.

## 1.0.1 (2026-10-01)

Text only: no change to the server address or to what the plugin contains on your computer. Update
to get the new skill text (`claude plugin update litetms@litetms`).

- One module, "ChatGPT, Claude & MCP", turns on every AI app in LiteTMS instead of three (Claude,
  ChatGPT, MCP Server). README, skill and descriptions say so.
- Read-only is the state today, not the plan. Tools that make changes are added later, step by step;
  the skill tells the model to follow such a tool's description and confirm with the user first.
- When that module is off, the LiteTMS Allow page offers the way forward: an administrator turns it
  on there (**Turn on and continue**), anyone else asks the people who can (**Ask them to turn it
  on**). The page also warns when the user's role cannot see an area yet.
- The skill's error reference quotes the server's actual sentences (module off, account on hold or
  inactive, permission, rate limit, LiteTMS unavailable).
- Both plugin manifests carry the same description ("Read-only for now").
- New `SECURITY.md`: report a vulnerability privately to security@litetms.eu.
- README: a sentence to paste into any AI agent so it connects itself.
- README: one-click links that open Claude, Cursor or VS Code with LiteTMS filled in (each asks you to
  confirm), and a prefilled link for Claude Team and Enterprise Owners.
- Codex installs the plugin from this repository: `codex plugin marketplace add CodeJungle/litetms-ai`,
  then `/plugins` or `codex plugin add litetms@litetms`.
- New `gemini-extension.json`: `gemini extensions install https://github.com/CodeJungle/litetms-ai`
  adds the server and the skill to Gemini CLI.

## 1.0.0 (2026-09-30)

First release.

- The LiteTMS MCP server address (`https://mcp.litetms.eu/mcp`) for Claude, Claude Code, ChatGPT,
  Codex and other MCP apps.
- The `litetms` skill: how to search drivers and employees, present the results and answer questions
  about document expiry.
- Two read-only tools on the server: `search_drivers` and `get_driver`.
