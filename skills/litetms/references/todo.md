# To-do tools reference

All five tools belong to the LiteTMS MCP server at `https://mcp.litetms.eu/mcp`. They reach the
signed-in user's own private to-do list: what they see under **To-Do List** in LiteTMS and in the
**To-do** tab of LiteTMS Mobile. Nobody else, administrators included, sees that list.

They come with the **To-Do List** module: the server lists them only while the user's company has
that module turned on. When they are missing, `get_navigation` shows the module's status and link.

| Tool | OAuth scope | LiteTMS permission | Hints |
|------|-------------|--------------------|-------|
| `todo_list_tasks` | `todo:read` | none (the list is the user's own) | `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true` |
| `todo_create_task` | `todo:write` | none | `readOnlyHint: false`, `destructiveHint: false`, `idempotentHint: false` |
| `todo_update_task` | `todo:write` | none | `readOnlyHint: false`, `destructiveHint: true`, `idempotentHint: false` |
| `todo_complete_task` | `todo:write` | none | `readOnlyHint: false`, `destructiveHint: true`, `idempotentHint: false` |
| `todo_delete_task` | `todo:write` | none | `readOnlyHint: false`, `destructiveHint: true`, `idempotentHint: false` |

Every tool has `openWorldHint: false`. A result is `structuredContent` plus one text block holding
the same JSON; an error is `isError: true` with one plain sentence.

## One task

| Field | Meaning |
|-------|---------|
| `id` | The task's id. Pass it as `task_id`. |
| `title` | What has to be done. |
| `notes` | Notes, or `null`. |
| `list_id`, `list_name` | The user's list the task is in; `null` is the Inbox. |
| `due_date` | The due day in the user's timezone, `YYYY-MM-DD`, or `null`. |
| `due_time` | Local due time `HH:MM`, or `null` for the whole day. |
| `when` | The group on the to-do page: `overdue`, `today`, `tomorrow`, `week` (next 7 days), `later`, `no_date` or `done`. |
| `reminder` | `{preset, date, time, push, email, sms, sent}`. `preset` is `none` when there is no reminder; `date` and `time` are the local moment it rings; `push`, `email`, `sms` are the channels; `sent` is `true` once it went out. |
| `repeat` | `none`, `daily`, `weekdays`, `weekly`, `monthly` or `yearly`. |
| `priority` | `none`, `low`, `medium` or `high`. |
| `color` | The colour label `#rrggbb`, or `null`. |
| `steps` | The checklist: `[{title, done}]`. |
| `completed`, `completed_at` | Whether it is ticked off, and when (ISO 8601, UTC). |
| `url` | Opens the task in LiteTMS. |

Never returned: the owner's id, step ids, internal timestamps and the link to a repeat's next
occurrence.

## todo_list_tasks

Title: **List my to-do tasks**

| Property | Type | Rule |
|----------|------|------|
| `view` | string, optional | `all` (default): every open task, overdue and soonest first, no due day last. `today`: open tasks overdue or due today. `upcoming`: open tasks due after today. `completed`: ticked off, most recent first (up to 200). |
| `list_id` | string, optional | A list's `id`, or `inbox` for the tasks in no list. |
| `query` | string, optional | Trimmed. When given, 2 to 100 characters, matched in the title or the notes. |
| `page` | integer, optional | 1 to 1000. Default 1. |
| `page_size` | integer, optional | 1 to 50. Default 20. |

Result: `tasks`, `total`, `page`, `page_size`, `has_more`, `today` (the user's own date), `counts`
(`overdue`, `today`, `upcoming`, `open`, `completed`) and `lists` (`id`, `name`, `open_tasks`).

## todo_create_task

Title: **Add a to-do task**

| Property | Type | Rule |
|----------|------|------|
| `title` | string, required | 1 to 300 characters after trimming. |
| `notes` | string or null | Up to 10,000 characters. |
| `due_date` | string or null | `YYYY-MM-DD`, from 2000-01-01 to 2100-12-31. |
| `due_time` | string or null | `HH:MM`, 24-hour. Needs a `due_date`. |
| `reminder` | string | `none`, `at_due`, `10m`, `1h`, `1d` (these need a due time), `day_morning`, `prev_day_morning` (09:00 on the due day or the day before; need a due day), `custom` (with `reminder_date` and `reminder_time`). It must be in the future. |
| `reminder_date`, `reminder_time` | string | Only with `reminder: "custom"`. |
| `remind_push` | boolean | Push to LiteTMS Mobile or a browser with notifications on. Default `true`. |
| `remind_email`, `remind_sms` | boolean | Default `false`. Each e-mail or SMS counts towards the company's usage; SMS needs a phone number in the user's profile. Setting one is limited to ten an hour per user. |
| `repeat` | string | `none` (default), `daily`, `weekdays`, `weekly`, `monthly`, `yearly`. Needs a `due_date`. |
| `priority` | string | `none` (default), `low`, `medium`, `high`. |
| `list_id` | string or null | A list's `id`; `inbox` or `null` is the Inbox (default). |
| `color` | string or null | `#rrggbb`. |
| `steps` | list | Up to 30 `{title, done}` objects, title up to 200 characters. |

The in-app notification is sent for every reminder, whatever the channels.

## todo_update_task

Title: **Change a to-do task**

`task_id` (string, required) plus at least one `todo_create_task` property. Only what is sent
changes.

- `due_date: null` removes the due day and its time, and stops a repeat.
- `due_time: null` keeps the day, without a time.
- A reminder set relative to the due moment moves with a new due day; a custom one stays.
- `list_id: "inbox"` (or `null`) moves the task to the Inbox.
- `steps` replaces every step: send the whole checklist, with `done` for each.
- `reminder: "none"` removes the reminder.

## todo_complete_task

Title: **Tick off a to-do task**

`task_id` (string, required) and `completed` (boolean, default `true`; `false` marks it not done
again). Result: `task`, `next_task` (the next occurrence a repeating task got, or `null`) and
`next_task_removed` (`true` when marking it not done again took that occurrence back).

## todo_delete_task

Title: **Delete a to-do task**

`task_id` (string, required). Result: `{deleted: true, id, title}`. The task disappears from the list
on the web and in LiteTMS Mobile, and its reminder is not sent.

## Errors the server answers with

| Sentence | Meaning |
|----------|---------|
| `Task not found.` | Not one of the user's tasks, deleted, or not a valid id. |
| `That moment has already passed. Pick a time in the future.` | The reminder would ring in the past. |
| `This reminder needs a due date (and time). Set it first or pick a custom reminder.` | A preset needs a due day or time the task does not have. |
| `A repeating task needs a due date.` | `repeat` other than `none` without a `due_date`. |
| `Pick a day before you set a time.` | `due_time` without a `due_date`. |
| `This list no longer exists. Pick another one.` | `list_id` is not one of the user's lists. |
| `Add your phone number in your profile to get reminders by SMS.` | `remind_sms: true` without a phone number in the user's profile. |
| `Too many e-mail or SMS reminders were set from AI apps in the last hour. Try again later, or set this reminder in LiteTMS.` | Ten writes an hour, across all of the user's apps, may turn on an e-mail or SMS reminder or move the reminder of a task that has one. Nothing was saved: say so and offer the task's link. |
| A sentence naming a property | The value of that property is not accepted; the sentence says what is. |
