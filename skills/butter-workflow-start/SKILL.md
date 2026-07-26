---
name: butter-workflow-start
description: |-
  Trigger: the user wants to begin, kick off, or scope a new piece of work (e.g. "start this task", "let's plan this out", references a new issue/ticket) with no existing spec for it yet. Classifies Track A/B/C, plans the working branch, bootstraps user-preferences, and writes docs/specs/{TASK-ID} spec files for every track before pausing for approval.
  Skip: an approved docs/specs/{TASK-ID}/00-META.md already exists for the current task (Status is planned or later) — use `butter-workflow-implement` instead to continue that work.
---

# Butter Workflow Start

Start a workflow from the context the user provides and leave enough repository state for another tool or session to continue.

## Inputs

- The context the user provides about the task, in whatever form they give it.

## Workflow

1. Read repository instructions first: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or equivalent files when present.
2. Bootstrap global `butter-workflow-user-preferences` if missing:
   - Source template: `skills/butter-workflow-user-preferences/`.
   - Runtime skill locations:
     - `~/.claude/skills/butter-workflow-user-preferences/SKILL.md`
     - `~/.agents/skills/butter-workflow-user-preferences/SKILL.md`
   - Shared data location: `~/.agents/preferences/`.
   - Create missing `active.md`, `candidates.md`, and `rejected.md` from the template files under `skills/butter-workflow-user-preferences/references/`.
   - Preserve existing preference data.
3. Read `~/.agents/preferences/active.md` when it exists. Do not read `candidates.md` during start.
4. Understand the task from the provided context. Use it as-is and do not ask for any particular input form. When the context references retrievable external resources, enrich your understanding with available Jira/GitHub MCP tools, or with `gh` for GitHub when MCP is unavailable.
5. Collect project context:
   - Current branch and clean/dirty working tree.
   - Existing branch naming convention with `git branch --list`.
   - Build/test/package files relevant to the repository.
6. Set base branch to the branch active at workflow start.
7. Plan the working branch name (do not create it):
   - Classify work type as `feature`, `fix`, `refactor`, or `docs`.
   - Derive task id from the issue key, GitHub issue number as `gh-N`, or a short slug when no id exists.
   - Follow existing branch prefix conventions when clear; otherwise use `feature/<task-id>-<slug>`, `fix/<task-id>-<slug>`, `refactor/<task-id>-<slug>`, or `docs/<task-id>-<slug>`.
   - Record the base branch and this working branch name in `00-META.md`. The implement stage creates and checks out the branch.
8. Classify Track Type:
   - Track A: small, low-risk, few files, no API/DB/auth/security/shared-module impact, rollback is easy.
   - Track B: needs planning docs, may touch multiple files, fits existing architecture, requires tests or type checks.
   - Track C: Track B plus auth, security, payment, permission, shared-core, architecture, migration, or broad refactor risk.
9. Confirm the track with the user when the classification is ambiguous or when Track C is selected. For obvious Track A/B, proceed and state the assumption.
10. Do not create or check out any branch during start. The working branch is only recorded as the planned value in `00-META.md`; the implement stage creates and checks it out.
11. Write the spec by track. Do not implement, commit, push, or open a PR during start for any track.
    - Track A: create `docs/specs/{TASK-ID}/` and write only `00-META.md` and `01-SPEC.md` (lightweight spec).
    - Track B/C: create `docs/specs/{TASK-ID}/` and write `00-META.md`, `01-SPEC.md`, `02-PLAN.md`, one or more `03-TASK-*.md`, and `04-PREFERENCES.md`.
12. For Track C, add `## Risk Review Targets` to `02-PLAN.md` and self-check whether each high-risk scope has review focus.
13. Stop in feedback mode after writing the spec for every track (A/B/C):
    - Give the user the spec directory path.
    - Summarize what was written: list every spec file created in this run, each rendered as a clickable Markdown link to the file (for example, `[00-META.md](/absolute/path/to/docs/specs/{TASK-ID}/00-META.md)`), with a short note of what each covers. List only files actually written for the track (Track A: `00-META.md`, `01-SPEC.md`; Track B/C: the full set including every `03-TASK-*.md` written). Never list a file that was not written.
    - Wait for approval before implementation.

## Document Rules

- `00-META.md` is the workflow state source of truth for every track.
- Track A writes only `00-META.md` and `01-SPEC.md`. `02-PLAN.md`, `03-TASK-*.md`, and `04-PREFERENCES.md` are Track B/C only.
- `01-SPEC.md` describes what users need and success criteria. Do not include function names, file names, or implementation choices.
- `02-PLAN.md` describes architecture boundaries, module responsibilities, call flow, data/API shape, test strategy, constraints, and task split.
- `03-TASK-*.md` files must be implementable units and should align with commit-sized changes.
- `04-PREFERENCES.md` records user feedback and preference candidates for finish only. Do not store workflow status there.
- Keep verification logs out of files. Summarize verification in the PR body or final response.

## Git And External Tools

- Use `git` only for read-only checks (status, diff, branch list) during start. Do not create branches, commit, or push.
- Use `gh` only for GitHub PR creation, PR lookup, and review comment lookup when no MCP tool is available and `gh` is installed/authenticated.
- Do not modify issue bodies or comments unless the user approves.

## File Templates

`00-META.md` (all tracks; set `Track Type` to A, B, or C):

```markdown
# Meta

- Track Type: A
- Base branch:
- Working branch:
- PR URL:
- Status: planned
```

`04-PREFERENCES.md` (Track B/C only):

```markdown
# Preferences

## User Feedback
- None yet.

## Preference Candidates
- None yet.

## Rejected Candidates
- None yet.
```
