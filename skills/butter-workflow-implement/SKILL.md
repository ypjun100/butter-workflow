---
name: butter-workflow-implement
description: |-
  Trigger: docs/specs/{TASK-ID}/00-META.md exists with Status planned, and the user approves that spec or asks for it to be built ("implement this", "build the approved plan", "continue the implementation"). Both conditions are required. Reads workflow state, applies the approved track scope, runs verification, commits task-sized changes, pushes, and creates or updates a PR.
  Skip: no docs/specs/{TASK-ID}/00-META.md exists — the user never started a Butter Workflow, so do the requested work directly rather than pulling in this stage. Also skip for spec-file edits and single follow-up fixes inside an active workflow — apply those directly. Also skip once a PR already exists and is awaiting review — use `butter-workflow-code-review` instead.
---

# Butter Workflow Implement

Implement the approved spec while preserving the docs as the handoff surface.

## Workflow

1. Read repository instructions first.
2. Read `~/.agents/preferences/active.md` when it exists and follow it while implementing. Read `candidates.md` and `rejected.md` only for the duplicate and promotion checks in `## Preference Capture`; never apply a candidate as a working rule.
3. Find the active task docs:
   - Prefer the task id or path supplied by the user.
   - Otherwise inspect `docs/specs/*/00-META.md` and choose the one whose status is `planned` or whose working branch matches the current branch.
   - Ask the user only when multiple plausible active specs exist.
4. Read the spec for the task's track:
   - Always: `00-META.md` and `01-SPEC.md`.
   - Track B/C also: `02-PLAN.md` and relevant `03-TASK-*.md`.
   - Track C also: `02-PLAN.md` `Risk Review Targets`.
   - Track A implements from `00-META.md` and `01-SPEC.md` only.
5. Prepare the working branch from `00-META.md` before implementing. Ask first when the working tree is dirty or any branch create/switch operation would be risky:
   - If the `Working branch` does not exist, create it from the `Base branch` with `git switch -c <working-branch> <base-branch>`.
   - If it already exists, switch to it.
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
11. Recommend the `butter-workflow-code-review` skill. It is the last stage of the workflow.

## Follow-Up Requests

This stage is done once the PR exists. An active workflow does not make every
later message a stage transition. Route the next one:

- A specific code fix, or a list of feedback to apply — apply it, run targeted
  verification, commit, and push to the working branch. Do not run
  `butter-workflow-code-review` for it.
- A request to review the change — hand off to `butter-workflow-code-review`.
- A spec change — update the spec file, then make the matching code change.

## Preference Capture

Keep this section identical in `butter-workflow-start`,
`butter-workflow-implement`, and `butter-workflow-code-review`. When you change
one copy, change all three.

Check every user message that arrives while this skill is active for a reusable
preference. This includes messages sent after the stage's main work is done:
spec feedback, approval comments, review replies, and follow-up fix requests.

Record one only when all of these hold:

- It still applies to the next task after this one ends.
- It does not depend on a specific file, function, value, or issue.
- It is not already stated in the project instruction files (`AGENTS.md`,
  `CLAUDE.md`, or equivalent). Project rules are not preferences.
- It is not in `~/.agents/preferences/rejected.md`.

These are not preferences: a specific bug fix, a rename or value change, a
scope adjustment that applies only to this spec, a question, an approval, or a
rejection.

On a match, read all three files under `~/.agents/preferences/` and take the
first branch that fits:

1. Equivalent entry in `rejected.md` — record nothing.
2. Equivalent entry in `active.md` — do nothing.
3. Contradicts an entry in `active.md` — change no file and ask the user which
   one wins.
4. Equivalent entry in `candidates.md` — remove it there and add it to
   `active.md`. This is the promotion path.
5. Otherwise — add it to `candidates.md`.

Judge equivalence by meaning, not by string match. The same rule worded
differently is the same entry.

When writing:

- Create a missing preference file before writing to it and preserve any
  existing data.
- Add or move single entries only. Never rewrite or reformat a whole file.
- File the entry under one of `Planning`, `Architecture`, `Naming`, `Testing`,
  `Implementation`, `Review And PR`, `Working Style`, `Communication`. Add the
  heading when the target file lacks it.
- Replace a section's `- None yet.` placeholder when adding its first real
  entry to that section.

Also apply the request to the current work. Recording it is not a substitute
for acting on it.

Do not ask for approval before recording. After a record or a promotion, end
the response with one line, written in the language the user is working in:

- `Preference recorded (candidate): <one-line summary>`
- `Preference promoted (active): <one-line summary>`

Say nothing when no preference was recorded.

## Constraints

- Do not perform code review as a separate critique during implementation. Defer full diff review to the code-review stage.
- Do not apply preference candidates as working rules. Only `active.md` is execution context; `candidates.md` and `rejected.md` are read solely for `## Preference Capture` bookkeeping.
- Keep verification output out of files unless the repository already has a convention for test artifacts.
- If the implementation materially differs from the spec (`01-SPEC.md`, or `02-PLAN.md` for Track B/C), update it before or with the code change.
