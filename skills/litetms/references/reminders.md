# Reminder tools reference

All four tools belong to the LiteTMS MCP server at `https://mcp.litetms.eu/mcp`. They reach the
reminders the signed-in user created themselves: the **My reminders** tab of LiteTMS. Reminders other
people shared with the user, and the automatic **Deadlines & alerts** (document expiries, renewals),
are not part of them.

| Tool | OAuth scope | LiteTMS permission | Hints |
|------|-------------|--------------------|-------|
| `list_reminders` | `reminders:read` | `reminders.view` | `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true` |
| `create_reminder` | `reminders:write` | `reminders.create` | `readOnlyHint: false`, `destructiveHint: false`, `idempotentHint: false` |
| `update_reminder` | `reminders:write` | `reminders.edit` | `readOnlyHint: false`, `destructiveHint: true`, `idempotentHint: false` |
| `delete_reminder` | `reminders:write` | `reminders.delete` | `readOnlyHint: false`, `destructiveHint: true`, `idempotentHint: false` |

Every tool has `openWorldHint: false`. A result is `structuredContent` plus one text block holding
the same JSON; an error is `isError: true` with one plain sentence.

## One reminder

`create_reminder` and `update_reminder` return one reminder; `list_reminders` returns a list of them.

| Field | Meaning |
|-------|---------|
| `id` | The reminder's id. Pass it to `update_reminder` or `delete_reminder` as `reminder_id`. |
| `title` | What to be reminded of. |
| `description` | Details, or `null`. |
| `date` | The day, `YYYY-MM-DD`. |
| `time` | Local time `HH:MM` as entered in LiteTMS, or `null` for an all-day reminder. |
| `all_day` | `true` when there is no time. |
| `remind_days_before` | How many days before `date` the reminder starts showing on the user's dashboard. `0` means on the day. |
| `priority` | `low`, `medium` or `high`. |
| `visibility` | `personal` (only the user), `coworkers` (the user and chosen colleagues), `role` (the user and everyone with chosen roles) or `company` (everyone). |
| `days_until` | Days from the user's today to `date`: `0` is today, negative once it has passed. |
| `url` | Opens the reminder in LiteTMS. |

Never returned: who else it is shared with (people or roles), who created it, timestamps, and whether
it was dismissed.

## list_reminders

Title: **List my reminders**

| Property | Type | Rule |
|----------|------|------|
| `status` | string, optional | `upcoming` (default): dated today or later, soonest first. `past`: dated before today, most recent first. `all`: everything, soonest first. |
| `from` | string, optional | `YYYY-MM-DD`. Only reminders on or after this date. |
| `to` | string, optional | `YYYY-MM-DD`. Only reminders on or before this date. Not before `from`. |
| `query` | string, optional | Trimmed. When given, 2 to 100 characters. Matches the title or the description, literally (`%` and `_` are ordinary characters). |
| `page` | integer, optional | 1 to 1000. Default 1. |
| `page_size` | integer, optional | 1 to 50. Default 20. |

Result: `reminders` (list), `total` (across all pages), `page`, `page_size`, `has_more`, and `today`:
the user's own date in their LiteTMS timezone. Use `today` to place "tomorrow", "next week" or
"overdue"; do not rely on another clock.

## create_reminder

Title: **Create a reminder**

| Property | Type | Rule |
|----------|------|------|
| `title` | string, required | 1 to 255 characters after trimming. |
| `date` | string, required | `YYYY-MM-DD`, a real calendar date from 2000-01-01 to 2100-12-31. |
| `time` | string or null, optional | `HH:MM`, 24-hour (`09:30`, not `9:30`). Leave out or `null` for all day. |
| `description` | string or null, optional | Up to 5000 characters. |
| `remind_days_before` | integer, optional | 0 to 365. Default 0. |
| `priority` | string, optional | `low`, `medium` (default) or `high`. |
| `visibility` | string, optional | `personal` (default) or `company`. `company` shows the reminder to everyone in the company and sends each of them a notification in LiteTMS. |

Sharing with chosen colleagues or roles is not available through the tools; the user does it in
LiteTMS on the reminder's page.

## update_reminder

Title: **Change a reminder**

`reminder_id` (string, required) plus at least one of the `create_reminder` properties. Only the
properties sent change; the others keep their values.

- `time: null` makes the reminder all day; a time makes an all-day reminder a timed one.
- `description: null` or `""` removes the details.
- `visibility` can be set to `personal` or `company`. When it is not sent, a reminder shared with
  chosen colleagues or roles keeps that sharing.

Returns the reminder as saved.

## delete_reminder

Title: **Delete a reminder**

`reminder_id` (string, required). The reminder disappears from the user's reminders, calendar and
dashboard, and from everyone it was shared with. There is no undo through the tools.

Returns `{"deleted": true, "id": "…", "title": "…"}`.

## Errors

| Message | When |
|---------|------|
| `Reminder not found.` | The id is malformed or unknown, the reminder was deleted, or another person created it. |
| `title is required.`, `date is required.`, `reminder_id is required.` | A required property is missing. |
| `date must be a calendar date in the form YYYY-MM-DD, from 2000-01-01 to 2100-12-31.` | `date`, `from` or `to` (named in the sentence) is not a usable date. |
| `time must be HH:MM in 24-hour form, for example 09:30, or null for an all-day reminder.` | The time has another shape. |
| `visibility can be personal or company here. Sharing with chosen colleagues or roles is set in LiteTMS.` | `coworkers` or `role` was sent. |
| `Send at least one field to change: …` | `update_reminder` received only an id. |
| `Unknown property: … Allowed: …` | A property the tool does not have. |
| `You do not have permission to do this in LiteTMS. An administrator can change your role.` | The user's role lacks the reminder permission this tool needs. |
| `Too many requests. Try again in N seconds.` | More than 60 tool calls within 60 seconds on this connection. |

The module, account and availability errors are the same for every tool: see
[drivers.md](drivers.md#errors).

## What LiteTMS records

Each change writes the same audit entry the reminders page writes (`reminder.created`,
`reminder.updated`, `reminder.deleted`), marked as made through a connected AI app, with the user's
name.
