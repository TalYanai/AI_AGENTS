# Product brief: weekly status report for my team

**My role:** coordinator and sole builder. **Status:** in regular use (stated by me; no usage data kept). **Form:** a Claude skill with a frozen report template and a data file.

## 1. Problem
I coordinate a team of 12 instructors working in partner frameworks across the south. Every week I need a clear picture of the team for the meeting with my manager: placements, challenges, schedule changes and upcoming events. Compiling it by hand meant collecting scattered notes, writing the structure from scratch each time, and risking an inconsistent layout or a missed issue.

## 2. Users
| User | Role | What they need |
|---|---|---|
| Me (operator) | Gives the week's updates, reviews the output | A report in minutes, and a continuous view of everyone on the team |
| My manager (reader) | Reads it before our weekly meeting | The same predictable, scannable layout each week, with the risks and the wins visible |
| Peer coordinators (observers) | Saw a demonstration | A repeatable way of reporting they might adopt |

## 3. Solution
A Claude skill with a fixed three-page Hebrew (right-to-left) PDF design. The design lives in a script and never changes. Everything that changes from week to week lives in one data file.

Each week:
1. I tell the skill the updates: a changed meeting date, a new placement, a new challenge, notes from the week.
2. It applies only those updates to the data file, and fixes everything the new date makes stale (relative labels such as "today" or "next week", the order of the timeline).
3. It fills a fixed field-report block with one good point from the week and one challenge to handle, taken from my notes.
4. It builds the PDF and checks that it is exactly three pages with nothing cut off or overlapping.
5. It returns the PDF and the updated data file, and states in one or two sentences what changed and what looks stale or uncertain. I review before it goes to my manager.

## 4. Decisions and trade-offs (visible in the skill's rules)
- **Frozen design, separate data.** The report looks the same every week; only the data file changes. This keeps the output predictable for the reader and limits what can go wrong in a run.
- **Never invent facts.** Names appear exactly as in the data, and anything stale or uncertain is flagged in the delivery note instead of being filled in.
- **Colour means status, not people.** Status colours describe an item (done, in progress, gap, focused follow-up), and a rule forbids using red for a person.
- **Return the data file each week.** The next run starts from it, so no database or shared platform is needed.
- **Human in the loop.** I trigger it and review it. The skill does not collect information from chats or email on its own.
- **One region, one user.** I built it for my own team first, rather than a multi-user tool for all coordinators.

## 5. Measures
These are my own estimates. Nothing was measured.
- Compiling the weekly report: saves about two hours.
- Daily follow-up of the team: at least half an hour less each day.
- Adoption: used every week by me and my manager; I presented the method to peer coordinators.

## 6. Limits and what I would do next
- Single user and a single team. It runs on my computer and needs a local browser engine to render the PDF.
- No run history or usage numbers, so the time savings are estimates.
- Next: measure the time for four weeks, collect my manager's feedback on the layout, and decide with peer coordinators whether a shared version is worth building.
