# Issue-Based Agentic Development Workflow Proposal

This document is the repository copy of the workflow proposal behind Butter
Workflow. It describes the method, shared artifacts, and tool handoff model that
the Claude Code and Codex plugin implementations follow.

## Goal

Butter Workflow makes agentic development portable across tools and sessions by
using repository files as the source of truth. A task can start in Claude Code,
continue in Codex, and return for implementation without relying on chat
history, while user preferences accumulate in a shared user-level store.

The workflow is issue-based and Spec-Driven. Every non-trivial task starts from
an issue, link, or clear task context, then follows a track selected by risk and
planning depth.

## Core Principles

- Keep task state in the repository, not in one chat thread.
- Use the same workflow stages in Claude Code and Codex.
- Keep tool-specific commands thin; put reusable behavior in shared skills.
- Dispatch independent research together rather than in sequence, so a plan's
  depth is not paid for in waiting.
- Judge a finished plan with an agent that did not write it.
- Use available Jira, GitHub, or similar MCP tools first when they exist.
- Use `git` for repository work such as branch, status, diff, commit, and push.
- Use `gh` only for GitHub PR work when MCP tools are unavailable.
- Avoid bundling external service credentials or MCP setup into the plugin.
- Capture user preferences continuously, at every stage, and keep the bar low
  enough that anything useful to the next task gets stored. Project rules stay
  in project files such as `AGENTS.md`.

## Track Model

### Track A: Small Low-Risk Change

Track A is for small, well-scoped changes. It writes a lightweight spec, pauses
for user approval, and then implements from that spec through verification,
commit, push, and PR creation.

Track A creates:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
```

Use Track A when:

- The behavior is obvious and localized.
- There is no architecture, migration, auth, permission, payment, or shared-core
  risk.
- The expected diff is small enough to review directly.

### Track B: Planned Change

Track B is for feature or fix work that benefits from a written spec and task
plan but stays within the existing architecture.

Track B creates:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
  02-PLAN.md
  03-TASK-*.md
```

Use Track B when:

- The work spans several files or steps.
- The implementation should be checked against a written plan.
- The change is meaningful but not high-risk.

### Track C: High-Risk Planned Change

Track C uses the same document structure as Track B and adds explicit risk
review targets. It is for work where a second pass in another tool or model is
strongly recommended.

Use Track C when the work touches:

- Authentication or authorization.
- Permissions, payments, billing, or account boundaries.
- Security-sensitive flows or secrets.
- Shared core modules, public contracts, or broad refactors.
- Data migrations or irreversible state changes.
- Architecture-level decisions with broad blast radius.

## Invocation Model

The three stages are not entered the same way.

Implement and code review are entered from repository state that proves a
workflow is already running: a spec directory exists, `00-META.md` carries a
status, a PR is open. That state is checkable, so these stages can be inferred
from plain language.

Start has no such state. Before it runs, nothing in the repository says a
workflow is beginning, so the only available signal is phrasing — and "start
this task" is indistinguishable from an ordinary request to go do the work. The
user therefore declares the start explicitly by naming the skill.

Routing resolves in this order:

1. The user names the start skill or Butter Workflow — run the start stage.
2. The state for a later stage exists and the request asks for that stage's
   work — run that stage. Either condition alone is not enough.
3. Otherwise — handle the request directly. This is the default, and it stays
   the default inside a running workflow: editing a spec document or applying a
   named fix is an ordinary request, not a stage transition.

## Workflow Stages

### 1. Start

The start stage receives an issue URL, issue key, PR context, or task summary.
It gathers context, identifies the base branch, records the intended working
branch name, and classifies the work as Track A, B, or C. It does not create or
check out the branch; the implement stage does that.

Start writes the spec for the chosen track and then stops for user approval
before implementation. For Track A it writes a lightweight spec (`00-META.md`
and `01-SPEC.md`). For Track B or C it also writes the plan and task files.

#### Planning Research

A Track B or Track C plan rests on several readings of the repository: what
already does this work and who calls it, how the repository writes this kind of
code, what the change touches and what depends on it, how the result gets
verified, and what the referenced issue says. None of those readings needs
another one's result, so they go out together and the wait is the slowest of
them rather than their sum.

Depth is why this matters more than speed does. When every reading is paid for
in waiting, research gets cut short, and a plan built on shallow research
proposes a new function beside one that already does the job and defines its
own naming and placement next to the repository's own.

The spec is written while the research runs. It carries user needs and success
criteria and no implementation choices, so it has nothing to wait for.

Each research agent is read-only, returns findings rather than proposals,
anchors every item to a location in the repository, and states when a search
came up empty. Silence and "nothing found" are different answers.

Research that surfaces risk the classification missed moves the track from B to
C, and the user is told when it moves.

#### Plan Validation

A finished Track B or Track C plan is checked by an agent that did not write it,
before the plan reaches the user and before the task files are split out of it.
The reasoning matches preference capture: an agent in the middle of producing
something deprioritizes any judgement attached to it.

The check covers reuse, minimal change, architecture fit, and convention, and
for Track C whether every high-risk scope has a matching risk review target.
Each finding names the point in the plan, the repository location that backs it,
and what the plan should say instead. A verdict with no findings has to state
what was searched for and not found, so an empty pass cannot stand in for a
search.

Findings are applied and the plan is re-checked once. One round rather than a
loop, because a gate that can run indefinitely is a gate that stalls the stage.
Findings still open after that round, and findings the plan author disagrees
with, stay unapplied and are reported with their reason when the stage pauses.

The task files are written after the plan settles, so applying a finding does
not mean rewriting the split. The validation record is not written to the spec
directory; it appears in the handoff summary, one line per finding, alongside
the spec file list.

Track A writes no plan and skips both the research fan-out and this gate.

### 2. Implement

The implement stage resumes from the spec: `00-META.md` and `01-SPEC.md` for
every track, plus `02-PLAN.md` and the `03-TASK-*.md` files for Track B/C. It
creates and checks out the working branch from the base branch, then implements
one task at a time, verifies the result, commits completed work, pushes the
branch, and creates or updates the PR.

The implementation stage should keep commits aligned with task boundaries when
practical.

### 3. Code Review

The code-review stage can run in the same tool or in a different tool. It
collects the relevant diff, issue context, plan, task documents, and Track C
risk targets.

The review focuses on correctness, regressions, missing tests, security,
boundary mistakes, and mismatches between the plan and implementation. Feedback
is either applied directly when low-risk or returned to the user when judgment
is needed.

Code review is the last stage. It sets `Status: reviewed`, which is the
terminal state of the workflow.

## Handoff Model

Butter Workflow treats repository files as the handoff boundary.

The main handoff artifact is:

```text
docs/specs/{TASK-ID}/00-META.md
```

It records the track, branch, PR, current status, and next expected
stage. The rest of the spec directory records the plan and task details.

Because the state is in files, a user can switch tools or sessions without
reconstructing the task from memory.

## User Preferences Model

The plugin separates workflow logic from user preference data.

Workflow logic lives in the installed plugin. Preference data lives in a shared
user-level location:

```text
~/.agents/preferences/
  preferences.md
```

`preferences.md` holds everything captured so far, grouped under a fixed set of
category headings. Every stage reads it and applies it.

The bootstrap template is bundled as a resource of `butter-workflow-start`, so
no internal helper skill is exposed to users. Bootstrap creates the file when it
is absent and otherwise leaves it alone.

### What Gets Captured

The bar is anything that would help write a better plan, change, or review on
the next task. It is deliberately wider than what a user would call a
"preference".

Feedback arrives attached to an instance. Users say "this component is too
complex, simplify it", not "I prefer simple code". The generality sits in the
rule behind the request rather than in how the request is phrased, so the rule
is what gets extracted and stored. Uncertainty resolves toward recording, since
a wrong entry takes one line to delete while a missing one is invisible.

Four things are skipped: an equivalent entry already stored, a rule already
stated in the project instruction files, a pure one-off with no rule behind it,
and a message with no content such as a bare approval.

Equivalence is judged by meaning rather than by exact wording, so the same rule
phrased two different ways is treated as one entry.

### Capture Timing

Capture runs throughout the workflow. Every stage evaluates the user messages it
receives — including the ones that arrive after the stage's main work is done,
such as spec feedback and follow-up fix requests.

A recorded preference applies from the next task onward.

### Delegated Judgement

The stage does not judge its own messages. Any message carrying content is
handed verbatim to a separate capture agent, which runs alongside the work the
user actually asked for. The stage decides only whether a message has content,
never whether it is worth recording.

The split matters because an agent in the middle of the user's actual request
will deprioritize any judgement attached to it. Handing the judgement to an
agent that has nothing else to do is what makes capture reliable. It also keeps
the cost flat when one message carries dozens of separate points, and keeps the
reasoning out of the main context.

Routing does not gate capture. A message handled as a spec edit or a code fix
still reaches the capture agent.

Tools without a subagent mechanism run the same contract inline at the end of
the response. Criteria, writing rules, and reporting are identical; only the
place it runs differs.

### Conflicts And Removal

A new entry that contradicts a stored one replaces it, on the grounds that the
user's latest instruction is their current one. The replacement is reported
rather than applied silently. Asking instead is not an option available to a
background agent.

A user asking for an entry to be dropped removes it. Removal is not permanent: a
later message making the same point can put it back.

### Reporting

Each record, replacement, or removal is reported in one line at the end of the
response that triggered it. Capture never asks for approval first, which keeps
it out of the way while still letting the user correct a wrong capture
immediately.

## Recommended Cross-Tool Flow

One recommended flow is:

```text
Claude Code start
Claude Code implement
Codex code-review
```

This is only a recommendation. Any stage can run in either tool as long as the
stage reads the repository state first and writes the updated state before
ending.

## External Tool Rules

- Use available Jira or GitHub MCP tools first for issues, PRs, and review
  comments.
- Use `gh` only for GitHub PR creation, PR lookup, or review-comment lookup
  when MCP tools are unavailable.
- Use `git` for branch creation, status checks, diffs, commits, and pushes.
- Do not modify issue bodies or comments without explicit user approval.

## Repository Outputs

Every track creates a spec directory. Track A writes a lightweight spec:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
```

Track B and Track C add the plan and task files:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
  02-PLAN.md
  03-TASK-*.md
```

`00-META.md` is the current workflow state source of truth. Preference data is
never written into the spec directory; it lives in
`~/.agents/preferences/preferences.md`.
