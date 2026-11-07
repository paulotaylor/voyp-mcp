# Writing the call context

The `context` is everything the Voyp voice agent knows. It cannot ask you questions during
the call, and it does not see your conversation with the user. If something is not in the context, the
agent will not know it, or will guess.

## Checklist

A good context covers:

1. **Who and on whose behalf.** "You are calling Luigi's Trattoria on behalf of Maria Silva."
2. **The goal**, in one sentence. "Book a dinner table."
3. **The details** the callee will ask for: date, time, party size, names, reference numbers,
   callback number, address, item, budget.
4. **Acceptable alternatives**, with a clear order of preference. "8pm. If not possible, any time
   from 7:00 to 9:30pm. Indoors preferred, terrace is fine."
5. **What to find out.** "Ask whether they have vegan options and if there is parking."
6. **Limits.** What the agent must not agree to or share. "Do not give a deposit or card
   details. If they need a deposit, say Maria will call back to pay."
7. **How to end.** "Repeat the booking back to confirm it, get a confirmation name or number,
   thank them, and say goodbye."
8. **When to give up.** Voicemail, a wrong business, or a request outside the limits.
   "If you reach voicemail, leave no message and hang up." Or give the exact short message to leave.

Use dates the callee will understand ("Saturday 14 November" rather than "next Saturday"), and
write times in the local style. Keep it to what this call needs; long background confuses the agent.

## Template

```
You are a polite assistant calling <BUSINESS / PERSON> on behalf of <USER NAME>.

Goal: <one sentence>.

Details:
- <detail 1>
- <detail 2>
- Callback number, only if they ask: <number or "do not share">

If the first choice is not available: <alternatives, in order of preference>.
Also find out: <questions>.

Do not: <limits, e.g. pay, give card details, agree to a deposit, sign up for anything>.
If they ask for anything outside these instructions, say <USER NAME> will follow up directly.

To finish: repeat back what was agreed, ask for a confirmation name or number, thank them and end the call.
If you reach voicemail or a phone menu you cannot get through: <leave this message / hang up>.
If asked, say you are an AI assistant calling for <USER NAME>.
```

## Example: good

```
You are a polite assistant calling Luigi's Trattoria on behalf of Maria Silva.

Goal: book a dinner table for 4 people on Saturday 14 November.

Details:
- Name for the booking: Maria Silva
- Party: 4 adults, one is vegetarian
- Preferred time: 8:00pm
- Callback number, only if they ask: +1 415 555 0199

If 8:00pm is not available, accept any time from 7:00pm to 9:30pm, earlier is better.
If nothing is free on Saturday, do not book; ask what is available on Friday 13 November
and report it.
Also find out: whether they have vegetarian mains.

Do not give card details or agree to a deposit. If one is needed, say Maria will call back.

To finish: repeat the date, time, party size and name, ask for a confirmation number if they
have one, thank them and say goodbye.
If you reach voicemail, hang up without leaving a message.
If asked, say you are an AI assistant calling for Maria.
```

## Example: too thin

```
Book a table at Luigi's for Saturday.
```
This has no name, party size, time, alternatives, or limits. The agent will have to guess or fail.

## Language

Set the `language` argument of `start_call` to the language the callee speaks (`pt-PT` for a
Lisbon dentist, `fr-FR` for a Paris bakery). Names, addresses, and reference numbers in the
context are kept as written.
