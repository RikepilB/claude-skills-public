---
name: debug-evidence-loop
description: Diagnose a software defect through reproduction, minimal evidence, hypothesis testing, and focused verification. Do not use to implement unrelated improvements.
---

# Debug Evidence Loop

Reproduce or precisely characterize the failure. Gather the smallest relevant logs, isolate plausible causes, test one hypothesis at a time, and record the evidence that supports the conclusion.

Write down the observed behavior, expected behavior, reproduction steps and environment. Change
one plausible variable at a time. A log line, test or trace must distinguish the current hypothesis
from alternatives before implementing a fix.

After a requested fix, repeat the original reproduction and run the smallest relevant regression
check. Report what was verified and what remains untested. Do not run destructive commands or
read unrelated configuration.

## Red flags

| Temptation | Better move |
| --- | --- |
| Guess from an error message alone | Reproduce and inspect the first failing boundary. |
| Change several things at once | Test one cause so the result is interpretable. |
| Call a passing unit test a production fix | Repeat the original user-visible failure. |
