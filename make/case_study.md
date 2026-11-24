# Automating sign-ups and prototyping an AI assistant for a community organisation

*Sanitised case study. No organisation, people, boards, sheets or links are named. Product names are replaced by generic terms.*

## Summary

**The problem.** A community organisation that runs small recurring meetings handled most sign-ups, duplicate checks, reminders and host recruitment by hand, on top of its existing work-management boards. Attendees, prospective hosts and coordinators all needed different follow-ups, and the manual route was uneven.

**My role.** As a volunteer partner, I designed the automations and specified them in detail. I assembled them in Make's visual editor, with AI coding assistants helping, and I prototyped a chatbot assistant on a no-code chatbot platform. I worked on top of the organisation's existing boards, starting from a written specification that described six scenarios (open meeting, waitlist, host recruitment, personal commitment, post-meeting follow-up, risky leads).

**How I mapped the process.** For each scenario I wrote down the trigger, where the data lives, the decision points and the outgoing message. Then I built one small scenario per trigger.

**What I built in Make.** Seven scenarios, 147 modules in total. All are event-driven; none runs on a timer.
- *Host sign-up intake (13 modules):* webhook, classify the submission, log to a spreadsheet, check existing leads, normalise the phone number, then create or flag a board item.
- *Attendee sign-up (12):* webhook, classify, log to a spreadsheet (copying a template tab if the tab is missing), and create a board item, with retry handlers.
- *New-contact messaging (13):* a board status change creates a contact and sends a template message through a messaging app.
- *Duplicate check (57):* four lookups on the board, a match count, then status updates and notes on old and new items.
- *Group count button (40):* counts group members per category and writes the totals back.
- *Nearby prospects (5) and assistant callback (7):* connect the board to the assistant.

**The assistant.** A chat assistant with a written instruction set: it detects one of five intents, asks one question at a time, replies in the user's language (Hebrew by default), and returns structured output for Make to process. Host recruitment has two pass/fail gate questions, then a nine-question scoring form (0 to 31) and a request for photos. A risk policy escalates to a person.

**Shipped versus prototyped.** The Make scenarios exist as complete, exported blueprints. Whether they ran in production, and for how long, is status not verified. The assistant is a documented prototype with an embed page; its live use is status not verified.

**Lessons.** Keep scenarios small and trigger-based, wait briefly before reading a fresh item, and retry on error so failures are not lost.

**Limitations.** I have no usage numbers or outcome data. The specification left open when to hand off to a person and which language model to use. One scenario has no visible radius filter.

Diagrams: [01](01_registration_intake.svg), [02](02_new_contact_message.svg), [03](03_duplicate_check.svg), [04](04_group_count_button.svg), [05](05_assistant_and_nearby_prospects.svg), [06](06_assistant_dialog.svg).

## Appendix

### A. Problem and context
The specification targeted: lead handling by location, three reminders with confirmation, structured host recruitment, automatic feedback, and organic marketing. Systems named: work-management boards, a messaging-API provider (decision still open), Make, a calendar, and BI reporting. The service area was deliberately narrow, and the specification lists missing inputs (handoff triggers, a Q&A set, model and security preferences).

### B. Scenario mapping
| Specified scenario | Evidence in the blueprints |
|---|---|
| A open meeting | Assistant callback scenario (B: inferred link to scenario A) |
| B waitlist | Nearby-prospects scenario (B: inferred) |
| C host recruitment | Host sign-up intake plus assistant host path (scored) |
| D personal commitment | No scenario found |
| E post-meeting | No scenario found |
| F risky leads | Duplicate check and the assistant's risk policy (B: inferred) |

### C. The automations in detail
1. **Host sign-up intake (13 modules).** Webhook, a switcher with four cases and a default, a sheet row, a list of items in one board group, a text aggregator, phone parsing, then a router with two branches (lead already present, lead new). Each branch creates a board item and updates the sheet row. If creation fails, an error handler writes the failure to the sheet instead of stopping.
2. **Attendee sign-up (12).** Webhook, switcher (four cases and a default), router with two routes. Route one writes a sheet row; if that fails, a filter on "tab missing" copies a template tab and writes again. Route two creates a board item. Four Break handlers retry failed steps.
3. **New-contact messaging (13).** Watch a board column, wait 5 seconds, read the item, check the contact exists, check four status conditions, parse the phone, HTTP call with API-key auth, send a template message (filtered by a duplicate check), assign a label once sent. Five Break handlers.
4. **Duplicate check (57).** Webhook, wait 10 seconds, read the item and a column value, parse phone, then a router with four routes. Each route runs a board lookup by column value and a nested router on the match count: one outcome updates the item status, the other changes status and adds update notes to both the new and the old item. 23 Break handlers, 8 routers, 9 status-update modules, 8 note-creation modules.
5. **Group count button (40).** Webhook from a board button, read the item, router (is a group linked?), read the host item (is a host linked?), read the group, then seven list-and-aggregate pairs: six participant categories plus unassigned. A final module writes the counts. Branches without a group or host just set a status. 15 Break handlers.
6. **Nearby prospects (5).** Watch for new board items, list board items, iterate, distance lookup, send assistant message. No filter appears in the blueprint, so the radius rule is status not verified.
7. **Assistant callback (7).** Assistant lead event, router, board search by column value, a second router, two assistant messages (one filtered), and a webhook response.

Error handling: four scenarios (attendee sign-up, new-contact messaging, duplicate check, group count) use Break with automatic retry and store incomplete executions; host intake instead writes failures to the sheet. Scenario scheduling: none; all instant.

### D. Assistant design
- **Persona and scope:** a warm, concise chat assistant for the organisation; one question at a time; no code or JSON shown to the user.
- **Intents:** A join an open meeting; B waitlist; C host intake; F risky lead; human handoff.
- **Knowledge:** the instruction text and a separate wording source for questionnaires. Event details are never invented; real data only arrives through named system commands from Make (for example show meeting, waitlist release, reminder, feedback).
- **Host intake:** two mandatory gates (a minimum seating capacity and a minimum run of consecutive weekly sessions), then nine scored questions, a total of 0 to 31, and a few photos.
- **Guardrails:** low temperature (0.2); prompt-injection and credential requests escalate at once; softer signals (repetition, spam, refusal to answer, identity mismatch, exploitative requests) get one warning and then escalate.
- **Structured output:** one JSON object with intent, a fields block (contact data, gate answers, venue answers, photos, risk flags) and a short reply. A later revision of the schema drops the score and adds escalation flags and a cooldown flag; which version was deployed is status not verified.
- **Languages:** replies in the user's language, Hebrew by default.

### E. What was shipped versus prototyped
- Scenarios: complete, exported blueprints (A). Production status not verified.
- Assistant: instructions, wording source and an embed page (A). Live use status not verified.
- A message-template sheet of outbound messages exists (A); whether it was loaded into the bot is status not verified.

### F. Lessons and limitations
Short waits before reading new items avoid race conditions. Retry handlers were applied almost everywhere. The duplicate scenario grew large because each match key repeats the same logic; a sub-routine or a single lookup would be cleaner. No test records, run history, error rates or user feedback exist in the material, so I make no claims about impact.
