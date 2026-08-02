---
name: butter-workflow-code-review
description: |-
  Trigger: the user wants a PR or diff reviewed (e.g. "review this PR", "check this diff") once implementation has produced a PR or a meaningful branch diff. Collects track-specific context (issue, plan, task, risk targets), reviews the diff and review comments, applies safe fixes, and escalates design or scope-changing feedback.
  Skip: no PR and no meaningful branch diff exists yet — use `butter-workflow-implement` first.
---

# Butter Workflow Code Review

Review the implemented diff with context from the issue and workflow docs.

## Workflow

1. Read repository instructions first.
2. Read `~/.agents/preferences/active.md` when it exists and review against it. Read `candidates.md` and `rejected.md` only for the duplicate and promotion checks in `## Preference Capture`; never apply a candidate as a working rule.
3. Determine review target:
   - Prefer a PR URL supplied by the user or recorded in `00-META.md`.
   - Otherwise review the current branch diff against the base branch from `00-META.md`.
4. Collect context by track:
   - Track A: PR diff or branch diff, related issue, existing PR review comments, and `01-SPEC.md` as the requirement reference.
   - Track B: Track A context plus `02-PLAN.md` and relevant `03-TASK-*.md`.
   - Track C: Track B context plus `Risk Review Targets`.
5. Use MCP for PR and review comment access when available. If unavailable, use `gh` only for PR lookup, PR diff, and review comments when installed/authenticated.
6. Review in this order:
   - Requirement mismatch.
   - Bugs and behavioral regressions.
   - Side effects outside the planned scope.
   - Architecture boundary violations.
   - Security, authentication, authorization, privacy, and data integrity issues.
   - Missing tests or weak verification.
   - Conflicts with `active.md` preferences.
7. For Track C, explicitly inspect every `Risk Review Targets` item and report whether the implementation satisfies its `review-focus`.
8. Classify findings:
   - Immediate fixes: clear bug or small stability/test fix, low scope, no plan/preference conflict.
   - User decision: design change, scope expansion, plan conflict, preference conflict, or trade-off with no obvious winner.
9. Apply immediate fixes when safe, run targeted verification, commit, and push.
10. Present user-decision items with the feedback, benefit, downside, recommendation, and why user input is needed.
11. Update PR body or comment with review/fix summary when a PR is available.
12. Update `00-META.md` to `Status: reviewed` only after review and required safe fixes are complete.

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

## Review Output Format

Lead with findings ordered by severity. Use file and line references when possible.

For no findings, say that no blocking issues were found and list residual risk or unrun verification.

Avoid broad refactors unless they directly reduce risk identified by the review.
