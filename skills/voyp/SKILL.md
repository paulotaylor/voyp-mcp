---
name: voyp
description: Make real outbound phone calls on the user's behalf with the Voyp MCP server — book restaurant tables, schedule appointments, ask businesses questions, check opening hours or availability, follow up on orders. Use when the user asks you to call, phone, ring or contact a business or person by phone, or wants something done that needs a phone call, and Voyp MCP tools (start_call, get_call, get_live_call, hangup_call, search_places, search_place, search_place_by_number, get_user) are available.
license: MIT
compatibility: Requires the Voyp MCP server, hosted (https://api.voyp.app/mcp/stream, OAuth) or local stdio (npx voyp-mcp with VOYP_API_KEY), and a Voyp account with call credits.
metadata:
  author: Control Space BV
  homepage: https://voyp.app
  repository: https://github.com/paulotaylor/voyp-mcp
  version: "1.0.0"
---

# Voyp phone calls

Voyp places a **real phone call** to a real person. A Voyp voice agent talks to whoever answers.
That agent knows **only** the `context` you give it, so you brief it like an assistant you
are sending to make the call. Every call uses the user's paid credits and reaches someone
who did not ask to be called, so check before dialling and keep the call short.

## Tools

The tool names come from the Voyp MCP server. Your client may add a prefix, such as `mcp__voyp__start_call`.

| Tool | Use it to | Server |
|---|---|---|
| `start_call(number, context, language)` | Place the call. Returns the call `id` and a live URL | both |
| `get_live_call(id)` | Status and transcript while the call is running | hosted |
| `get_call(id)` | Details and transcript (hosted: the final result after the call ends) | both |
| `hangup_call(id)` | End a call that is running | both |
| `search_places(search)` | Find businesses, e.g. "sushi restaurants in Lisbon" | local |
| `search_place(place, location)` | Get the phone number and details of one business | local |
| `search_place_by_number(number)` | Find which business a number belongs to | local |
| `get_user()` | Check the connection and the user's profile | both |

If a tool is missing, use the ones you have. For example, with no place search tool, get the
number from the user or a web search. See [references/tools.md](references/tools.md) for
arguments and responses.

## Workflow

### 1. Understand the task
Find out who to call, what outcome the user wants, and the details the callee will ask for:
name for the booking, date and time, party size, order or account number, callback number.
Also ask what is acceptable if the first choice is not available ("8pm, or anything between
7 and 9", "any weekday morning"). Ask the user for anything essential that is missing. Do not invent
personal details.

### 2. Find and check the number
- If the user did not give a number, use `search_place` (or `search_places`) to find it.
  If several businesses match, show the options and let the user choose.
- Convert the number to **E.164**: `+` then country code then number, with no spaces
  (for example `+14155550123` or `+351212345678`). If the country is unclear, ask.
- Use `search_place_by_number` when the user gives a bare number and you need to check what it is.

### 3. Write the call context
This is what decides whether the call works. Follow
[references/call-context.md](references/call-context.md). In short, the context must say:
who the call is for, the goal, every detail the callee may need, acceptable alternatives, what
to find out, what **not** to agree to or share, and how to end the call.
Write it in the second person ("You are calling … on behalf of …").

Set `language` to the language the callee speaks, as a BCP-47 tag (`en-US`, `en-GB`, `pt-PT`, `fr-FR`,
`es-ES`, `de-DE` …). Use the language of the place you are calling, not the user's language. Write the context in
any language; the agent speaks the one in `language`.

### 4. Confirm with the user — always
Before `start_call`, show the user:
- the business or person and the **number** in E.164 format,
- the call **language**,
- a short summary of what the agent will say and what it may agree to.

Wait for a clear yes. Repeat this for each call, including retries and calls to a different
number. Earlier approval does not cover a new call. The only exception is when the user has
already approved this exact call (number and purpose) in the same message, for example
"yes, call +1 415 555 0123 and book for 4 at 8pm".

### 5. Start the call
Call `start_call`. Give the user the **live URL** (`liveUrl`, or `url` on the local server) so
they can listen to or follow the call, and the call `id`.

### 6. Follow the call
- **Hosted server:** call `get_live_call(id)` about every 15–30 seconds while `active` is
  `true`. Tell the user how it is going only when something changes. Do not paste every transcript update.
  When `active` becomes `false`, call `get_call(id)` once for the final transcript and `summary`.
- **Local server:** call `get_call(id)` about every 15–30 seconds until the call status shows
  it has ended (`completed`, `failed`, `busy`, `no-answer`, `canceled`).
- If your client cannot wait between calls, tell the user the call is running, give the live
  URL, and offer to check the result when they ask.
- Stop following the call after about 10 minutes with no change, and tell the user.

### 7. End the call early only when needed
Use `hangup_call(id)` when the user asks you to, or when the call is clearly going wrong: the
wrong business, an automated phone menu loop, or the agent being asked to agree to something
the user did not approve. This ends a live conversation, so do not use it for small issues.

### 8. Report back
Tell the user the result in a few lines: what was agreed (date, time, name, reference
number, price), anything still open, and what the user needs to do next. Use the transcript;
do not guess. If the call failed (busy, no answer, voicemail, wrong number), say so and offer
another try. That new try needs approval again, as in step 4.

## Rules

- **Never** call emergency numbers (911, 112, 999, 000 …). Tell the user to call those themselves.
- Do not make harassing, threatening, deceptive, spam, marketing, or bulk calls, or calls
  pretending to be a real person other than "an assistant calling on behalf of <user>".
  The agent must say it is an AI assistant if asked.
- Share only the personal details the task needs. Never put passwords, full card
  numbers, PINs, or government ID numbers in the context. If the callee asks for payment, the
  agent should say the user will complete payment themselves.
- One call at a time to the same number unless the user asks for more. Do not redial
  again and again: ask the user before each retry.
- Calls use credits. If `start_call` returns an error about credits or payment, tell the user
  they can buy credits at https://voyp.app/app.html.

## Errors

| Symptom | What to do |
|---|---|
| `401` / "Invalid or expired access token" | Ask the user to sign in to Voyp again (hosted) or check `VOYP_API_KEY` (local) |
| "Missing required OAuth scope" | Ask the user to reconnect and grant the calls and profile permissions |
| "Unable to start call" | Check the E.164 number, then the user's credits; tell the user |
| "Call not found" right after starting | Wait a few seconds, then try again |
| `busy` / `no-answer` / `failed` | Report it and offer to try later (with new approval) |

## More detail

- [references/tools.md](references/tools.md): arguments and responses of each tool
- [references/call-context.md](references/call-context.md): how to write a good call context, with a template
- [references/examples.md](references/examples.md): complete example conversations
