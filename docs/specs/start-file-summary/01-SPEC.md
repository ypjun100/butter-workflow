# Spec

## Background

When the start workflow finishes, it pauses in feedback mode and hands the user
the spec directory path. The user only learns *where* the output lives, not
*what* was produced. To see which spec files were written, the user must open
the directory themselves. This is friction, and it makes the handoff easy to
misread — especially across tracks, where Track A writes only two files while
Track B/C write the full set.

## What users need

After the start workflow writes the spec files, the final message to the user
should summarize what was created before the approval pause:

- A concise list of every spec file that was written for this run.
- Each listed file is presented as a hyperlink the user can click to open the
  file directly, rather than plain text they must locate themselves.
- A short note on what each file covers, so the user can judge coverage without
  opening the directory.
- The existing spec directory path and the request to approve before
  implementation, preserved.

The summary must reflect only the files actually written for the current track
(Track A: the lightweight set; Track B/C: the full set), so it never claims a
file exists when it does not.

## Success criteria

- After a start run, the final response enumerates each spec file that was
  created under the spec directory, with a brief description of each.
- Each enumerated file is rendered as a clickable hyperlink to that file, not as
  plain text.
- The list matches the track: Track A shows exactly the lightweight files;
  Track B/C show the full set including every task file that was written.
- The final response still includes the spec directory path and the
  approval-before-implementation pause.
- The summary never lists a file that was not written in this run.

## Out of scope

- Changing which files each track writes.
- Changing the implement, review, or finish workflows.
- Any change to how or when the workflow pauses for approval, beyond adding the
  file summary to the handoff message.
