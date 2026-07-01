---
name: butter-workflow-implement
description: Implement an approved Butter Workflow plan. Use after Track B/C planning docs exist and the user approves implementation, or when resuming implementation from docs/specs/{TASK-ID}; reads workflow state, applies tasks, runs verification, commits task-sized changes, pushes, and creates or updates a PR.
---

# Butter Workflow Implement

Implement the approved plan while preserving the docs as the handoff surface.

## Workflow

1. Read repository instructions first.
2. Read `~/.agents/preferences/active.md` when it exists. Do not read `candidates.md`.
3. Find the active task docs:
   - Prefer the task id or path supplied by the user.
   - Otherwise inspect `docs/specs/*/00-META.md` and choose the one whose status is `planned` or whose working branch matches the current branch.
   - Ask the user only when multiple plausible active specs exist.
4. Read the spec for the task's track:
   - Always: `00-META.md` and `01-SPEC.md`.
   - Track B/C also: `02-PLAN.md` and relevant `03-TASK-*.md`.
   - Track C also: `02-PLAN.md` `Risk Review Targets`.
   - Track A implements from `00-META.md` and `01-SPEC.md` only.
5. Verify that the current branch matches `Working branch`. If not, switch to it or ask before continuing when switching would be risky.
6. Implement only the current task scope. Do not expand beyond the spec without updating it or asking the user when scope/risk changes.
7. Run targeted verification after each meaningful task:
   - Use repo-native commands from package scripts, build files, test config, or existing documentation.
   - Prefer focused tests first, then broader checks if shared behavior changed.
8. Commit task-sized changes:
   - Follow the repository's existing commit style.
   - Use `git` for status, diff, add, commit, and push.
   - It is acceptable to skip a commit for investigation-only tasks with no file changes.
9. After all tasks are complete:
   - Push the working branch.
   - Create a PR with MCP when available. If unavailable, use `gh` only for PR work when installed/authenticated.
   - Use the repository PR template when present.
   - Include background, summary, verification, related issue, and any user decisions needed.
10. Update `00-META.md`:
    - Set `Status: implemented`.
    - Fill `PR URL` when known.
    - Commit the metadata update if it is part of the implementation branch.
11. Recommend `butter-workflow-code-review`.

## Constraints

- Do not perform code review as a separate critique during implementation. Defer full diff review to the code-review stage.
- Do not read or apply preference candidates. Only `active.md` is execution context.
- Keep verification output out of files unless the repository already has a convention for test artifacts.
- If the implementation materially differs from the spec (`01-SPEC.md`, or `02-PLAN.md` for Track B/C), update it before or with the code change.
