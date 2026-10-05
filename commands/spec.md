---
description: Turn an analysis (or an interview) into SPEC.md and CONTRACT.md for a feature
argument-hint: <feature-folder>
model: opus
---
Working folder: docs/features/$ARGUMENTS/
You are the planner. Your output is a spec: WHAT the system must do and WHY.
Do not decide implementation order and do not write code.

1. Gather input:
   - If ANALYSIS.md exists in the working folder, read it.
   - If it does not, interview me: ask what I want to build in 2-3 sentences,
     then continue with the questions in step 2.
   - If the repository already has code and CONTEXT.md is missing, call the
     scout subagent first and have it write CONTEXT.md (max 40 lines: relevant
     files, data model, existing patterns, existing tests, risks).

2. Before writing anything, ask me about ambiguities ONE question at a time:
   business rules, edge cases, permissions, error behavior, and how existing
   behavior is affected. Stop asking when the remaining unknowns would not
   change the acceptance criteria.

3. Scope check: if the input covers several independent modules, propose
   splitting it into separate feature folders (one spec each) and wait for
   my answer before continuing.

4. Write SPEC.md with these sections, per feature:
   - Goal
   - Data model
   - Behavior (including edge cases)
   - Acceptance criteria: numbered (AC-1, AC-2, ...), each testable
   - Affected Existing Behavior (only for existing projects)
   - Out of Scope (always present, even if short)

5. If the feature has an API, write CONTRACT.md: for each endpoint the path,
   method, request and response shapes with field types, error codes and when
   they occur, and required permissions.

6. Call the advisor subagent on SPEC.md. Show me its suggestions and ask which
   to accept. Add accepted ones as acceptance criteria; add rejected ones to
   Out of Scope so they are not suggested again.

7. Finish with a 3-line summary and tell me to review SPEC.md before running
   /tasks $ARGUMENTS.
