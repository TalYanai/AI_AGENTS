# AI agents and automations

Case studies of AI assistants and automations that I designed and specified. I describe the behaviour, the guardrails and the data flow, and I assemble the result with low-code tools (Make, n8n) and AI coding assistants.

**Status:** documentation and design notes. Nothing here claims production use, usage numbers or outcomes; those were not verified. Where something is a prototype or has known flaws, the write-up says so.

## Contents

- [`make/case_study.md`](make/case_study.md): sign-up automation and a chat-assistant prototype for a community organisation (seven Make scenarios, 147 modules).
- [`make/technical_notes.md`](make/technical_notes.md): sanitised structure of the seven scenarios, error handling and assistant design.
- `make/*.svg`: six flow diagrams.
- [`product/weekly-status-report-prd.md`](product/weekly-status-report-prd.md): product brief for a weekly status-report automation I use with my own team (problem, users, decisions, estimated measures, limits).
- [`agents/agent-patterns.md`](agents/agent-patterns.md): five agent patterns from my projects, with an evidence column.

Related: n8n workflow documentation lives in the [n8n](https://github.com/TalYanai/n8n) repository.

## Honest status

| Piece | Status | Evidence |
|---|---|---|
| Make scenarios (7) | Complete exported blueprints; production use not verified | Static reading of exports |
| Community chat assistant | Documented prototype; live use not verified | Instructions and embed page |
| Agent patterns (5) | Mixed; see the table in the patterns file | Code reviewed, export, prototype |

Names, identifiers and links from the original material are removed; product names are generalised.
