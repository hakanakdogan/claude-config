---
description: Turn an analysis or a short brief into one PLAN.md (spec + contract + tasks) for a feature
argument-hint: <feature-folder> [one-line brief]
model: opus
---
Working folder: docs/features/<first word of $ARGUMENTS>/  (rest of $ARGUMENTS = brief, if any)
You are the planner. Output: ONE file, PLAN.md. No code.

1. Input (read each source once):
   - ANALYSIS.md if present. If only a .docx/.pdf exists, convert it once
     (`pandoc <file> -t gfm -o ANALYSIS.md`, else Python) and continue.
   - No analysis: use the brief from $ARGUMENTS.
   - Parent EPIC.md, if present: read it and only the "Contract" sections of
     prerequisite features' PLAN.md. Reference them, do not redefine.
   - Existing code and no CONTEXT.md: call `scout` with the scope and the
     CONTEXT.md path. Do not explore the code yourself.

2. Questions: ONE message, at most 6 numbered questions, each with your
   recommended answer. Only ask what would change an acceptance criterion.
   Also in that message, list at most 3 likely gaps (security, data consistency,
   performance, missing edge case) as "suggested ACs". I answer everything in one reply.
   If the input spans several independent features, propose the split here instead:
   on my yes, write EPIC.md (features, order, shared concepts) + one subfolder per
   feature with its ANALYSIS.md slice, then stop.

3. Write PLAN.md (template below). Rules for tasks:
   - Vertical slice per layer: one endpoint or one screen with its logic and tests.
     Never split by technical layer. One layer per task: [BE] or [FE].
   - Prefer fewer, larger tasks (typically 3-6) that each fit one coder session.
   - CONTEXT.md shows no tests in an area that changes → T01 locks current behavior.
   - [FE] tasks depend on the [BE] task that builds their endpoint.
   - Coverage: every AC is referenced by at least one task. Check before saving.

```
# <Feature>
## Goal            (2-3 sentences)
## Data model      (only new/changed fields)
## Behavior        (bullets, incl. edge cases and errors)
## Acceptance criteria
- AC-1 ... (each testable)
## Out of scope    (incl. rejected suggestions)
## Contract        (per endpoint: METHOD path, request, response, errors, permission; or "none")
## Tasks
### T01 [BE] <title> | deps: none | AC: 1,2 | contract: POST /x | status: TODO
Files: ...
Do: one short paragraph
```

4. Reply in 3 lines: task count (BE/FE), coverage OK, then:
   "Review PLAN.md, then run /build <feature-folder>".
