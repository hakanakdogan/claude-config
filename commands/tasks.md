---
description: Split an approved SPEC.md into ordered, vertical-slice tasks in TASKS.md
argument-hint: <feature-folder>
model: opus
---
Working folder: docs/features/$ARGUMENTS/
You are the task breaker. Your output is a plan: HOW and IN WHAT ORDER the
spec will be built. Do not change the spec; if it is unclear or contradictory,
stop and ask me instead of guessing.

1. Read SPEC.md, and CONTRACT.md and CONTEXT.md if they exist.

2. Write TASKS.md. Each task must:
   - Be a vertical slice within its layer: one endpoint or one screen with
     its validation, logic, and tests together. Do not split by technical
     layer (model, repository, service, controller as separate tasks).
   - Belong to one layer: [BE] or [FE], never both.
   - Fit in a single coder session and be testable on its own.
   - Reference the acceptance criteria it satisfies.
   - Point to the relevant CONTRACT.md endpoint when it touches the API.
   A typical feature has 4-8 tasks. Prefer fewer, larger tasks as long as
   each still fits in one coder session.

3. Ordering and checkpoints:
   - If CONTEXT.md shows missing tests in an area that will change, T01
     writes tests that lock in the current behavior; feature work depends
     on it.
   - A [FE] task that consumes an endpoint depends on the [BE] task that
     builds it, unless it explicitly works against a contract-based mock
     (then add an integration task at the end).
   - Group tasks into logical groups of 2-4 tasks. Mark the last task of
     each group with "Checkpoint: yes". The final task is always a
     checkpoint.

4. Format, one section per task:

   ## T01 [BE] - Title
   - Status: TODO
   - Depends on: none | T0X
   - Spec refs: AC-1, AC-3
   - Contract: POST /orders/{id}/cancel | none
   - Files: expected files to touch
   - Goal: one paragraph
   - Acceptance criteria: testable, taken from the spec refs
   - Checkpoint: yes | no

   Allowed status values: TODO | IN_PROGRESS | DONE | BLOCKED.

5. Coverage check: verify that every AC in SPEC.md is referenced by at
   least one task. List any uncovered AC and fix TASKS.md before finishing.

6. Finish with: number of tasks, BE/FE split, checkpoints, and the coverage
   result. Tell me to review TASKS.md before running /next-task $ARGUMENTS.
