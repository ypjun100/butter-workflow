<p align="center">
  <img src="assets/butter-workflow-banner.png" alt="Butter Workflow banner" width="100%">
</p>

<h1 align="center">Butter Workflow</h1>

<p align="center">
  Portable issue-based Spec-Driven Development as a shared skill for any AI coding agent.
</p>

<p align="center">
  <img alt="Agent Skill" src="https://img.shields.io/badge/Agent%20Skill-SKILL.md-111827?style=flat-square">
  <img alt="Spec-driven workflow" src="https://img.shields.io/badge/workflow-spec--driven-F59E0B?style=flat-square">
</p>

<p align="center">
  <a href="#what-it-provides">What It Provides</a> ·
  <a href="docs/workflow-proposal.md">Workflow Design</a> ·
  <a href="#installation">Installation</a> ·
  <a href="#usage">Usage</a> ·
  <a href="#track-model">Track Model</a>
</p>

Butter Workflow keeps the working context in repository files so a task can
move between tools, models, and sessions without depending on chat history.

See [Workflow Design](docs/workflow-proposal.md) for the full methodology and
handoff model.

## What It Provides

- Four shared workflow stages: start, implement, code-review, and finish.
- One shared skill set (`skills/`) that works with any agent supporting the
  Agent Skills format (`SKILL.md`) — no tool-specific command wrappers.
- Track A/B/C routing for small changes, planned changes, and high-risk changes.
- Handoff spec documents under `docs/specs/{TASK-ID}/` for every track.
- Shared user preference memory under `~/.agents/preferences/`.

## Installation

Install directly from this GitHub repository with the
[skills.sh](https://www.skills.sh/) CLI. No clone required:

```bash
npx skills add ypjun100/butter-workflow
```

Update to the latest version later with:

```bash
npx skills update
```

This works the same way for Claude Code, Codex, and any other agent that
supports the shared Agent Skills format. Trade-off to know: unlike a plugin
marketplace, `npx skills` has no version pinning or changelog — it installs
whatever is on the default branch, so check this repository's commit history
before updating if that matters to you. Restart the tool or start a new
session after installing so the newly loaded skills are available.

## Usage

Describe your intent in natural language and the matching stage runs
automatically:

```text
Start this task: <task context>
Implement the approved plan.
Review this PR: <PR URL>
Finish up and capture preferences.
```

To be explicit about which stage runs, mention the skill by name instead:

```text
Use the `butter-workflow-start` skill with <task context>
Use the `butter-workflow-implement` skill
Use the `butter-workflow-code-review` skill with <PR URL>
Use the `butter-workflow-finish` skill
```

On Codex, skills are invoked with a `$` prefix (`$butter-workflow-start`,
`$butter-workflow-implement`, `$butter-workflow-code-review`,
`$butter-workflow-finish`).

### Skills

| Skill | Role |
|---|---|
| `butter-workflow-start` | Starts a workflow from the context the user provides, classifies track type, plans the working branch name, writes spec docs for every track, and summarizes the written files as hyperlinks before pausing for approval. |
| `butter-workflow-implement` | Creates or switches to the working branch, implements the approved spec for any track, verifies changes, commits, pushes, and creates a PR. |
| `butter-workflow-code-review` | Reviews a PR or branch diff with issue, plan, task, and risk-target context. |
| `butter-workflow-finish` | Captures reusable user preferences after a workflow. |
| `butter-workflow-user-preferences` | Provides the shared preference data model and bootstrap template. |

## Track Model

- Track A: small low-risk changes; writes a lightweight spec (`00-META.md` + `01-SPEC.md`) and pauses for approval before implementing.
- Track B: planned feature or fix within existing architecture; creates the full `docs/specs/{TASK-ID}/` set.
- Track C: high-risk Track B work involving auth, security, payment, permissions, shared core, architecture, migration, or broad refactor risk; adds `Risk Review Targets`.

## Tool Rules

- Use available Jira/GitHub MCP tools first for issues, PRs, and review comments.
- Use `gh` only for GitHub PR work when MCP is unavailable.
- Use `git` for repository work: branch, status, diff, commit, and push.
- Do not modify issue bodies or comments without user approval.

## Repository Outputs

Every track creates a spec directory. Track A writes a lightweight spec:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
```

Track B/C add the plan, task, and preference files:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
  02-PLAN.md
  03-TASK-*.md
  04-PREFERENCES.md
```

`00-META.md` is the current workflow state source of truth. `04-PREFERENCES.md` is only an input for finish-stage preference capture.
