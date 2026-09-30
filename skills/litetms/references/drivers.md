# Driver tools reference

Both tools belong to the LiteTMS MCP server at `https://mcp.litetms.eu/mcp`.

| Property | Value |
|----------|-------|
| OAuth scope | `drivers:read` |
| LiteTMS permission the user needs | viewing employees (`employees.view`) |
| Effect | read-only: `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: false` |
| Visibility | only people in the signed-in user's branches |
| Result | `structuredContent` with the object below, plus one text block holding the same JSON |
| Error | `isError: true` with one plain sentence |

## search_drivers

Title: **Search drivers**

Search the company's drivers and other staff kept in the LiteTMS Employees tab by name. Returns a
short list with each person's status, position and currently selected vehicle. Use `get_driver` with
an id from this list for contact details and documents.

### Input

No other properties are accepted.

| Property | Type | Rule |
|----------|------|------|
| `query` | string, optional | Trimmed. When given, 2 to 100 characters. Matches first, middle and last name, in any letter case. |
| `status` | string, optional | One of `active`, `terminated`, `archived`, `all`. Default `active`. |
| `position` | string, optional | At most 100 characters. The exact stored position code, for example `driver`. |
| `page` | integer, optional | 1 or more. Default 1. |
| `page_size` | integer, optional | 1 to 25. Default 10. |

### Output

```json
{
  "drivers": [
    {
      "id": "uuid",
      "name": "Jan Kowalski",
      "status": "active",
      "position": "driver",
      "drives_orders": true,
      "vehicle": "WX 12345",
      "trailer": "WX 9876P",
      "app_status": "online",
      "url": "https://acme.litetms.eu/en/employees/{id}"
    }
  ],
  "total": 32,
  "page": 1,
  "page_size": 10,
  "has_more": true
}
```

| Field | Meaning |
|-------|---------|
| `id` | The person's id. Pass it to `get_driver` as `driver_id`. |
| `name` | First and last name. |
| `status` | The person's employment status, such as `active`. |
| `position` | The stored position code, or `null`. |
| `drives_orders` | `true` when the person is assigned as the driver on a transport order. |
| `vehicle` | Registration of the vehicle the person selected most recently, or `null`. |
| `trailer` | Registration of the trailer the person selected most recently, or `null`. |
| `app_status` | Activity of the person's LiteTMS Mobile app: `online`, `recent`, `away` or `offline`. |
| `url` | Opens the record in LiteTMS. |
| `total` | Number of people matching the search. |
| `page`, `page_size` | The page returned and its size. |
| `has_more` | `true` when another page exists. |

The list is ordered by last name, then first name, then id.

## get_driver

Title: **Get driver details**

Get one driver's or employee's details from LiteTMS: status, position, languages, ADR qualification,
branches, phone numbers, e-mail addresses, documents with expiry dates, and the vehicle and trailer
they currently use.

### Input

No other properties are accepted.

| Property | Type | Rule |
|----------|------|------|
| `driver_id` | string (uuid), required | An `id` returned by `search_drivers`. |

### Output

```json
{
  "id": "uuid",
  "name": "Jan Kowalski",
  "first_name": "Jan",
  "last_name": "Kowalski",
  "status": "active",
  "position": "driver",
  "employment_start_date": "2024-03-01",
  "termination_date": null,
  "languages": ["pl", "en"],
  "adr": {"approved": true, "classes": ["2", "3"]},
  "branches": ["Warsaw"],
  "phones": [{"number": "+48 600 100 200", "label": "work", "primary": true}],
  "emails": [{"email": "jan@example.com", "primary": true}],
  "documents": [
    {
      "type": "driving_license",
      "name": "Driving licence",
      "expires_on": "2027-05-01",
      "days_to_expiry": 213,
      "status": "valid"
    }
  ],
  "vehicle": "WX 12345",
  "trailer": null,
  "app_status": "offline",
  "url": "https://acme.litetms.eu/en/employees/{id}"
}
```

| Field | Meaning |
|-------|---------|
| `employment_start_date`, `termination_date` | ISO 8601 dates (`YYYY-MM-DD`) or `null`. |
| `languages` | Language codes the person speaks. |
| `adr` | Dangerous goods qualification: `approved` and the list of ADR `classes`. |
| `branches` | Names of the company branches the person belongs to. |
| `phones` | Each with `number`, `label` and `primary`. |
| `emails` | Each with `email` and `primary`. |
| `documents` | Each with `type`, `name`, `expires_on`, `days_to_expiry` and `status`. |
| `vehicle`, `trailer`, `app_status`, `url` | As in `search_drivers`. |

### Document status

| `status` | Rule |
|----------|------|
| `expired` | `expires_on` is before today. |
| `expiring` | `expires_on` falls within the warning period set for that document (30 days when none is set). |
| `valid` | `expires_on` is later than that. |
| `unknown` | The document has no expiry date. |

`name` is the label the Employees page shows for that document, in English.

## Errors

| Message | When |
|---------|------|
| `Driver not found.` | The id is malformed or unknown, the record was deleted, or the person is outside the user's branches. |
| A sentence about the connection or a module being off | An administrator turned the connector module off in LiteTMS, or the company account is not active. |
| A sentence about a missing permission | The user's role does not include viewing employees. |
| A sentence with a wait in seconds | More than 60 tool calls were made within 60 seconds on this connection. |

## Never returned

Neither tool returns: date or place of birth, gender, citizenship, nationality, addresses, bank
accounts, cards, salary, cooperation terms, notes, feedback, custom fields, document numbers, file
attachments, avatars, mobile login codes, GPS positions or order data.
