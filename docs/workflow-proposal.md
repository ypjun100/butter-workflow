# Issue-Based Agentic Development Workflow Proposal

This document is the repository copy of the workflow proposal behind Butter
Workflow. It describes the method, shared artifacts, and tool handoff model that
the Claude Code and Codex plugin implementations follow.

## Goal

Butter Workflow makes agentic development portable across tools and sessions by
using repository files as the source of truth. A task can start in Claude Code,
continue in Codex, return for implementation, and finish with reusable
preferences captured without relying on chat history.

The workflow is issue-based and Spec-Driven. Every non-trivial task starts from
an issue, link, or clear task context, then follows a track selected by risk and
planning depth.

## Core Principles

- Keep task state in the repository, not in one chat thread.
- Use the same workflow stages in Claude Code and Codex.
- Keep tool-specific commands thin; put reusable behavior in shared skills.
- Use available Jira, GitHub, or similar MCP tools first when they exist.
- Use `git` for repository work such as branch, status, diff, commit, and push.
- Use `gh` only for GitHub PR work when MCP tools are unavailable.
- Avoid bundling external service credentials or MCP setup into the plugin.
- Capture reusable user preferences after a workflow, but keep project rules in
  project files such as `AGENTS.md`.

## Track Model

### Track A: Small Low-Risk Change

Track A is for small, well-scoped changes that do not need planning documents.
The workflow can move from start directly through implementation, verification,
commit, push, and PR creation.

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
  04-PREFERENCES.md
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

## Workflow Stages

### 1. Start

The start stage receives an issue URL, issue key, PR context, or task summary.
It gathers context, identifies the base branch, creates the working branch, and
classifies the work as Track A, B, or C.

For Track A, start may continue through implementation and PR creation when the
change is small enough.

For Track B or C, start creates the spec directory and writes the initial state,
spec, plan, task files, and preference capture notes.

### 2. Implement

The implement stage resumes from `00-META.md`, `02-PLAN.md`, and the
`03-TASK-*.md` files. It implements one task at a time, verifies the result,
commits completed work, pushes the branch, and creates or updates the PR.

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

### 4. Finish

The finish stage closes the workflow by collecting reusable observations. For
Track B and C, it reads `04-PREFERENCES.md` along with relevant review and
implementation context.

Stable preferences are promoted into the user's shared preference store.
One-off observations stay as candidates, and rejected generalizations are
recorded to avoid repeating them.

## Handoff Model

Butter Workflow treats repository files as the handoff boundary.

The main handoff artifact is:

```text
docs/specs/{TASK-ID}/00-META.md
```

It records the issue, track, branch, PR, current status, and next expected
stage. The rest of the spec directory records the plan and task details.

Because the state is in files, a user can switch tools or sessions without
reconstructing the task from memory.

## User Preferences Model

The plugin separates workflow logic from user preference data.

Workflow logic lives in the installed plugin. Preference data lives in a shared
user-level location:

```text
~/.agents/preferences/
  active.md
  candidates.md
  rejected.md
```

`active.md` contains confirmed preferences that should influence future work.
`candidates.md` stores observations that need more evidence before promotion.
`rejected.md` records patterns that should not be generalized.

The `user-preferences` skill provides the data model and templates used by the
workflow.

## Recommended Cross-Tool Flow

One recommended flow is:

```text
Claude Code start
Claude Code implement
Codex code-review
Claude Code finish
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

Track B and Track C workflows create:

```text
docs/specs/{TASK-ID}/
  00-META.md
  01-SPEC.md
  02-PLAN.md
  03-TASK-*.md
  04-PREFERENCES.md
```

`00-META.md` is the current workflow state source of truth.
`04-PREFERENCES.md` is only an input for finish-stage preference capture.
