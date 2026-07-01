# Spec

## Background

In the current workflow, a small low-risk task (Track A) goes straight from the
start stage into implementation and PR creation. This makes the agent act on its
own judgment and open a pull request before the user has reviewed any plan, which
feels arbitrary and hard to course-correct.

## What Is Needed

Every track, including small low-risk work, should produce a written spec during
the start stage and pause for user approval before any implementation happens.
Implementation should always proceed from that written spec.

For a small task the spec must stay lightweight so the overhead matches the size
of the change, while still giving the user something concrete to approve.

## Success Criteria

- Starting a small (Track A) task produces a written spec that captures what is
  needed and the success criteria, then stops and waits for user approval.
- No code changes and no PR are produced during the start stage for any track.
- The implementation stage reads the written spec for every track, including
  small tasks, before making changes.
- A small task's spec stays lightweight and does not require the full multi-part
  planning document set used for larger work.
- All user-facing and contributor-facing workflow documentation accurately
  states that small tasks also produce a spec and pause for approval, with no
  remaining claims that small tasks skip planning or implement directly from
  start.

## Out Of Scope

- Changing the Track B / Track C document set or their behavior.
- Changing track classification rules (what counts as Track A/B/C).
- Changing the finish or preference-capture stages.
