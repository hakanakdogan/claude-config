# Global working rules (keep this file short: it loads into every session)

- Answer tersely. No preamble, no recap of what you just did, no restating the plan.
- Read only what the task needs: Grep/Glob first, then Read with offset/limit.
  Never re-read a file you already read or just edited.
- Never paste full logs or large outputs; keep only the failing lines.
- Tests: while iterating run only the related tests; run the full check once at the end.
- Small change (≤ 3 files, no new API/data model)? Just do it. No spec, no subagents.
- Ask questions in ONE batched message with your recommended answer for each.
- Project rules live in the project's CLAUDE.md; project verify command is `sh scripts/verify`
  if it exists.
