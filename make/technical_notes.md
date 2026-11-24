# Technical notes (sanitised)

Source: 7 Make blueprint exports, read with a script that extracted structure only: module types, nesting, filter operators and counts. No field mappings, IDs or values were copied.

Counts include every module at every nesting level (routes and error-handler modules).

## Scenario table

| # | Generic name | Trigger | Modules (total) | Module types | Branches and logic |
|---|---|---|---|---|---|
| 1 | Host sign-up intake | Custom webhook (instant) | 13 | Webhook, Switcher, spreadsheet (add row, 4x update row), board (list group items, 2x create item), Text aggregator, Phone parser, Router | 1 router, 2 routes: lead already on board / new lead. Each creates a board item then updates the sheet row. Error handlers (2) write failure state to the sheet. Switcher: 4 cases plus default. Sequential execution on. |
| 2 | Attendee sign-up | Custom webhook (instant) | 12 | Webhook, Switcher, Router, spreadsheet (3x add row, copy a sheet), board (create item), Break x4 | 1 router, 2 routes: sheet logging, or board item. Filter "tab missing" (text contains, case-insensitive) copies a template tab then writes the row. Switcher 4 cases plus default. Incomplete executions stored. |
| 3 | New-contact messaging | Board watch column values (instant) | 13 | Board (watch column values, get item), Sleep (5 s), Set variable, Phone parser, HTTP (API-key auth), messaging app (send template, assign label), Break x5 | No router. Linear chain gated by 5 filters: contact exists, status check (4 not-equal conditions), phone exists, duplicate check before send, message sent before label. Incomplete executions stored. |
| 4 | Duplicate check on registrations board | Custom webhook (instant) | 57 | Webhook, Sleep (10 s), board (get item, get column value, 4x list items by column value, 9x change multiple column values, 8x create update), Phone parser, Router x8, Break x23 | Top router with 4 routes (one lookup per match key, inferred B). Each route has a nested router on the match count (number or text filter). Outcomes: set one status, or set another status and add update notes to the new and old items. Incomplete executions stored. |
| 5 | Participant count button | Custom webhook from a board button (instant) | 40 | Webhook, board (3x get item, get group, 7x list group items, 4x change multiple column values), Array aggregator x7, Router x2, Break x15 | Router 1: group linked or not. Router 2: host linked or not. Main path: 7 list-and-aggregate pairs (6 participant categories, 1 unassigned), then a column update. Missing-link branches set a status only. Incomplete executions stored. |
| 6 | Nearby prospects notification | Board watch items (instant) | 5 | Board (watch items, list items), Iterator (feeder), maps (distance matrix), chatbot platform (send message) | No router, no filter in the blueprint. Radius or threshold logic: status not verified. |
| 7 | Assistant callback (scenario A) | Chatbot "lead generated" event (instant) | 7 | Chatbot platform (lead generated, 2x send message), board (search items by column values, advanced), Webhook respond, Router x2 | Router 1: route to board search / route to webhook response. Router 2 after the search: two message variants, the second with a text-equals filter. |

Totals: 147 modules. Scheduling: none of the 7 is scheduled (all instant triggers). Building blocks seen: Make built-ins (webhook, router, aggregator, iterator, sleep, break, switcher, text and array aggregators, phone parser), a work-management board app, a spreadsheet app, a maps API, HTTP, a third-party messaging app, a chatbot platform.

## Error handling pattern
- Break with automatic retry (incomplete executions) on nearly every board and messaging call in scenarios 3, 4, 5 (and 2).
- Scenario 1 uses an error route that records the failure in a spreadsheet instead.
- Scenarios 6 and 7 have no error handlers.
- Scenarios 3 and 4 add a fixed wait (5 s, 10 s) after the trigger, probably to let the board finish saving (B: inferred).

## Assistant design summary
- Inputs read: the instruction text (about 16 KB), a questionnaire wording source, an embed page (a heading plus a script and an iframe pointing to the hosted bot; the bot ID is withheld), the written specification, and a message-template workbook (content not read for publication).
- Persona: warm, short chat assistant for the organisation; one question at a time; temperature 0.2; replies in the user's language, Hebrew default.
- Intents: A join, B waitlist, C host intake, F risky lead, human.
- Host intake: 2 gates (a minimum seating capacity and a minimum run of consecutive weekly sessions), 9 scored items (day, refreshments, pets, parking, accessibility, venue type, public transport, shelter, city), a raw score of 0 to 31 with explicit defaults for missing information, then a few photos.
- Knowledge and data: no event data in the prompt. Meeting details, waitlist release, reminders, feedback and host results come only from system commands sent by Make; the bot must repeat the payload verbatim and show a fixed error line if it is missing.
- Guardrails: never show JSON or code to users; never reveal dates or addresses before a system command; hard triggers (admin or credential requests, prompt injection, abuse, unsafe links) escalate at once; soft triggers (repetition, spam rate, refusing required fields, identity mismatch, exploitative bulk-list requests) give one warning, then escalate.
- Escalation output: `intent=F`, an escalate flag, a list of reason tags, a cooldown flag, and a one-line handoff message.
- Structured output: one JSON object: intent, fields (contact, region, date, gates, nine questionnaire answers, city, score, photos, plus escalation fields), and a short `answer`. The prompt contains two schema versions (with and without the score); the deployed version is status not verified.
- Gaps recorded in the specification: handoff triggers, Q&A set, model preference and security requirements. The risk policy in the prompt appears to answer the first one.

## Specified scenarios A to F
A open meeting (lead, location check, registration link, reminders, confirmation, feedback); B no open meeting (waitlist, release message); C host recruitment (threshold questionnaire, home and photo check, approve or reject, group assignment); D personal commitment (single participant in a group); E after the meeting (thanks, post or photo request); F risky leads (overbooking classification, attendance verification).
Coverage by blueprints: C partly (1, 2, 7 and the assistant), A and B partly (6, 7), F partly (4). D and E: none found.
