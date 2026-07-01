# Task 1: Update start skill for Track A spec

File: `skills/butter-workflow-start/SKILL.md`

## Changes

1. Frontmatter `description`: remove "either implement a small Track A task or
   create docs/specs planning files for Track B/C"; replace with wording that
   says start creates spec docs for every track and pauses for approval.
2. Step 11 "Execute by track":
   - Track A: create `docs/specs/{TASK-ID}/` with `00-META.md` and `01-SPEC.md`
     only (lightweight). Do not implement, commit, push, or open a PR.
   - Track B/C: unchanged (full doc set).
3. Step 13: change the stop/feedback rule to apply to all tracks (A/B/C) — write
   the spec, give the docs path, wait for approval before implementation.
4. Document Rules section: note that Track A uses only `00-META.md` +
   `01-SPEC.md`, and that `02-PLAN.md` / `03-TASK-*.md` / `04-PREFERENCES.md` are
   Track B/C only.
5. `00-META.md` template: keep, and make clear `Track Type` can be A/B/C.

## Done When

- The skill instructs Track A to produce a lightweight spec and stop, with no
  implementation in start, and no leftover instructions to implement/PR from
  start.
