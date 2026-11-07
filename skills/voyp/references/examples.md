# Example sessions

Tool names are shown without a client prefix.

## 1. Restaurant booking (local server, with place search)

**User:** Book me a table for 2 at Bella Notte in San Francisco on Friday at 7:30. Name is Alex Kim.

**Agent:** calls `search_place(place="Bella Notte", location="San Francisco, CA")` and gets the
address and the number `(415) 555-0142`, which becomes `+14155550142`.

**Agent:**
> I found Bella Notte at 123 Example St, +1 415 555 0142. I'll call them in English (en-US) and:
> - book a table for 2 on Friday 13 November at 7:30pm, under Alex Kim,
> - accept 7:00–8:30pm if 7:30 is taken,
> - not give any card details.
>
> Shall I call now?

**User:** Yes.

**Agent:** calls `start_call(number="+14155550142", language="en-US", context="You are a polite assistant calling Bella Notte restaurant on behalf of Alex Kim. Goal: book a dinner table for 2 people on Friday 13 November at 7:30pm ...")`.

> Calling Bella Notte now. You can follow the call live at https://www.voyp.app/live/CA… .

The agent calls `get_call(id)` every ~20 seconds until the call status is `completed`, then:

> Done. You have a table for 2 at Bella Notte on Friday 13 November at **7:45pm** (7:30 was
> full), under Alex Kim. They'll hold it for 15 minutes.

## 2. Appointment in another language (hosted server)

**User:** Call my dentist, Clínica Sorriso, +351 21 000 0000, and move my Thursday appointment to next week, any morning. I'm João Pereira.

**Agent:**
> I'll call Clínica Sorriso at +351210000000 in Portuguese (pt-PT) to move João Pereira's
> appointment on Thursday 12 November to any morning from 16 to 20 November. If no morning is free, I'll
> ask for the options and not change anything. OK to call?

**User:** Go ahead.

`start_call(number="+351210000000", language="pt-PT", context="You are a polite assistant calling Clínica Sorriso on behalf of João Pereira ...")`, then `get_live_call(id)` every ~20s while `active` is `true`, then `get_call(id)` for the `summary`.

> Your appointment is now on **Tuesday 17 November at 10:00**. The old Thursday slot was cancelled.

## 3. Ending a call early

**User:** Stop, that's the wrong place.

**Agent:** calls `hangup_call(id)`.
> I've ended the call. Do you want me to look up the right number?

## 4. A question only

**User:** Can you ask the bike shop on Main Street if they have a 54cm road bike in stock?

The agent finds the number, confirms the number and the exact question with the user,
starts the call with a context that only asks about stock (and says "do not reserve or buy
anything"), and reports the answer.
