---
description: Build all TODO tasks of a feature's PLAN.md with coder subagents, then review once
argument-hint: <feature-folder>
model: sonnet
---
Working folder: docs/features/$ARGUMENTS/
You are the orchestrator. Never write code; never read source files.
Read PLAN.md once; afterwards only update task status lines in it.

1. Resume: a task with status IN_PROGRESS and uncommitted changes in `git status`
   → send it back to `coder` with "continue and verify".
   If PLAN.md has no `start:` line under the title, add `start: <current HEAD hash>`.
   That hash is START.

2. Loop until no TODO task is runnable:
   - Pick every TODO task whose deps are DONE. If two or more are runnable and their
     Files lists do not overlap, launch their `coder` calls in parallel (one message).
   - Pass each coder ONLY: its task section, the contract lines it references,
     and the CONTEXT.md path. Never pass the whole PLAN.md.
   - DONE → set status DONE, commit `T0X: <title>` (only that task's files).
   - FAILED or CONTRACT_CHANGE → set BLOCKED, stop, tell me why in 2 lines.

3. When every task is DONE: call `reviewer` once with range START..HEAD and the
   AC list from PLAN.md.
   - CHANGES_REQUIRED: send all CRITICAL items to `coder` in one call, commit
     `fix: review`, and stop (no second review unless I ask).
   - Append MAJOR/MINOR lines to PLAN.md under "## Review notes".

4. Reply in at most 4 lines: tasks done, review verdict, open items.
