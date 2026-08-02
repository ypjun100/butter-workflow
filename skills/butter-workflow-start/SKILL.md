---
name: butter-workflow-start
description: |-
  Trigger: the user wants to begin, kick off, or scope a new piece of work (e.g. "start this task", "let's plan this out", references a new issue/ticket) with no existing spec for it yet. Classifies Track A/B/C, plans the working branch, bootstraps shared preference data, and writes docs/specs/{TASK-ID} spec files for every track before pausing for approval.
  Skip: an approved docs/specs/{TASK-ID}/00-META.md already exists for the current task (Status is planned or later) — use `butter-workflow-implement` instead to continue that work.
---

# Butter Workflow Start

Start a workflow from the context the user provides and leave enough repository state for another tool or session to continue.

## Inputs

- The context the user provides about the task, in whatever form they give it.

## Workflow

1. Read repository instructions first: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or equivalent files when present.
2. Bootstrap shared preference data under `~/.agents/preferences/` if missing:
   - Use this skill's `references/active.template.md`,
     `references/candidates.template.md`, and
     `references/rejected.template.md`.
   - Create only missing `active.md`, `candidates.md`, and `rejected.md`.
   - Preserve existing preference data.
3. Read `~/.agents/preferences/active.md` when it exists and let it shape the spec. Read `candidates.md` and `rejected.md` only for the duplicate and promotion checks in `## Preference Capture`; never apply a candidate as a working rule.
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
    - Track B/C: create `docs/specs/{TASK-ID}/` and write `00-META.md`, `01-SPEC.md`, `02-PLAN.md`, and one or more `03-TASK-*.md`.
12. For Track C, add `## Risk Review Targets` to `02-PLAN.md` and self-check whether each high-risk scope has review focus.
13. Stop in feedback mode after writing the spec for every track (A/B/C):
    - Give the user the spec directory path.
    - Summarize what was written: list every spec file created in this run, each rendered as a clickable Markdown link to the file (for example, `[00-META.md](/absolute/path/to/docs/specs/{TASK-ID}/00-META.md)`), with a short note of what each covers. List only files actually written for the track (Track A: `00-META.md`, `01-SPEC.md`; Track B/C: the full set including every `03-TASK-*.md` written). Never list a file that was not written.
    - Wait for approval before implementation.

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

## Document Rules

- `00-META.md` is the workflow state source of truth for every track.
- Track A writes only `00-META.md` and `01-SPEC.md`. `02-PLAN.md` and `03-TASK-*.md` are Track B/C only.
- `01-SPEC.md` describes what users need and success criteria. Do not include function names, file names, or implementation choices.
- `02-PLAN.md` describes architecture boundaries, module responsibilities, call flow, data/API shape, test strategy, constraints, and task split.
- `03-TASK-*.md` files must be implementable units and should align with commit-sized changes.
- Preference data belongs in `~/.agents/preferences/`, never in the spec directory. See `## Preference Capture`.
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

`Status` moves `planned` → `implemented` → `reviewed`. `reviewed` is the
terminal state; there is no separate finish stage.
