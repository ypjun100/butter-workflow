---
name: butter-workflow-finish
user-invocable: false
description: Finish a Butter Workflow by capturing reusable user preferences. Use after implementation or review to inspect Track B/C 04-PREFERENCES.md, current conversation, accepted review feedback, and preference files, then update shared user-preferences data without storing workflow state there.
---

# Butter Workflow Finish

Finish the workflow by updating shared preference data, not by approving release readiness.

## Workflow

1. Read repository instructions first.
2. Read shared preference files:
   - `~/.agents/preferences/active.md`
   - `~/.agents/preferences/candidates.md`
   - `~/.agents/preferences/rejected.md`
3. Determine the active workflow:
   - Prefer a task id or docs path supplied by the user.
   - Otherwise inspect `docs/specs/*/00-META.md` and choose the one matching the current branch or PR.
4. For Track B/C, read `04-PREFERENCES.md`.
5. For Track A, use only current conversation context and PR/review context; do not create `04-PREFERENCES.md`.
6. Identify observations from:
   - Explicit user feedback.
   - User-selected implementation or design direction.
   - User edits to code or docs.
   - Accepted review feedback.
   - Repeated comments or corrections.
7. Classify each observation:
   - Active: repeated preference, explicitly approved for future use, or already established.
   - Candidate: observed once and likely reusable.
   - Rejected: task-specific, conflicts with project rules, conflicts with active preferences, or user says not to generalize.
8. Update only the shared preference data files under `~/.agents/preferences/`.
9. Do not store workflow state, issue status, PR status, or verification results in user-preferences.
10. Update Track B/C `00-META.md` to `Status: finished` after preference capture is complete.

## Preference File Rules

- Keep entries concise and reusable.
- Preserve existing data unless moving a candidate to active or rejected.
- When promoting a candidate, remove or mark the old candidate to avoid duplicates.
- If no reusable preference was observed, say so and make no preference-data changes.

## Output

Summarize:

- Preferences added, promoted, or rejected.
- Files changed.
- Any follow-up the user must decide.
