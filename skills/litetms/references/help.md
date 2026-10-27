# Help tools reference

Both tools belong to the LiteTMS MCP server at `https://mcp.litetms.eu/mcp`. They give the AI app
what the assistant inside LiteTMS knows about the product: the help articles, and the pages and
modules this user can open. Every signed-in user may call them.

| Tool | OAuth scope | LiteTMS permission | Hints |
|------|-------------|--------------------|-------|
| `search_help` | `help:read` | none | `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true` |
| `get_navigation` | `help:read` | none | `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true` |

## search_help

Title: **Search LiteTMS help**

| Property | Type | Rule |
|----------|------|------|
| `query` | string, required | 2 to 200 characters: the question in the user's own words and language. |
| `query_en` | string, optional | 2 to 200 characters: the same question in English. The articles are in English and Polish, so a question in any other language is matched through this one. |
| `limit` | integer, optional | 1 to 3. Default 3. |

Result: `results`, each with `title`, `excerpt` (the part of the article that matched, about 600
characters at most), `language` (`en` or `pl`, the language of the title and excerpt) and `links`
(`[{label, url}]`: the exact pages and tabs the article names, already in the user's interface
language). An empty list means the topic is not documented.

## get_navigation

Title: **Pages and modules I can open**

No properties. Result:

| Field | Meaning |
|-------|---------|
| `pages` | `[{name, url, area, description}]`: every page of the side menu (`area: "main"`) and of the administration panel (`area: "admin"`) the user may open, with the name shown on screen in their language and what the page holds. |
| `modules` | `[{name, status, url, description, can_open}]`: every add-on module of the Module Marketplace. `status` is `active`, `trial`, `ending` (on until the paid period ends) or `not_active`. `url` opens the module when `can_open` is `true`, else its Marketplace page for a user who can turn modules on, else `null`. |
| `can_turn_on_modules` | Whether this user may turn modules on. Otherwise an administrator of the company does. |
| `marketplace_url` | The Module Marketplace, or `null` for a user who cannot turn modules on. |
| `mobile_app` | `google_play_url`, `app_store_url` and `company_address`: the address LiteTMS Mobile asks for at sign-in. |

Prices, free trials and subscription dates are not part of the answer: they are on the module's
Marketplace page.
