# Plan

## Architecture Boundaries

The change lives entirely in the plugin's shared skill definitions and the
methodology documentation. There is no runtime code; the "behavior" is the
instructions the skills give to the agent. The repository uses a single shared
`skills/` directory consumed by both Claude Code and Codex, so each skill is
edited once.

Affected surfaces:

- `skills/start/SKILL.md` — defines what start does per track.
- `skills/implement/SKILL.md` — defines which docs implement reads.
- `skills/code-review/SKILL.md` — defines per-track review context.
- `docs/workflow-proposal.md` — methodology source of truth.
- `README.md` — user-facing track model and outputs.

## New Track A Document Shape

Track A produces a lightweight spec directory:

```text
docs/specs/{TASK-ID}/
  00-META.md   # state source of truth (Track Type: A)
  01-SPEC.md   # what is needed + success criteria
```

Track A does NOT create `02-PLAN.md`, `03-TASK-*.md`, or `04-PREFERENCES.md`.
Track B / Track C keep their existing full document set unchanged.

Note: because Track A no longer captures preferences via `04-PREFERENCES.md`,
the finish stage's Track A preference input continues to come from the
conversation and review context, exactly as it does today. The finish stage is
out of scope and is not modified.

## Behavior Changes

### Start stage

- Track A: instead of implementing + committing + pushing + PR, create the
  lightweight spec directory (`00-META.md` + `01-SPEC.md`), then stop in feedback
  mode and wait for user approval — the same pause B/C already use.
- The "stop after planning and wait for approval" rule now applies to all tracks
  (A/B/C), not only B/C.
- Branch creation still happens in start for every track.
- Update the skill `description` frontmatter so it no longer says start may
  implement a small Track A task.

### Implement stage

- Generalize doc reading: always read `00-META.md` and `01-SPEC.md`; additionally
  read `02-PLAN.md`, relevant `03-TASK-*.md`, and Track C risk targets when they
  exist (Track B/C). Track A implements from `00-META.md` + `01-SPEC.md`.
- The rest of implement (verify, commit, push, PR, set `Status: implemented`)
  is unchanged and now also covers Track A.

### Code-review stage

- Track A review context gains `01-SPEC.md` (in addition to diff, issue, and PR
  comments), since a spec now exists for Track A.

### Documentation

- `docs/workflow-proposal.md`: update the Track A section, the Start stage
  description, and Repository Outputs so Track A is described as "lightweight
  spec, pause for approval, implement from spec," removing claims that Track A
  skips planning docs or runs start-through-PR directly.
- `README.md`: update the Track Model list, the "What It Provides" bullets, the
  start skill role row, and Repository Outputs to reflect the Track A spec.

## Test / Verification Strategy

No executable tests exist (documentation/skill repo). Verification is
consistency review:

- Grep the repo for residual claims that Track A skips docs or implements from
  start (e.g. "no planning docs", "directly through implementation", "Track B/C
  workflows create").
- Confirm start / implement / code-review skills agree on the Track A doc set.
- Confirm README skills table and AGENTS.md documentation rule still hold.

## Constraints

- Keep Claude command wrappers and Codex entrypoints thin; all behavior stays in
  shared skills (per `AGENTS.md`).
- Keep verification logs out of committed files.
- Write commit messages, PR title, and PR body in English (per `AGENTS.md`).

## Task Split

- `03-TASK-1`: Update `start` skill (Track A → lightweight spec +
  stop for approval; all-track pause; frontmatter).
- `03-TASK-2`: Update `implement` skill (read Track A docs).
- `03-TASK-3`: Update `code-review` skill (Track A context + spec).
- `03-TASK-4`: Update `docs/workflow-proposal.md` and `README.md`.
