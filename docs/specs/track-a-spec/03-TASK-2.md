# Task 2: Update implement skill to read Track A docs

File: `skills/butter-workflow-implement/SKILL.md`

## Changes

1. Step 4 "Read": generalize so implement always reads `00-META.md` and
   `01-SPEC.md`, and additionally reads `02-PLAN.md`, relevant `03-TASK-*.md`,
   and Track C `Risk Review Targets` when they exist (Track B/C).
2. Make clear Track A implements from `00-META.md` + `01-SPEC.md` only.
3. Keep verify / commit / push / PR / `Status: implemented` steps as-is; confirm
   wording does not assume Track B/C-only docs exist.

## Done When

- Implement reads the spec for every track, including Track A, before changing
  code, and does not require `02-PLAN.md` / `03-TASK-*.md` for Track A.
