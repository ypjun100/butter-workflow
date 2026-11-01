# Spec Templates

Write headings in the language of the spec. Keep the `00-META.md` field keys and the `## Decisions`, `## Open Questions`, and `## Risk Review Targets` headings in English; other stages look them up by name.

## `00-META.md`

All tracks. Set `Track Type` to A, B, or C. `## Open Questions` exists only while `Status` is `drafting`.

```markdown
# Meta

- Track Type: A
- Base branch:
- Working branch:
- PR URL:
- Status: drafting

## Decisions

- <one decision per line; mark your own calls>

## Open Questions

- <one pending question per line>
```

## `01-SPEC.md`

```markdown
# Spec

## Goal

## Requirements

## Success Criteria

## Out of Scope
```

## `02-PLAN.md`

Track B/C. `## Risk Review Targets` is Track C only.

```markdown
# Plan

## Approach

## Constraints

## Verification

## Tasks

- 03-TASK-01: <one line>

## Risk Review Targets

| # | Risk | review-focus |
|---|---|---|
| 1 | | |
```

## `03-TASK-NN.md`

Track B/C. One file per commit-sized task.

```markdown
# Task NN: <title>

## Target

## Change

## Done Criteria
```
