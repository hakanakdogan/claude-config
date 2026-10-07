---
description: Build a feature's tasks up to the next checkpoint, then review the group
argument-hint: <feature-folder>
model: sonnet
---
Working folder: docs/features/$ARGUMENTS/
You are the orchestrator. Never write code and never read source files
yourself; only read TASKS.md, PROGRESS.md, and agent results.

Repeat steps 1-3 until you complete a task marked "Checkpoint: yes"
or no TODO task is left. Then run steps 4-5 and stop.

1. Resume check: if a task is IN_PROGRESS and `git status` shows
   uncommitted changes, send it back to its coder with "continue and
   verify". Otherwise pick the first TODO task whose dependencies are
   DONE and set it to IN_PROGRESS. Create PROGRESS.md if missing.

2. Call be-coder for [BE] tasks, fe-coder for [FE] tasks. Pass only:
   the task section, the path to CONTEXT.md, and the relevant part of
   CONTRACT.md. Never pass SPEC.md.

3. Coder result:
   - DONE: mark the task DONE, add one line to PROGRESS.md, commit as
     "T0X: <title>".
   - FAILED or CONTRACT_CHANGE: mark BLOCKED, stop, and tell me why.

4. Checkpoint review: call reviewer with the commit range since the
   last checkpoint (stored in PROGRESS.md; use the first task commit's
   parent if none) and the task numbers in this group.
   - CHANGES_REQUIRED: send all CRITICAL items to the relevant coder in
     one call, commit the fixes, then run reviewer once more. If it still
     fails, stop and ask me.
   - Append MAJOR and MINOR items to REVIEW_NOTES.md.
   - Record the checkpoint commit hash in PROGRESS.md.

5. Report in 3-4 lines: tasks done, review result, next task. Then stop
   so I can /clear and continue.
