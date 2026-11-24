# How I structure agents

Five patterns from my own projects. I design and specify these systems and build them with low-code tools and AI coding assistants. The evidence column says how far each claim is backed. Nothing here claims production use, usage numbers or outcomes.

| Project | Pattern | Control and guardrails | Evidence |
|---|---|---|---|
| Personal assistant | An orchestrator calls four specialised agents (sports, market data, fitness, calendar) and Claude composes one morning brief. | Chat questions are routed by a keyword list, not by a model. | Code reviewed |
| Agent Builder | An orchestrator and six agents, wired on a visual canvas. | A Claude classifier returns one word (capped at ten tokens) to choose the agent, with a general fallback. | Code reviewed |
| PartyPlanner | Two single-purpose agents in sequence: a Designer, then a Sender. | The Sender runs at temperature 0 with two scoped e-mail tools. Validation runs before any model call. | Workflow export |
| Community assistant | Intent-based dialog with structured JSON output for an automation platform to act on. | One question at a time, no invented data, hard and soft escalation to a person. | Documented prototype |
| TYFitness and Moodle MCP | Tools exposed to a model through MCP. | Read-only tool allow-lists (three tools for Garmin, eight for Moodle) and token scrubbing. | Running, live check |

Evidence terms: "Code reviewed" means the source was read, not necessarily run in this review; "Workflow export" means a static reading of an exported n8n file; "Documented prototype" means written instructions and exports exist but live use is not verified; "Running, live check" means a live check was made at the time of writing.

## What these taught me
- Keep each agent single-purpose and give it only the tools it needs.
- Move deterministic steps out of the model: in PartyPlanner the image rendering became a plain node.
- Routing by keyword is cheap but brittle, which is why Agent Builder uses a classifier.
- Validate input before any model call, so tokens are not spent on empty requests.
- Prefer read-only tool allow-lists when a model reaches personal data.

Related write-ups: [Make case study](../make/case_study.md), [Make technical notes](../make/technical_notes.md).
