---
name: butter-workflow-start
description: |-
  Trigger: the user names this skill or Butter Workflow itself — `/butter-workflow-start`, `$butter-workflow-start`, "use the butter-workflow-start skill", or a request that mentions Butter Workflow by name. Classifies Track A/B/C, plans the working branch, bootstraps shared preference data, and writes docs/specs/{TASK-ID} spec files for every track before pausing for approval.
  Skip: the request only describes work to do — "start this task", "let's plan this out", a pasted issue link — without naming this skill or Butter Workflow; do that work directly instead. Also skip when an approved docs/specs/{TASK-ID}/00-META.md already exists for the current task (Status is planned or later) — use `butter-workflow-implement` to continue that work.
---

# Butter Workflow Start

Start a workflow from the context the user provides and leave enough repository state for another tool or session to continue.

## Invocation

Run this stage only when the user named it. Naming means the tool's invocation
syntax (`/butter-workflow-start`, `$butter-workflow-start`) or a plain mention
of this skill or of Butter Workflow. Describing a task is not naming it, however
much it sounds like the start of one.

This stage is the exception among the three. The later stages can be inferred
from repository state that proves a workflow is running; before this stage runs,
no such state exists, so the user's declaration is the only signal.

If this stage was entered from a message that only described work, say so in one
line and ask whether to run the workflow or just do the work — before reading
repository state and before writing any file.

## Inputs

- The context the user provides about the task, in whatever form they give it.

## Workflow

1. Read repository instructions first: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or equivalent files when present.
2. Bootstrap shared preference data under `~/.agents/preferences/`:
   - Use `preferences.md` when it exists.
   - Otherwise create it from this skill's `references/preferences.template.md`.
3. Read `~/.agents/preferences/preferences.md` when it exists and apply it while writing the spec and plan. It is the only preference file.
4. Understand the task from the provided context. When the context references retrievable external resources, enrich your understanding with available Jira/GitHub MCP tools, or with `gh` for GitHub when MCP is unavailable.
5. Collect project context:
   - Current branch and clean/dirty working tree.
   - Existing branch naming convention with `git branch --list`.
   - Build/test/package files relevant to the repository.
6. Set base branch to the branch active at workflow start.
7. Plan the working branch name; the implement stage creates and checks it out:
   - Classify work type as `feature`, `fix`, `refactor`, or `docs`.
   - Derive task id from the issue key, GitHub issue number as `gh-N`, or a short slug when no id exists.
   - Follow existing branch prefix conventions when clear; otherwise use `feature/<task-id>-<slug>`, `fix/<task-id>-<slug>`, `refactor/<task-id>-<slug>`, or `docs/<task-id>-<slug>`.
8. Classify Track Type:
   - Track A: small, low-risk, few files, no API/DB/auth/security/shared-module impact, rollback is easy.
   - Track B: needs planning docs, may touch multiple files, fits existing architecture, requires tests or type checks.
   - Track C: Track B plus auth, security, payment, permission, shared-core, architecture, migration, or broad refactor risk.
9. Confirm the track with the user when the classification is ambiguous or when Track C is selected. For obvious Track A/B, proceed and state the assumption.
10. Open the spec before asking anything:
    - Resume a `drafting` spec when the user points at it or asks to continue, or when the task id above matches a `docs/specs/{TASK-ID}/` whose `00-META.md` is `drafting`. Pick the dialogue up from its `## Decisions` and `## Open Questions`. Ask only when it is unclear which spec is meant.
    - Otherwise create `docs/specs/{TASK-ID}/` and write `00-META.md` from this skill's `references/spec-templates.md` with `Status: drafting`, the base branch, and the planned working branch.
11. Hold the design dialogue (`## Design Dialogue`) and fill in `01-SPEC.md` as each part is confirmed. Track B/C first reads this skill's `references/planning-research.md` and dispatches the research fan-out it describes, in a single batch, so the dialogue runs while it does. Track A skips the fan-out.
12. Track B/C: read the research reports before planning. Move the track from B to C when they surface risk the classification missed, tell the user when it moves, and confirm Track C with them before planning continues.
13. Track B/C: write `02-PLAN.md` from the research reports and the dialogue, using this skill's `references/spec-templates.md`. The `03-TASK-*.md` files come after validation. Track A writes only `00-META.md` and `01-SPEC.md`. Do not implement or open a PR during start.
14. For Track C, add `## Risk Review Targets` to `02-PLAN.md` and self-check whether each high-risk scope has review focus.
15. Track B/C: read this skill's `references/plan-validation.md` and hand `02-PLAN.md` to a validation agent as it describes before going further. Apply what it finds, re-validate once, then write the `03-TASK-*.md` files from the settled plan and check that none of them repeats `02-PLAN.md`. Track A skips this gate.
16. Request approval and stop, for every track. First remove `## Open Questions` from `00-META.md` and set `Status: planned`.
    - Give the user the spec directory path.
    - List every spec file written in this run as a clickable Markdown link (for example, `[00-META.md](/absolute/path/to/docs/specs/{TASK-ID}/00-META.md)`) with a short note of what it covers. Put the files the user reviews (`00-META.md`, `01-SPEC.md`, and on Track B/C `02-PLAN.md`) apart from the `03-TASK-*.md` files written for the implementing agent. Never list a file that was not written.
    - Point the user to the calls you made without asking; they are marked in `## Decisions`.
    - Track B/C: add one line per validation finding — what was applied, and what was left unapplied and why.
    - Wait for approval before implementation.

## Design Dialogue

Ask before deciding. The user holds what the repository cannot show — know-how, the project's history, where it is heading — and a plan built on guesses about those comes back as rework.

- Read before asking: the request, the instruction files, and the code and documents involved. Never ask what they already answer. On Track B/C, later questions build on the research reports.
- Open by playing the request back in your own words with what you found that bears on it — a constraint, a conflict, a cheaper path — and end with one question.
- Ask one question per message and say what its answer changes. Play goals, non-goals, and success criteria back part by part for confirmation.
- When a point settles, name what is still open, or say nothing is. A reply about something that arrived mid-dialogue, such as a research report, ends by pointing back to the pending question.
- A decision goes to the user when two or more options are viable, their tradeoffs pull against each other, and the pick depends on the user's know-how, history, or direction. Give that fork its own message: what each option means here, its tradeoffs, the combination when options can combine, and your recommendation with its reason.
- Decide the rest yourself: what the repository, the documents, or `preferences.md` answers, or where one option is plainly better.
- Record settled points in `00-META.md` under `## Decisions`, marking your own calls, and pending questions under `## Open Questions`. Move a question to Decisions once it is answered or deferred.
- Ask in a plain message. A structured question feature is fine to use, but nothing here depends on one.
- When an answer goes to the preference capture agent, pass along the question it answers. If a captured entry comes from a question still open, say so beside its report line.
- The dialogue ends when no fork the user must decide is left. If the user asks to stop the questions, decide the rest and record each as your own call.
- Track A: play your understanding back once, get it confirmed, and write `01-SPEC.md`.
- If another skill offers its own design or brainstorming process alongside this stage, follow this section and keep the output in `docs/specs/{TASK-ID}/`.

## Follow-Up Requests

Once `Status` is `planned`, this stage stays paused. While it is still `drafting`, a reply is part of the design dialogue; handle it under `## Design Dialogue`. An active workflow does not make every later message a stage transition. Route the next one:

- Spec change ("fix X in the spec", "add Y to the plan") — edit the named spec file, restate what changed, and stay paused. Do not re-run this stage and do not start implementing.
- Approval to build ("looks good, start implementing") — hand off to `butter-workflow-implement`.
- Anything else — an ordinary request; carry it out directly. This stage runs again only when the user names it again.

Routing is independent of `## Preference Capture`. Whichever branch a message takes, it still goes to a capture agent.

## Preference Capture

Keep this section identical in `butter-workflow-start`,
`butter-workflow-implement`, and `butter-workflow-code-review`. When you change
one copy, change all three.

### What Counts

Record anything that would help you write a better plan, a better change, or a
better review next time. The bar is deliberately low. It is not limited to what
the user would call a "preference".

Most useful signals arrive attached to one specific thing. The user says "this
component is too complex, simplify it", not "I like simple code". **Extract the
general rule behind the request and record that rule.** Never skip a request
just because it names a file, a function, or a value.

- "Write the plan in Korean, not English" → plan documents are written in
  Korean from now on.
- "This component's logic is too complicated, unpack it" → favor simple, direct
  code; do not pile on defensive handling for unlikely cases.
- "Leave the tool credit out of the commit message" → commit messages carry the
  change description only.

When unsure, record it. A wrong entry costs the user one line to delete. A
missing entry is invisible to them and never gets corrected.

Skip only these:

- `preferences.md` already holds an entry that means the same thing.
- The project instruction files (`AGENTS.md`, `CLAUDE.md`, or equivalent)
  already state it. Project rules are not preferences.
- It is a pure one-off with no rule behind it: a single value, a rename, a
  scope tweak for this task alone.
- It carries no content: a bare approval, acknowledgement, or rejection.

Judge equivalence by meaning, not by string match. The same rule worded
differently is the same entry.

### Delegate The Judgement

Do not decide what is worth recording yourself. When a user message carries any
content at all — feedback, a request, a question, a correction — hand the
message to a separate capture agent and get on with the work the user asked
for. The two run in parallel.

The only call you make is whether the message has content, not whether it is
worth recording. A message that is nothing but "ok", "looks good", or "go
ahead" needs no agent. Everything else gets one.

This holds no matter how `## Follow-Up Requests` routes the message. A spec
edit, a code fix, a list of review replies — each still gets a capture agent.
Routing decides what work you do; it never decides whether capture happens.

Give the capture agent:

- The user's message, verbatim and unsummarized.
- The current stage name.
- The path `~/.agents/preferences/preferences.md` and the project instruction
  file paths.

Instruct it to:

1. Read `preferences.md`. When it is missing, create it with the eight
   category headings listed below, each holding `- None yet.`
2. Read the project instruction files.
3. Split the message into separate items. One message may carry dozens; judge
   each on its own.
4. Extract the general rule behind each item.
5. Record the rule when it could shape a future plan, change, or review. Record
   when unsure.
6. Skip items that hit a skip condition.
7. Replace the older entry when a new one contradicts it. Do not ask.
8. Delete an entry when the user asks for it to be dropped.
9. Write the file.
10. Return one line per entry recorded, replaced, or deleted. Return nothing
    otherwise.

Writing rules for the agent:

- Add, replace, or delete single entries only. Never rewrite or reformat the
  whole file.
- File each entry under one of `Planning`, `Architecture`, `Naming`, `Testing`,
  `Implementation`, `Review And PR`, `Working Style`, `Communication`. Add the
  heading when the file lacks it.
- Replace a section's `- None yet.` placeholder when adding its first real
  entry.
- Never ask for approval first.

When the tool has no subagent mechanism, do the same work inline at the end of
your response. Same criteria, same writing rules, same report. Only the place it
runs changes.

### Report

Collect the capture agent's result before ending the turn and append its lines
to your response, written in the language the user is working in:

- `Preference recorded: <one-line summary>`
- `Preference updated: <old entry> → <new entry>`
- `Preference removed: <deleted entry>`

Say nothing when nothing was recorded.

Recording is never a substitute for acting. Apply the request to the current
work as well.

## Document Rules

- `00-META.md` is the workflow state source of truth for every track. `Status` moves `drafting` → `planned` → `implemented` → `reviewed`; `reviewed` is terminal, and there is no separate finish stage.
- The user reviews `00-META.md`, `01-SPEC.md`, and `02-PLAN.md`, so keep them short. `03-TASK-*.md` files are written for the implementing agent.
- Decision history lives only in `00-META.md` `## Decisions`, one line per decision. Mark the calls you made without asking, in the document's language (for example `(agent call)`). The other files state decisions, not how they were reached or what was rejected.
- `01-SPEC.md`: goal, requirements, success criteria, and out of scope. No function names, file names, or implementation choices.
- `02-PLAN.md` (Track B/C): implementation direction, constraints, verification, one line per task, and `## Risk Review Targets` on Track C.
- `03-TASK-*.md` (Track B/C): one commit-sized task each, with its target, change, and done criteria. No procedure and no assigned development method; the implementing agent chooses those.
- `02-PLAN.md` and the `03-TASK-*.md` files do not repeat each other.
- Preference data belongs in `~/.agents/preferences/preferences.md`, never in the spec directory. See `## Preference Capture`.
- Keep verification logs out of files. Summarize verification in the PR body or final response.

## Git And External Tools

- Use `git` only for read-only checks (status, diff, branch list) during start. Do not create branches, commit, or push.
- Use `gh` only for GitHub PR creation, PR lookup, and review comment lookup when no MCP tool is available and `gh` is installed/authenticated.
- Do not modify issue bodies or comments unless the user approves.
