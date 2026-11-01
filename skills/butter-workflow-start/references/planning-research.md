# Planning Research

Track B/C only. A plan needs the repository read from several angles, and those angles do not depend on each other. Send them out together so the wait is the slowest strand rather than the sum of all of them.

## Axes

- `reuse` — functions, modules, and utilities that already do what this task needs, and the sites that call them.
- `convention` — how this repository already writes this kind of code: naming, file placement, layering, error handling, logging, where tests live and what they look like.
- `impact` — the files the change actually touches, what calls or depends on them, and which contracts could break.
- `verification` — build, test, lint, and type-check commands, the test framework, and where the existing tests for this area live.
- `external` — the referenced issue, PR, or ticket. Use MCP first and `gh` when MCP is unavailable.

Dispatch every applicable axis in one batch. Use one agent per axis when capacity allows. If fewer subagent slots are available than applicable axes, group axes across the available agents before dispatching so every axis starts in that batch; do not wait for one agent to finish solely to reuse its slot. Drop an axis only when this task gives it nothing to find: no referenced external resource means no `external` axis.

Give each agent:

- The user's task context, verbatim and unsummarized.
- Its assigned axis or axes and what each axis is looking for.
- The project instruction file paths and `~/.agents/preferences/preferences.md`.

Instruct each agent to:

1. Stay read-only. Do not write files, edit the spec, or create branches or commits.
2. Report findings under each assigned axis, not proposals. What to build is the main agent's call.
3. Anchor every repository finding to a repository location and every external finding to its source URL or resource identifier. Mark anything it could not anchor as unconfirmed.
4. Say so explicitly when a search came up empty. Silence is not the same answer as nothing found.

When the tool has no subagent mechanism, work the axes yourself in sequence. Same axes, same read-only rule, same evidence requirement. Only the place they run changes.
