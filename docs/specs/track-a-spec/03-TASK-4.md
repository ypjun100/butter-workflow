# Task 4: Update methodology docs and README

Files: `docs/workflow-proposal.md`, `README.md`

## Changes — workflow-proposal.md

1. Track A section: replace "do not need planning documents" and "move from start
   directly through implementation ... PR creation" with the new behavior — Track
   A writes a lightweight spec (`00-META.md` + `01-SPEC.md`), pauses for approval,
   and implements from the spec.
2. Stage "1. Start": remove "For Track A, start may continue through
   implementation and PR creation"; state that start writes a spec for every
   track and stops for approval.
3. Repository Outputs / Track model: reflect that Track A also creates a (lighter)
   spec directory.

## Changes — README.md

1. "What It Provides": adjust the bullet that implies docs are Track B/C only.
2. Skills table row for `start`: reflect spec-for-every-track.
3. "Track Model" list: change "Track A: ... no planning docs" to the lightweight
   spec behavior.
4. "Repository Outputs": note Track A produces `00-META.md` + `01-SPEC.md`;
   Track B/C add the rest.

## Done When

- No remaining text claims Track A skips planning docs or implements directly
  from start; docs and README match the start/implement skills.
- README skills table still matches current skill directories (AGENTS.md rule).
