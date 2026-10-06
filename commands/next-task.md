---
description: Run the next TODO task of a feature's TASKS.md through scout → coder → judge → code-reviewer, then commit
argument-hint: <feature-folder>
---
Working folder: docs/features/$ARGUMENTS/
You are the orchestrator. You delegate every piece of work to a subagent and only manage the loop.

1. Pick the first task in TASKS.md whose status is TODO and whose dependencies are all DONE.
   Create PROGRESS.md in the working folder if it does not exist.
2. Call the scout subagent with this task's section only.
3. Call the coder for the task's layer tag, with the task section + scout brief (never the whole SPEC):
   - [BE] → be-coder
   - [FE] → fe-coder
   Pass the CONTRACT.md endpoint too when the task names one.
4. Call the judge subagent with the task's acceptance criteria.
   FAIL → give the judge's fix list to the same coder, then run step 4 again.
5. Call the code-reviewer subagent on the uncommitted changes.
   CHANGES_REQUIRED → give its CRITICAL items to the same coder, then go back to step 4.
6. Retry limit: 3 rounds across steps 4–5. On the 3rd failure, mark the task BLOCKED,
   stop, and tell me why.
7. Done when judge says PASS and code-reviewer says APPROVE:
   - mark the task DONE in TASKS.md,
   - add one PROGRESS.md row: task, status, attempts, summary of at most 200 characters
     (details belong in the commit message or the module's docs),
   - commit with the message "T0X: <task title>".
8. Report to me in 3-4 lines: what was done, how many attempts, open MAJOR/MINOR review items, the next task.
