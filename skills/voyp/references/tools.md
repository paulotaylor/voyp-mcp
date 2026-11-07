# Voyp MCP tools reference

Voyp has two MCP servers. Both have the core call tools; only the local one has place search.

| | Hosted (remote) | Local (stdio) |
|---|---|---|
| Endpoint | `https://api.voyp.app/mcp/stream` (Streamable HTTP) | `npx -y voyp-mcp` |
| Auth | OAuth 2.1 with PKCE (the client signs in through the browser), or a Voyp API key as `Authorization: Bearer sk_...` | `VOYP_API_KEY` environment variable |
| Tools | `start_call`, `get_live_call`, `get_call`, `hangup_call`, `get_user` | `start_call`, `get_call`, `hangup_call`, `search_places`, `search_place`, `search_place_by_number`, `get_user` |

All tools return JSON as text. The hosted server also returns `structuredContent`.
When a tool fails, the result has `isError: true`, plus `{"success": false, "error": "..."}`
(hosted) or `Voyp API error: ...` (local).

---

## start_call

Starts an outbound call. **This places a real call and uses credits.** Confirm with the user first.

| Argument | Type | Required | Notes |
|---|---|---|---|
| `number` | string | yes | E.164, e.g. `+14155550123` |
| `context` | string | yes | Full briefing for the voice agent. See [call-context.md](call-context.md) |
| `language` | string | hosted: yes, local: no | BCP-47 tag of the language spoken on the call. Default `en-US` |

The voice, recording, and business settings come from the user's Voyp profile.

**Hosted response**
```json
{
  "success": true,
  "id": "CA1234...",
  "active": true,
  "status": "in-progress",
  "number": "+14155550123",
  "context": "You are calling ...",
  "language": "en-US",
  "liveUrl": "https://www.voyp.app/live/CA1234...",
  "messages": [],
  "updatedAt": "2026-11-07T10:00:00.000Z",
  "duration": null,
  "summary": null,
  "recordingUrl": null
}
```

**Local response**
```json
{ "success": true, "id": "CA1234...", "url": "https://www.voyp.app/live/CA1234..." }
```

---

## get_live_call (hosted only)

Status and transcript while a call is running. Returns the same shape as `start_call`.
`active` is `true` while the call is running. When it becomes `false`, call `get_call`.

| Argument | Type | Required |
|---|---|---|
| `id` | string | yes |

`messages` holds the transcript so far:
```json
[
  { "id": "1", "role": "callee",    "text": "Hello, Luigi's, how can I help?", "time": 1731000000 },
  { "id": "2", "role": "assistant", "text": "Hi, I'd like to book a table for four tonight.", "time": 1731000004 }
]
```
`assistant` is the Voyp agent. `callee` is the person who answered.

---

## get_call

- **Hosted:** the final details of a call that has ended: the same shape as above with
  `active: false`, plus `summary`, `duration` (seconds), and `recordingUrl` when recording is on.
- **Local:** `{ "call": { ... } }` with the call record. Use its `state` / `status` field to see if
  the call has ended (`completed`, `failed`, `busy`, `no-answer`, `canceled`), and read its
  transcript and summary fields.

| Argument | Type | Required |
|---|---|---|
| `id` | string | yes |

---

## hangup_call

Ends a call that is running. This cannot be undone.

| Argument | Type | Required |
|---|---|---|
| `id` | string | yes |

Response: `{ "success": true }` or `{ "success": false, "error": "..." }`.

---

## search_places (local only)

Finds businesses that match a query with a location.

| Argument | Type | Required | Example |
|---|---|---|---|
| `search` | string | yes | `italian restaurants in New York City, US` |

Returns a list of places (name, address, phone number, rating, opening hours when available).

## search_place (local only)

Details of one business, including its phone number.

| Argument | Type | Required | Example |
|---|---|---|---|
| `place` | string | yes | `The Lane Salon` |
| `location` | string | yes | `San Francisco, CA` |

## search_place_by_number (local only)

Finds the business name and address for a phone number.

| Argument | Type | Required | Example |
|---|---|---|---|
| `number` | string | yes | `+14155550123` |

---

## get_user

The signed-in user's profile. Use it to check that the connection works and to get the
user's name and preferred language.

- Hosted: `{ "user": { "id": "...", "name": "...", "language": "en-US" } }`
- Local: `{ "user": { ... } }` with the full profile

## OAuth scopes (hosted)

An API key has all three scopes. With OAuth, the user chooses what to grant.

| Scope | Tools |
|---|---|
| `voyp.calls.write` | `start_call`, `hangup_call` |
| `voyp.calls.read` | `get_call`, `get_live_call` |
| `voyp.profile.read` | `get_user` |
