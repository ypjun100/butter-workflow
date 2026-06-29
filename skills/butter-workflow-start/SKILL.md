---
name: butter-workflow-start
description: Start an issue-based Spec-Driven Development workflow. Use when the user provides a Jira or GitHub issue, task link, or implementation context and wants Codex to classify Track A/B/C, create a working branch, bootstrap user-preferences, and either implement a small Track A task or create docs/specs planning files for Track B/C.
---

# Butter Workflow Start

Start an issue-based workflow and leave enough repository state for another tool or session to continue.

## Inputs

- Issue URL, issue key, GitHub issue number, or pasted issue context.
- Optional user constraints or implementation notes.

## Workflow

1. Read repository instructions first: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or equivalent files when present.
2. Bootstrap global `user-preferences` if missing:
   - Source template: `skills/user-preferences/`.
   - Runtime skill locations:
     - `~/.claude/skills/user-preferences/SKILL.md`
     - `~/.agents/skills/user-preferences/SKILL.md`
   - Shared data location: `~/.agents/preferences/`.
   - Create missing `active.md`, `candidates.md`, and `rejected.md` from the template files under `skills/user-preferences/references/`.
   - Preserve existing preference data.
3. Read `~/.agents/preferences/active.md` when it exists. Do not read `candidates.md` during start.
4. Collect issue context:
   - Prefer available Jira/GitHub MCP tools for issue title, body, comments, labels, and linked resources.
   - If MCP is unavailable, try the user-provided URL when accessible.
   - If neither works, ask for the issue body and relevant comments.
5. Collect project context:
   - Current branch and clean/dirty working tree.
   - Existing branch naming convention with `git branch --list`.
   - Build/test/package files relevant to the repository.
6. Set base branch to the branch active at workflow start.
7. Decide working branch:
   - Classify work type as `feature`, `fix`, `refactor`, or `docs`.
   - Derive task id from the issue key, GitHub issue number as `gh-N`, or a short slug when no id exists.
   - Follow existing branch prefix conventions when clear; otherwise use `feature/<task-id>-<slug>`, `fix/<task-id>-<slug>`, `refactor/<task-id>-<slug>`, or `docs/<task-id>-<slug>`.
8. Classify Track Type:
   - Track A: small, low-risk, few files, no API/DB/auth/security/shared-module impact, rollback is easy.
   - Track B: needs planning docs, may touch multiple files, fits existing architecture, requires tests or type checks.
   - Track C: Track B plus auth, security, payment, permission, shared-core, architecture, migration, or broad refactor risk.
9. Confirm the track with the user when the classification is ambiguous or when Track C is selected. For obvious Track A/B, proceed and state the assumption.
10. Create the working branch once with `git switch -c <branch>`.
11. Execute by track:
    - Track A: implement the change, run targeted verification, commit, push, and create a PR when possible. Then recommend `butter-workflow-code-review`.
    - Track B/C: create `docs/specs/{TASK-ID}/` and write `00-META.md`, `01-SPEC.md`, `02-PLAN.md`, one or more `03-TASK-*.md`, and `04-PREFERENCES.md`.
12. For Track C, add `## Risk Review Targets` to `02-PLAN.md` and self-check whether each high-risk scope has review focus.
13. Stop Track B/C in feedback mode after planning. Give the user the docs path and wait for approval before implementation.

## Document Rules

- `00-META.md` is the Track B/C state source of truth.
- `01-SPEC.md` describes what users need and success criteria. Do not include function names, file names, or implementation choices.
- `02-PLAN.md` describes architecture boundaries, module responsibilities, call flow, data/API shape, test strategy, constraints, and task split.
- `03-TASK-*.md` files must be implementable units and should align with commit-sized changes.
- `04-PREFERENCES.md` records user feedback and preference candidates for finish only. Do not store workflow status there.
- Keep verification logs out of files. Summarize verification in the PR body or final response.

## Git And External Tools

- Use `git` for branch creation, status, diff, commit, and push.
- Use `gh` only for GitHub PR creation, PR lookup, and review comment lookup when no MCP tool is available and `gh` is installed/authenticated.
- Do not modify issue bodies or comments unless the user approves.

## Track B/C File Templates

`00-META.md`:

```markdown
# Meta

- Issue URL:
- Track Type: B
- Base branch:
- Working branch:
- PR URL:
- Status: planned
```

`04-PREFERENCES.md`:

```markdown
# Preferences

## User Feedback
- None yet.

## Preference Candidates
- None yet.

## Rejected Candidates
- None yet.
```
