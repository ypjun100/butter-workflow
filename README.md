<p align="center">
  <img src="assets/butter-workflow-banner.png" alt="Butter Workflow banner" width="100%">
</p>

<h1 align="center">Butter Workflow</h1>

<p align="center">
  Portable issue-based Spec-Driven Development for AI coding agents that support Agent Skills.
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
  <a href="#track-model">Track Model</a> ·
  <a href="#planning">Planning</a> ·
  <a href="#preference-capture">Preference Capture</a>
</p>

Butter Workflow keeps the working context in repository files so a task can
move between tools, models, and sessions without depending on chat history.

See [Workflow Design](docs/workflow-proposal.md) for the full methodology and
handoff model.

## What It Provides

- Three shared workflow stages: start, implement, and code-review.
- One shared skill set (`skills/`) for agents supporting the Agent Skills
  format (`SKILL.md`) — no tool-specific command wrappers.
- Track A/B/C routing for small changes, planned changes, and high-risk changes.
- Parallel repository research behind every plan, and a separate validation
  pass before a plan reaches you.
- Handoff spec documents under `docs/specs/{TASK-ID}/` for every track.
- Shared user preference memory under `~/.agents/preferences/preferences.md`,
  captured automatically as you work.

## Installation

Install directly from this GitHub repository with the
[skills.sh](https://www.skills.sh/) CLI. No clone required:

```bash
npx skills add ypjun100/butter-workflow -g
```

The `-g` flag makes the workflow available across projects. Update the global
installation to the latest version later with:

```bash
npx skills update -g
```

The CLI supports Claude Code, Codex, and the other agents in its supported-agent
list. Agent-specific skill features may vary. Trade-off to know: unlike a
plugin marketplace, this installation path has no version pinning or changelog
UI — it installs whatever is on the default branch, so check this repository's
commit history before updating if that matters to you. Restart the tool or
start a new session after installing so the newly loaded skills are available.

## Usage

Start a workflow by naming the start skill. This is the only stage that
requires it:

```text
/butter-workflow-start <task context>          # Claude Code and similar
$butter-workflow-start <task context>          # Codex
Use the butter-workflow-start skill with <task context>
```

The remaining stages run from plain language:

```text
Looks good, start implementing.   ->  butter-workflow-implement
Review this PR: <PR URL>          ->  butter-workflow-code-review
```

Ordinary requests inside a running workflow stay ordinary. "Fix this line in
the spec" edits the spec document; "apply this feedback" applies it, commits,
and pushes. Neither advances a stage — only asking for the next stage's work
does.

### Skills

| Skill | Role |
|---|---|
| `butter-workflow-start` | Starts a workflow when explicitly named, from the context the user provides: classifies track type, plans the working branch name, researches the repository in parallel, writes spec docs for every track, validates the plan with a separate agent, and summarizes the written files as hyperlinks before pausing for approval. |
| `butter-workflow-implement` | Creates or switches to the working branch, implements the approved spec for any track, verifies changes, commits, pushes, and creates a PR. |
| `butter-workflow-code-review` | Reviews a PR or branch diff with issue, plan, task, and risk-target context, then closes out the workflow. |

All three stages also capture preferences while they run. See
[Preference Capture](#preference-capture).

## Track Model

- Track A: small low-risk changes; writes a lightweight spec (`00-META.md` + `01-SPEC.md`) and pauses for approval before implementing.
- Track B: planned feature or fix within existing architecture; creates the full `docs/specs/{TASK-ID}/` set.
- Track C: high-risk Track B work involving auth, security, payment, permissions, shared core, architecture, migration, or broad refactor risk; adds `Risk Review Targets`.

## Planning

For Track B and Track C, the start stage reads the repository from five angles
at once rather than one after another:

```text
reuse         what already does this, and who calls it
convention    how this repository writes this kind of code
impact        what the change touches, and what depends on it
verification  build, test, and lint commands, and where tests live
external      the referenced issue, PR, or ticket
```

The spec is written while they run, since it carries what you need rather than
how it gets built. Nothing waits on a result it does not use. Each angle
reports findings anchored to real locations in the repository, and a search
that came up empty is stated rather than left silent.

The finished plan then goes to an agent that did not write it, checking reuse,
minimal change, architecture fit, and convention:

```text
"add a date formatter"  ->  src/lib/date.ts:40 already has one
"add a config layer"    ->  one caller; inline it
"put the client here"   ->  every other client lives in src/api/
"add a tests/ folder"   ->  tests sit beside the source in this repo
```

A clean verdict has to say what it searched for and did not find; an empty pass
does not count. Findings get applied and the plan is re-checked once rather
than looped, so the gate cannot stall. Anything left unapplied is reported with
its reason when the stage pauses for your approval.

## Preference Capture

Every stage watches for preferences in what you say and records them as it
goes, so nothing depends on remembering to run a wrap-up step.

```text
~/.agents/preferences/
  preferences.md  # everything captured, applied to your work from the next task on
```

A request that contradicts an entry already in the file replaces it, since your
latest word is the current one — the swap is reported so you can put it back.
Asking for an entry to be dropped removes it.

The bar is low on purpose. Anything that would help write a better plan,
change, or review next time gets captured, and the rule is extracted from
whatever you happened to be talking about at the time:

```text
"Write the plan in Korean, not English"
  -> plan documents are written in Korean from now on

"This component's logic is too complicated, unpack it"
  -> favor simple, direct code over defensive handling for unlikely cases
```

Neither of those is phrased as a preference, and both name something specific.
Both are captured, because the rule is what matters and the rule generalizes.

The judging runs in a separate agent alongside your actual request, so a
message carrying thirty pieces of feedback does not turn into thirty pieces of
deliberation you have to sit through. Each record, replacement, or removal is
reported in a single line at the end of the response, so you can correct a bad
capture right away.

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

Track B/C add the plan and task files:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
  02-PLAN.md
  03-TASK-*.md
```

`00-META.md` is the current workflow state source of truth. Its `Status` moves
`planned` → `implemented` → `reviewed`, ending at `reviewed`. Preference data
never lives in the spec directory; it lives in
`~/.agents/preferences/preferences.md`.
