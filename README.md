<p align="center">
  <img src="assets/butter-workflow-banner.png" alt="Butter Workflow banner" width="100%">
</p>

<h1 align="center">Butter Workflow</h1>

<p align="center">
  Portable issue-based Spec-Driven Development for Claude Code and Codex.
</p>

<p align="center">
  <img alt="Claude Code plugin" src="https://img.shields.io/badge/Claude%20Code-plugin-111827?style=flat-square">
  <img alt="Codex plugin" src="https://img.shields.io/badge/Codex-plugin-10A37F?style=flat-square">
  <img alt="Spec-driven workflow" src="https://img.shields.io/badge/workflow-spec--driven-F59E0B?style=flat-square">
</p>

<p align="center">
  <a href="#what-it-provides">What It Provides</a> ·
  <a href="docs/workflow-proposal.md">Workflow Design</a> ·
  <a href="#installation">Installation</a> ·
  <a href="#claude-usage">Claude Usage</a> ·
  <a href="#codex-usage">Codex Usage</a> ·
  <a href="#track-model">Track Model</a>
</p>

Butter Workflow keeps the working context in repository files so a task can
move between tools, models, and sessions without depending on chat history.

See [Workflow Design](docs/workflow-proposal.md) for the full methodology and
handoff model.

## What It Provides

- Four shared workflow stages: start, implement, code-review, and finish.
- Claude Code commands as thin wrappers over the same shared skill logic.
- Codex skills loaded from the same `skills/` directory.
- Track A/B/C routing for small changes, planned changes, and high-risk changes.
- Track B/C handoff documents under `docs/specs/{TASK-ID}/`.
- Shared user preference memory under `~/.agents/preferences/`.

## Installation

Install Butter Workflow from the GitHub marketplace repository. You do not need
to clone this repository for normal installation.

### Claude Code

Add the GitHub repository as a Claude Code marketplace, then install the plugin:

```bash
claude plugin marketplace add ypjun100/butter-workflow --scope user
claude plugin install butter-workflow@butter-workflow --scope user
```

Use `--scope project` instead of `--scope user` when the marketplace and plugin
should be registered only for the current project.

### Codex

Add the GitHub repository as a Codex marketplace, then install the plugin:

```bash
codex plugin marketplace add ypjun100/butter-workflow
codex plugin add butter-workflow@butter-workflow
```

Restart the tool or start a new session after installing so newly loaded commands and skills are available.

## Claude Usage

Use the command wrappers:

```text
/butter-workflow:start <issue URL or context>
/butter-workflow:implement
/butter-workflow:code-review <PR URL>
/butter-workflow:finish
```

## Codex Usage

Use the skills directly:

```text
Use $butter-workflow-start with <issue URL or context>
Use $butter-workflow-implement
Use $butter-workflow-code-review with <PR URL>
Use $butter-workflow-finish
```

### Skills

| Skill | Role |
|---|---|
| `butter-workflow-start` | Starts a workflow from user-provided context (pasted notes or an issue link), classifies track type, creates the working branch, and creates planning docs when needed. |
| `butter-workflow-implement` | Implements approved Track B/C task docs, verifies changes, commits, pushes, and creates a PR. |
| `butter-workflow-code-review` | Reviews a PR or branch diff with issue, plan, task, and risk-target context. |
| `butter-workflow-finish` | Captures reusable user preferences after a workflow. |
| `user-preferences` | Provides the shared preference data model and bootstrap template. |

## Track Model

- Track A: small low-risk changes; no planning docs.
- Track B: planned feature or fix within existing architecture; creates `docs/specs/{TASK-ID}/`.
- Track C: high-risk Track B work involving auth, security, payment, permissions, shared core, architecture, migration, or broad refactor risk; adds `Risk Review Targets`.

## Tool Rules

- Use available Jira/GitHub MCP tools first for issues, PRs, and review comments.
- Use `gh` only for GitHub PR work when MCP is unavailable.
- Use `git` for repository work: branch, status, diff, commit, and push.
- Do not modify issue bodies or comments without user approval.

## Repository Outputs

Track B/C workflows create:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
  02-PLAN.md
  03-TASK-*.md
  04-PREFERENCES.md
```

`00-META.md` is the current workflow state source of truth. `04-PREFERENCES.md` is only an input for finish-stage preference capture.
