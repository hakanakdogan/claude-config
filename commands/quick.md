---
description: Small change done directly in this session: no spec, no subagents
argument-hint: <what to change>
model: sonnet
---
Do this change directly: $ARGUMENTS

- Grep/Glob to locate; ranged Reads only. Do not explore beyond what the change needs.
- If it turns out to need a new endpoint, a data model change, or more than ~5 files,
  stop and tell me to use /feat instead.
- Add or adjust a test when behavior changes.
- Run related tests, then `sh scripts/verify` once.
- Reply in at most 3 lines: changed files, verify result, anything I must decide.
