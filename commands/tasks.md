---
description: Split an approved SPEC.md into small, ordered, testable tasks in TASKS.md
argument-hint: <feature-folder>
model: opus
---
Working folder: docs/features/$ARGUMENTS/
You are the task breaker. Your output is a plan: HOW and IN WHAT ORDER the
spec will be built. Do not change the spec; if it is unclear or contradictory,
stop and ask me instead of guessing.

1. Read SPEC.md, and CONTRACT.md and CONTEXT.md if they exist.

2. Write TASKS.md. Each task must:
   - Fit in a single coder session and be testable on its own
   - Belong to one layer: [BE] or [FE] (never both)
   - Reference the acceptance criteria it satisfies
   - Point to the relevant CONTRACT.md endpoint when it touches the API

3. Ordering rules:
   - If CONTEXT.md shows missing tests in an area that will change, T01 writes
     tests that lock in the current behavior; feature work depends on it.
   - A [FE] task that consumes an endpoint depends on the [BE] task that
     builds it, unless it explicitly works against a contract-based mock
     (then add an integration task at the end).
   - Mark the last task of each logical group with "Checkpoint: yes".

4. Format:

   ## T01 [BE] - Title
   - Status: TODO
   - Depends on: none | T0X
   - Spec refs: AC-1, AC-3
   - Contract: POST /orders/{id}/cancel | none
   - Files: expected files to touch
   - Goal: one paragraph
   - Acceptance criteria: testable, taken from the spec refs
   - Checkpoint: yes | no

5. Coverage check: verify that every AC in SPEC.md is referenced by at least
   one task. List any uncovered AC and fix TASKS.md before finishing.

6. Finish with: number of tasks, BE/FE split, checkpoints, and the coverage
   result. Tell me to review TASKS.md before running /next-task $ARGUMENTS.
