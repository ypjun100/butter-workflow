---
name: user-preferences
user-invocable: false
description: Shared user preference memory for Butter Workflow. Use when a Butter Workflow stage tells Codex to read active preferences or finish a workflow by updating active, candidate, and rejected preference files stored under ~/.agents/preferences.
---

# User Preferences

Read and maintain reusable user development preferences for Butter Workflow.

## Data Location

Use `~/.agents/preferences/` as the single source of truth:

- `active.md`: confirmed preferences that should affect planning, implementation, and review.
- `candidates.md`: one-time observations waiting for more evidence. Read only during finish.
- `rejected.md`: observations that should not be generalized.

## Runtime Rules

- During start, implement, and code-review, read only `active.md`.
- During finish, read all three files.
- Do not store workflow state, issue status, PR status, verification logs, or project-specific temporary decisions here.
- Prefer short bullets grouped by category such as Planning, Architecture, Naming, Testing, Implementation, and Review And PR.
- Promote a candidate to active only when the same preference is observed again or the user explicitly says to apply it in the future.
- Move or copy an observation to rejected when it is task-specific, contradicted by project rules, contradicted by active preferences, or explicitly rejected by the user.

## Bootstrap

If the shared data directory is missing, create it from this skill's templates:

- `references/active.template.md`
- `references/candidates.template.md`
- `references/rejected.template.md`

Install this skill body into both tool skill locations when missing:

- `~/.claude/skills/user-preferences/SKILL.md`
- `~/.agents/skills/user-preferences/SKILL.md`

Preserve existing preference data during bootstrap or template upgrades.
