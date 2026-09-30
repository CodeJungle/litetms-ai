# LiteTMS for AI apps

![LiteTMS](assets/logo.png)

Connect Claude, ChatGPT, Codex or any other MCP app to your company's LiteTMS and ask about your
drivers in plain language.

[LiteTMS](https://litetms.eu) is a transport management system (TMS) for road carriers and freight
forwarders. This repository holds the LiteTMS plugin: the address of the LiteTMS MCP server and a
skill that teaches the AI app how to use it well.

## Who it is for

Dispatchers, forwarders, fleet managers and office staff of companies that already work in LiteTMS.
You sign in with your own LiteTMS account, and the AI app sees what you are allowed to see: your role
and your branches apply to every answer.

## What you can ask today

The connection has two tools. Both only read data.

| Tool | What it does |
|------|--------------|
| Search drivers | Finds drivers and other employees by name, status or position. Shows each person's status, position and the vehicle and trailer they currently use. |
| Get driver details | Shows one person's phone numbers, e-mail addresses, languages, ADR qualification, branches, employment dates and documents with their expiry dates. |

Questions that work:

- "List our active drivers and the vehicle each one uses."
- "What is Anna Nowak's phone number?"
- "When does Jan Kowalski's driving licence expire?"
- "Show me the employees who no longer work for us."
- "Which languages does Piotr Zielinski speak, and does he have ADR?"

Every answer can include a link that opens the person's record in LiteTMS.

Documents are read one person at a time. A question such as "whose documents expire this month?"
works well for a handful of people; for a large fleet the AI app will ask you to narrow it down.

Orders, vehicles, GPS positions and other parts of LiteTMS are not available yet. More tools are
added over time.

## What you need

1. A LiteTMS account that may view employees.
2. A connector module turned on for your company. An administrator does this in LiteTMS under
   Administration > Marketplace:

| Module | Turns on |
|--------|----------|
| Claude | Claude on the web, desktop and mobile, and Claude Code |
| ChatGPT | ChatGPT and Codex |
| MCP Server | Every MCP app, including the two above |

After that, each person connects their own account. Nobody shares a password or an API key.

## Connect

The server address is the same for every company:

```text
https://mcp.litetms.eu/mcp
```

If LiteTMS is listed in your AI app's directory, add it from there. Otherwise follow the steps for
your app.

### Claude (web, desktop, mobile)

1. Open Customize > Connectors.
2. Choose Add custom connector.
3. Paste the server address and choose Add.
4. Choose Connect.
5. Enter your company's LiteTMS address, sign in and choose Allow.

On Team and Enterprise plans an Owner adds the connector once under Organization settings >
Connectors. Each member then opens Customize > Connectors and chooses Connect. A connector added on
the web or desktop also works in the mobile app.

### Claude Code

Install the plugin. It adds the server and the LiteTMS skill:

```bash
claude plugin marketplace add CodeJungle/litetms-ai
claude plugin install litetms@litetms
```

Or add only the server:

```bash
claude mcp add --transport http litetms https://mcp.litetms.eu/mcp
```

Then start Claude Code, type `/mcp`, choose `litetms` and sign in.

### ChatGPT

In the ChatGPT desktop app:

1. Open Settings > MCP servers.
2. Choose Add server.
3. Enter a name, choose Streamable HTTP and paste the server address.
4. Choose Authenticate.
5. Enter your company's LiteTMS address, sign in and choose Allow.

In ChatGPT on the web, turn on Developer mode under Settings > Security and login, open the Plugins
page, choose the plus button and enter the server address.

### Codex

```bash
codex mcp add litetms --url https://mcp.litetms.eu/mcp
codex mcp login litetms
```

### Cursor, VS Code and other MCP apps

Most apps read a configuration like this one:

```json
{"mcpServers":{"litetms":{"url":"https://mcp.litetms.eu/mcp"}}}
```

VS Code uses a slightly different shape:

```json
{"servers":{"litetms":{"type":"http","url":"https://mcp.litetms.eu/mcp"}}}
```

An app that asks for a transport type needs Streamable HTTP. Sign-in uses OAuth, so there is no key
to paste.

## Signing in

The first time you connect, your browser opens `mcp.litetms.eu` and takes you through three steps:

1. **Your company's LiteTMS address.** Type the first part of the address you sign in at, for
   example `acme` for `acme.litetms.eu`.
2. **Your usual LiteTMS sign-in**, with your two-factor code if you use one.
3. **Allow or Cancel.** The page names the app and lists what it may read.

Type your password only on the LiteTMS page. The AI app never needs it.

## If something does not work

| What you see | What to do |
|--------------|------------|
| The company address is not accepted | Check the first part of the address in your browser when you are signed in to LiteTMS. |
| The Allow page says the connection is not available | Ask your administrator to turn on the module for your app under Administration > Marketplace. |
| The answer says you lack permission | Your role does not include viewing employees. Ask your administrator. |
| A person cannot be found | They may belong to a branch you do not see, or the name is spelled differently. |
| The app asks you to sign in again | The connection ended. Sign in again to restore it. |

## What this plugin contains

The plugin has no program code and runs nothing on your computer.

| File | Purpose |
|------|---------|
| `.mcp.json` | The server address for Claude and Claude Code. |
| `mcp.json` | The same address in the Agent Plugins format, used by ChatGPT and Codex. |
| `.claude-plugin/plugin.json`, `plugin.json` | Name, version and description of the plugin. |
| `.claude-plugin/marketplace.json` | Lets Claude Code install the plugin straight from this repository. |
| `skills/litetms/` | Instructions for the AI app: how to search, how to present results, what it must not guess. |

The assets folder holds the LiteTMS icon and logo. Every request the plugin causes goes to
`https://mcp.litetms.eu/mcp` and to no other address.

## Data and privacy

- **The AI app acts as you.** It reads only what your LiteTMS role and branches allow.
- **Read-only.** Nothing in LiteTMS can be added, changed or deleted through this connection.
- **What you ask for leaves LiteTMS.** The answers are sent to the AI app you connected and are
  handled under that provider's terms. Connect only an app your company accepts for staff data.
- **The plugin stores nothing.** It keeps no data, no password and no access key. Your AI app holds
  the sign-in, and you can end it at any time.
- **Some data is never sent:** home addresses, dates of birth, bank details, salary, notes, document
  numbers and attached files.
- **Your company keeps a record.** LiteTMS writes to the company's audit log that an app was
  connected, which tool was used and whose record was opened, as it does when you open an employee
  in the browser. LiteTMS receives only the search terms and ids the AI app sends, not your
  conversation.
- **Disconnecting.** In LiteTMS, open Settings > Connected apps and choose Disconnect next to the app.
  Administrators can see and end every connection of the company on the module's setup page.

How LiteTMS handles personal data: [privacy policy](https://litetms.eu/en/privacy-policy).
Terms of service: [terms](https://litetms.eu/en/terms).

## Support

Write to [contact@litetms.eu](mailto:contact@litetms.eu) or use the
[contact form](https://litetms.eu/en/contact). Tell us which AI app you use and what the error
message says. Please do not send passwords or personal data of your staff.

## License

[MIT](LICENSE). Copyright (c) 2026 CodeJungle Sp. z o.o.
