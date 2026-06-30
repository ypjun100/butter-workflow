# Repository Instructions

This repository stores portable Butter Workflow plugin configuration, commands,
and user-authored skills for Claude Code and Codex.

## Instruction Source Of Truth

- Keep repository-wide agent instructions in this file.
- Keep `CLAUDE.md` as a thin Claude Code entrypoint that points back to this
  file.
- Add tool-specific behavior only when it cannot be expressed as a shared
  repository rule.

## Project Scope

- Keep Claude Code commands and Codex entrypoints thin.
- Put reusable workflow behavior in shared skills under `skills/`.
- Use `README.md` for user-facing installation and usage.
- Use `docs/workflow-proposal.md` for workflow design and methodology.
- Do not store credentials, tokens, machine-local secrets, or private
  environment state in this repository.

## Skill Documentation

When creating, renaming, deleting, or materially changing a user-authored skill
under `skills/`, update the `README.md` skills table in the same change.

For each skill row:

- Use the skill directory name in backticks.
- Keep the role description brief and practical.
- Describe what the skill helps Codex do, not implementation details.
- Do not add bundled/system skills from `skills/.system/`.

Before committing skill changes, verify that the `README.md` table matches the
current user-authored skill directories.
