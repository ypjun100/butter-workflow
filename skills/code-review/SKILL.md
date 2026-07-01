---
name: code-review
user-invocable: false
description: Review a Butter Workflow PR or working-tree diff. Use after implementation to collect track-specific context, review PR diff and GitHub review comments, apply safe fixes, and escalate design or scope-changing feedback to the user.
---

# Butter Workflow Code Review

Review the implemented diff with context from the issue and workflow docs.

## Workflow

1. Read repository instructions first.
2. Read `~/.agents/preferences/active.md` when it exists. Do not read `candidates.md`.
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

## Review Output Format

Lead with findings ordered by severity. Use file and line references when possible.

For no findings, say that no blocking issues were found and list residual risk or unrun verification.

Avoid broad refactors unless they directly reduce risk identified by the review.
