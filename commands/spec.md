---
description: Turn an analysis (or an interview) into SPEC.md and CONTRACT.md for a feature
argument-hint: <feature-folder>
model: opus
---
Working folder: docs/features/$ARGUMENTS/
You are the planner. Your output is a spec: WHAT the system must do and WHY.
Do not decide implementation order and do not write code.

1. Gather input:
   - If ANALYSIS.md is missing but a .docx or .pdf file exists in the
     working folder, convert it first: use pandoc if available
     (`pandoc <file> -t gfm --extract-media=. --track-changes=all -o ANALYSIS.md`),
     otherwise convert it with Python. Preserve headings, tables, and lists.
     Show me any part that did not convert cleanly before continuing.
   - If ANALYSIS.md exists, read it. If there is no analysis at all,
     interview me: ask what I want to build in 2-3 sentences, then
     continue with step 2.
   - If the parent folder contains EPIC.md, read it. For every prerequisite
     feature listed there, read its SPEC.md and CONTRACT.md. Reference their
     endpoints and data models instead of redefining them.
   - If the repository has code and CONTEXT.md is missing, call the scout
     subagent with this feature's scope and the CONTEXT.md path.

2. Before writing anything, ask me about ambiguities ONE question at a time:
   business rules, edge cases, permissions, error behavior, and how existing
   behavior is affected. Stop asking when the remaining unknowns would not
   change the acceptance criteria.

3. Scope check: if the input covers several independent features, propose
   splitting and wait for my answer. If I agree, do only this and stop:
   - Write EPIC.md in the working folder: the features with one-line
     summaries, the dependency graph (prerequisites and what can run in
     parallel), and shared concepts.
   - Create one subfolder per feature (fX-short-name/) with an ANALYSIS.md
     containing only that feature's part of the analysis. Put content that
     spans several features into EPIC.md.
   - Suggest merging features too small to stand alone.
   - Show the recommended order, then tell me to run /spec on the first one.

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
