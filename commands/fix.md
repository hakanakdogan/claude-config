---
description: Bug fix: failing reproduction test first, then the smallest fix
argument-hint: <bug: expected vs actual>
model: sonnet
---
Bug: $ARGUMENTS

1. Locate with Grep; ranged Reads only.
2. Write the smallest failing test that reproduces the bug and run it (must be red).
   Cannot reproduce after an honest try → stop and report what you tried in 3 lines.
3. Root cause in one sentence (file:line), before changing anything.
4. Smallest fix touching only root-cause files. Never weaken, skip or delete tests,
   never swallow exceptions to hide the symptom.
5. Red test green, then `sh scripts/verify` once. The reproduction test stays.
6. After 3 failed fix attempts: stop and tell me whether the cause is in the code,
   the plan, or the requirement.
Reply in at most 4 lines: root cause, changed files, verify result.
