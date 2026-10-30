# Public REST API

Plain JSON routes under `/api/v1` that let another app add notes and tasks: an email client, a script, an automation (n8n, Home Assistant, iOS Shortcuts). It only adds: a token for it can't read, list or change any note. To read and change notes from an AI agent, use [MCP](mcp.md); to be told when something changes, use [webhooks](webhooks.md).


!!! note "Available from NoteTrace's next release"
    Earlier versions have no `/api/v1` routes and no `notes:create` scope.

## Authentication

Create a token under **Settings, API Tokens** with the **notes:create** scope, and send it as a bearer token. Creating a token asks for your password, and your two-step code when that's on (an account that only signs in through single sign-on: a sign-in in the last 15 minutes), and when it should expire: in 30 days, 90 days (the default), 1 year, or No Expiry. From the next release; see [API tokens](../auth/sessions.md#api-tokens). Every signed-in account on a server with user accounts can make one for itself; the notes it adds belong to that account. The MCP scopes stay admin-only.

```
Authorization: Bearer note_pat_...
```

Holding the scope is the opt-in: there's no server switch. A token with only `notes:create` can add notes and call `/api/v1/me`, and nothing else. An MCP token can't add through these routes unless it also holds `notes:create`.

## Routes

| Method | Path | Notes |
|---|---|---|
| GET | `/api/v1/me` | Any token: who it belongs to (`user`), this server (`instance.url`, `instance.version`) and the token's `scopes`. |
| POST | `/api/v1/notes` | `notes:create`: adds one note or task. Answers `201` with `{ note }`, the note as MCP's `get_note` gives it. |

### POST /api/v1/notes

| Field | Type | Notes |
|---|---|---|
| `title` | text | Up to 1,000 characters. |
| `text` | text | Markdown, for a text note. |
| `kind` | `text` or `checklist` | A checklist when `items` are given without `text`. |
| `items` | list | Checklist items: text, or `{ "text": ..., "due": "YYYY-MM-DD" }`. Up to 200. |
| `due` | `YYYY-MM-DD` | The due date for items that don't give their own. |
| `show_in_tasks` | true or false | A checklist shows every item in Tasks. |
| `labels` | list of names | Matched by name, ignoring case and a leading `#`; a missing label is created. Up to 20, each cut to 60 characters. |
| `color`, `pinned` | | As in the app. |
| `reminder` | object | `{ "at": "2026-11-10T09:00", "time_zone": "America/New_York", "repeat": "weekly" }`. `at` with an offset needs no `time_zone`; without either, your latest reminder's zone, else the server's. `repeat` is `daily`, `weekly`, `monthly`, `yearly` or `none`. A one-off time in the past is refused, and so is a date or time that doesn't exist (February 30, 25:00). |
| `source_url`, `source_title` | | Where it came from (`http` or `https`). A text note ends with a link to it, titled `source_title` or the address's host; a checklist has no text, so the address goes at the end of its first item, where the card's link preview shows it. It's plain Markdown, so sync, backups and export keep it. |

Everything is checked before anything is written: a refused request answers `400` with `{ error, code: "invalid", field }` and leaves nothing behind, new labels included.

## Safe retries

Send an `Idempotency-Key` header (1 to 200 visible characters, such as `mail-<message id>-1`). The same key with the same body within 24 hours answers exactly as the first time, with `Idempotent-Replayed: true`, and adds no second note. A retry within 24 hours never adds a second note, even if you deleted the first: it gets the first answer again. The same key with a different body answers `409` (`code: "idempotency_conflict"`); the order of the fields doesn't count as a difference. A refused request isn't remembered, so the key works once the body is fixed. Without the header every request adds a note.

Keys belong to your account, not to one token: two tokens of the same account that send the same key share it. After 24 hours a key is forgotten, and sending it again adds a new note.

One request is one note. To save several tasks from one email, send one checklist with an item for each, each with its own due date.

## Limits and errors

60 requests a minute per token (`API_RATE_LIMIT_PER_MIN`), with `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset` (Unix seconds) on every answer and `Retry-After` on a `429`. Errors carry a `code`: `auth_missing` and `auth_invalid` (401), `auth_scope` (403), `rate_limited` (429), `invalid` (400), `idempotency_conflict` (409), `not_found` (404). A browser page on another site can't use these routes through someone's browser (`403 cross_site`).

## Example: an email becomes a task

```sh
curl -X POST https://notes.example.com/api/v1/notes \
  -H "Authorization: Bearer note_pat_..." \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: mail-1234-task" \
  -d '{
    "title": "Invoice from Acme",
    "kind": "checklist",
    "items": [{ "text": "Pay the invoice", "due": "2026-11-10" }],
    "show_in_tasks": true,
    "labels": ["Mail"],
    "reminder": { "at": "2026-11-10T09:00", "time_zone": "America/New_York" },
    "source_url": "https://mail.example.com/?m=1234",
    "source_title": "Invoice email"
  }'
```

## Related

- [MCP](mcp.md): read and change notes from an AI agent.
- [Webhooks](webhooks.md): be told when notes change.
- [Settings reference](settings.md): API Tokens.
