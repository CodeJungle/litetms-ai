# Help tools reference

The three tools belong to the LiteTMS MCP server at `https://mcp.litetms.eu/mcp`. They give the AI
app what the assistant inside LiteTMS knows about the product: the help articles, the pages and
modules this user can open, and the films LiteTMS shows on its pages. Every signed-in user may call
them.

| Tool | OAuth scope | LiteTMS permission | Hints |
|------|-------------|--------------------|-------|
| `search_help` | `help:read` | none | `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true` |
| `get_navigation` | `help:read` | none | `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true` |
| `show_video_guides` | `help:read` | none | `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true` |

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

## show_video_guides

Title: **Show LiteTMS video guides**

| Property | Type | Rule |
|----------|------|------|
| `topic` | string, optional | `^[a-z0-9-]{1,40}$`: the page or module a film is about, the last part of its address in LiteTMS (`employees`, `todo`, `reminders`; `system-tour` for the tour). Left out, every film the user can watch. |

Result: `topic` (as asked, or `null`) and `videos`, each with `topic`, `kind` (`guide`: walks through
a page; `overview`: a module in about a minute), `title` (in the user's language), `duration`
(`2:47`) and `duration_seconds`, `language` and `language_name` (the version returned: the user's
language when the film was made in it, else the film's own), `video_url` (plays in any browser, no
sign-in), `poster_url`, `page_url` (the page or module in LiteTMS, `null` when the user cannot open
it) and `chapters` (`[{title, start, start_seconds}]`, empty for an overview). A guide whose page the
user may not open is left out.

Apps that render MCP Apps (Claude, ChatGPT) show the result in a player: poster, chapters, captions,
a link to the page. Other apps get the same data as text.

| Error sentence | What to do |
|----------------|------------|
| `There is no LiteTMS video about "x". Topics with a video: …` | Pick one of the topics listed (each with its English title) and call again. |
| `There is no LiteTMS video this user can watch.` | Say so; answer from the help instead. |
| `topic must be a topic name such as employees or todo.` | Send the short name, not the page's title. |
