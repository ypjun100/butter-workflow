# Plan Validation

Track B/C only. A plan drifts in the same three directions every time: it rebuilds something the repository already has, it reaches wider than the requirement asked for, and it invents conventions alongside the existing ones. Catch that before the user has to.

Hand the finished `02-PLAN.md` to a separate agent. The agent that wrote the plan does not validate it.

## When

After `02-PLAN.md` is written, and for Track C after `## Risk Review Targets` is added. Before any `03-TASK-*.md` exists — the task split comes out of the settled plan, not the draft.

## What The Validator Gets

- `00-META.md`, whose `## Decisions` separates what the user decided from the calls made without asking.
- `02-PLAN.md` and `01-SPEC.md`, in full.
- The planning research reports, verbatim and unsummarized.
- The project instruction file paths and `~/.agents/preferences/preferences.md`.

The validator stays read-only and inspects the repository independently instead of treating the research reports as complete. It does not write files, edit the spec or plan, or create branches or commits.

## What The Validator Checks

- **Reuse** — anything the plan proposes to build that the repository already has. Cite the existing location for every one of them.
- **Minimal change** — changes the requirement does not need, abstractions with no second caller, scope the user never asked for.
- **Architecture fit** — whether module and layer boundaries, dependency direction, and placement of responsibility match how this repository is already arranged.
- **Convention** — whether naming, file placement, error handling, and test location match the examples the `convention` research actually found, not the validator's own taste.
- **Document format** — whether `01-SPEC.md` and `02-PLAN.md` follow the skill's `## Document Rules`: no decision history or rejected options, one line per task, nothing that belongs in a task file.
- **Open decisions** — choices the plan leaves open, or makes as the planner's own call, where viable options pull against each other and the pick depends on the user's know-how, history, or direction. Leave out whatever the repository, the documents, or `## Decisions` already settles.
- Track C also: whether every high-risk scope has a matching entry under `## Risk Review Targets`.

## What The Validator Returns

- A one-line verdict, then the findings.
- Each finding: the point in the plan, the repository location that backs it, and what it should become instead.
- Questions for the user from the open-decision check, most important first.
- A clean verdict has to earn it. State what was searched for and not found. An empty pass with nothing behind it is not a pass.

## Applying It

- Ask the user the validator's questions before re-validating, one per message, in the fork format of the skill's `## Design Dialogue`. Record each answer in `## Decisions` and work it into the plan.
- Fix `02-PLAN.md` and send it back once. One re-validation round, not a loop.
- Ask any questions from the re-validation round before requesting approval, and mark the answers applied after it as not re-validated in the handoff summary.
- Findings still open after that round stay unapplied and go to the user in the handoff summary.
- A finding you disagree with stays unapplied too. Report it with the reason.
- Write the `03-TASK-*.md` files once the plan settles.
- Keep the validation record out of the spec directory. It belongs in the handoff summary, one line per finding.

When the tool has no subagent mechanism, run the same checks as a separate pass over the written plan before the task files exist. Same checks, same evidence requirement, same reporting. Only the place it runs changes.
