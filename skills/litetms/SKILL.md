---
name: litetms
description: "Use when the user asks about their transport company's drivers or employees in LiteTMS: who they are, how to reach them (phone, e-mail), which documents expire and when, which vehicle or trailer they drive, who is active and who no longer works for the company. Also use when the user wants to see, add, move, change or delete their own LiteTMS reminders, asks what the LiteTMS connection can do, why a person or a reminder cannot be found, or how to sign in to LiteTMS or disconnect it. Covers the search_drivers, get_driver, list_reminders, create_reminder, update_reminder and delete_reminder tools of the LiteTMS MCP server."
---

# LiteTMS

## What LiteTMS is

LiteTMS is a transport management system (TMS). Road carriers and freight forwarders use it to run
their fleet, drivers, contractors and transport orders.

The people asking are usually dispatchers, forwarders, fleet managers and office staff. They want an
answer they can act on: a number to call, a date to note, a reminder set, a link to open.

## What you are connected to

- **One company.** The connection reaches a single LiteTMS workspace: the one the user named when
  signing in.
- **One person's view.** Every tool runs as the signed-in user. Their role and their branches decide
  what comes back, so two colleagues can get different answers to the same question.
- **"Not found" can mean "not visible".** Never say a person does not exist in the company. Say you
  could not find them in the records this user can see.
- **Changes only through tools that make them.** A tool whose description says it adds, changes or
  deletes something changes LiteTMS as the user, never beyond their role; every other tool only
  reads. The reminder tools are such tools; for any tool like them, follow the rules under Reminders.

## Tools

| Tool | Use it to |
|------|-----------|
| `search_drivers` | Find people by name, status or position. Returns a short list. |
| `get_driver` | Get one person's details by the `id` taken from that list. |
| `list_reminders` | List the user's own reminders (upcoming by default) with their ids and the user's `today`. |
| `create_reminder` | Add a reminder for the user. |
| `update_reminder` | Change one of the user's reminders by its `id`: only the fields sent change. |
| `delete_reminder` | Delete one of the user's reminders by its `id`. |

Drivers are kept in the Employees tab together with other staff, so `search_drivers` also finds
office employees. The `position` field tells them apart (`driver` for drivers).

Exact inputs, outputs and the fields that are never returned: [references/drivers.md](references/drivers.md)
and [references/reminders.md](references/reminders.md). Read them when you need a field name, a filter
value or a limit.

## Workflow

1. **Search first.** Call `search_drivers` with the name the user gave as `query` (2 characters or
   more). Leave `query` out to list everyone.
2. **Mind the status.** The default is `active`. When the user asks about someone who left, or an
   active search finds nothing, search again with `status: "all"` before saying nobody matches.
3. **One match.** Call `get_driver` with its `id` when the user wants contact details, documents,
   languages, ADR, branches or employment dates. The list alone already answers who the person is,
   their status and which vehicle they use.
4. **Several matches.** Show the short list and ask which person is meant. Do not pick between two
   people with the same surname.
5. **No match.** Say so, add that the person may be outside the user's branches or permissions, and
   suggest checking the spelling.
6. **More than one page.** Results come 10 per page (25 at most). `has_more: true` means more exist.
   Say how many there are in total and offer the next page.
7. **Ids come from results only.** Never build or guess an `id`.

Questions about many people at once, such as "whose documents expire this month?", need one
`get_driver` call per person, because documents are not part of the list. For a short list, make the
calls and summarize. For a long list, say how many people there are and ask whether to go through all
of them or to narrow it down by name or position. When a tool asks you to wait before the next call,
wait and tell the user.

## Reminders

The reminder tools reach the reminders the user created (the My reminders tab of LiteTMS). Reminders
colleagues shared with them, and the automatic deadlines of documents and vehicles, are not there:
when asked about those, say so and offer `get_driver` for a person's document dates.

1. **Dates.** Turn "tomorrow", "on Friday" or "in two weeks" into `YYYY-MM-DD` from the user's own
   date. `list_reminders` returns it as `today`; call it first when you are not sure what day it is
   for the user. When a day could mean two dates ("next Friday" on a Thursday), ask.
2. **Times.** `HH:MM`, 24-hour. No time given means an all-day reminder; do not invent one.
3. **Find before you change.** To change or delete a reminder, call `list_reminders` (with `query`,
   or `from` and `to`, or `status: "all"` for an old one) and take its `id`. Never guess an id.
4. **A clear request: do it.** "Remind me on Friday at 9 to renew the insurance" needs no extra
   question. Create it, then say what was saved: title, date, time and the link.
5. **Ask first** when more than one reminder fits, when a detail is unclear, before deleting a
   reminder, and before setting `visibility: "company"`: everyone in the company then sees it and
   gets a notification. Never make a reminder company-wide unless the user asks for that.
6. **Change only what was asked.** Send `update_reminder` the fields that change and nothing else.
   Moving a reminder means a new `date` (and `time`); keep the title.
7. **After a change**, report it from the tool's answer, not from the request, and give the `url`.
   If a tool refuses, pass on its sentence: nothing was saved.

`remind_days_before` makes the reminder show on the dashboard that many days before its date. Use it
when the user asks to be reminded ahead ("two days before"); otherwise leave it out.

## Presenting results

- **A list of people:** a short table with name, position, status, vehicle and trailer. Make the name
  a link to the record's `url`.
- **One person:** a few labelled lines. Lead with what was asked.
- **Links:** every record has a `url` that opens it in LiteTMS. Offer it instead of describing where
  to click.
- **Dates:** show them as given (`YYYY-MM-DD`). Do not convert them to a format that can be misread,
  and do not work out new dates yourself.
- **Missing data:** `null` or an empty list means nothing is recorded. Say "not recorded in LiteTMS".
  Never invent a phone number, an e-mail address, a date or a vehicle.
- **Phones and e-mails:** put the one marked `primary` first and keep the phone `label`.
- **Position:** it is a stored code such as `driver`. Show it in plain words.
- **`app_status`** (`online`, `recent`, `away`, `offline`) says how recently the person's LiteTMS
  Mobile app was active. It is not a location and not proof that someone is at work.
- **`drives_orders`** says whether the person has been assigned to transport orders as a driver.

## Document expiry

Each document carries `expires_on`, `days_to_expiry` and a `status`. Use them as returned.

| `status` | Meaning | How to say it |
|----------|---------|---------------|
| `expired` | The expiry date has passed. | Expired on that date. Put these first. |
| `expiring` | The date falls inside the warning period set for that document (30 days when none is set). | Expires on that date, with the days left. |
| `valid` | The date is further away. | Valid until that date. |
| `unknown` | No expiry date is recorded. | No expiry date recorded. Do not call it valid. |

Order an answer by urgency: expired, expiring, valid, unknown. Document numbers and scanned files are
not available through the connection; give the record link for those.

## What it cannot do yet

- **No change without a tool for it.** People, documents and everything else no tool changes cannot
  be added, edited or deleted. When asked to, say so and give the record link so the user can do it
  in LiteTMS.
- **Sharing a reminder with chosen colleagues or roles** is set in LiteTMS, on the reminder's page.
  The tools can keep a reminder personal or make it company-wide.
- **Drivers and employees only.** Orders, the vehicle list, contractors, invoices, GPS positions and
  other areas are not available yet. Say that plainly. Do not answer such questions from general
  knowledge and do not guess.
- **Some personal fields are never returned:** home address, date and place of birth, bank details,
  salary, notes, document numbers and attached files. When asked, say the connection does not provide
  them.
- **More tools are added over time.** When the server offers a tool this document does not describe,
  follow that tool's own description.

## Privacy

The records describe real people.

- Show only what was asked for. A question about a phone number gets the phone number, not the whole
  record.
- Do not list the contact details of many people at once unless the user asks for exactly that.
- Do not save personal data to files or to memory, and do not pass it to other tools, unless the user
  asks.
- Use the data only for the user's request.

## Sign-in and troubleshooting

Before the tools work, the user signs in once in the browser. The sign-in has three steps:

1. **Company address.** The first part of the address they sign in at: `acme` for `acme.litetms.eu`.
2. **LiteTMS sign-in.** Their usual login, with the two-factor code if they use one.
3. **Allow or Cancel.** The page names the app and what it may read and change.

Never ask for a LiteTMS password or a sign-in code in the conversation. The user types them only on
the LiteTMS page.

Tool errors arrive as one plain sentence. Pass it on and add the next step:

| Situation | What to tell the user |
|-----------|-----------------------|
| The company address is not accepted | Check the first part of the address shown in the browser when signed in to LiteTMS. |
| The Allow page says the module is turned off, or a tool says the connection is not turned on | One module, "ChatGPT, Claude & MCP", turns on every AI app. On the Allow page an administrator chooses **Turn on and continue**; anyone else chooses **Ask them to turn it on**, which notifies the people who can, and connects again once it is on. An administrator can also turn it on in LiteTMS under Administration > Marketplace. |
| A tool says the user lacks permission | Their role does not include that area: viewing employees for people, the reminder permissions for reminders. An administrator can change that. The Allow page already warns about it before the user connects. |
| `Reminder not found.` | The reminder is not one the user created, or it was deleted. List the reminders again to find it. |
| `Driver not found.` | The person is not in the records this user can see. Search again, or check another branch in LiteTMS. |
| The app asks to sign in again | The connection ended or was disconnected. Signing in again restores it. |
| A tool asks to wait | Too many requests in a short time. Wait the stated number of seconds. |
| LiteTMS is temporarily unavailable | Try again in a moment. |

To disconnect, the user opens LiteTMS, goes to Settings > Connected apps and chooses Disconnect next
to the app.
