---
name: triage
description: Turn a raw bug report, failing test, error log, alert, or a whole queue of incoming problems into classified, deduplicated, routable tickets — severity, priority, component, owner, and one next action each. Use when the user says "triage this", "/triage", "how bad is this", "what should I fix first", "prioritize these bugs", "classify these failures", "is this a P0", "who owns this", or pastes an error, a stack trace, a CI failure, or a list of issues. Handles single items and batches. Do NOT use for fixing the bug (triage decides what gets attention, it does not repair), for code review, or for planning a feature backlog by business value.
---

Triage answers four questions and nothing else: **is it real, how bad is it, who owns it, what happens next.** Fixing is a separate job — resist it.

## Step 0 — Establish it is real (do this before classifying)

An unreproduced report is not triaged, it is filed. Work the ladder and stop at the first rung that holds:

1. **Reproduce** — run it, or state exactly what would. Record the command, environment, and version.
2. **Minimize** — strip the report to the shortest input that still fails.
3. **Isolate** — name the smallest suspected surface (file, service, endpoint, dependency, config).
4. **Capture** — exact error string, exit code, timestamp, request ID. Quote errors verbatim; never paraphrase an error.

If it cannot be reproduced, say so and classify as `NEEDS-INFO` with a list of exactly what is missing. Do not assign a severity to something you cannot confirm exists.

## Severity — what it does (objective)

Severity describes damage. It does not describe urgency, and it is never negotiated by how loudly the reporter complained.

| Sev | Definition |
|---|---|
| **S1** | Data loss, corruption, security exposure, or a core flow fully down in production. No workaround. |
| **S2** | Core flow broken or badly degraded for a real segment of users. Workaround exists but is painful. |
| **S3** | Non-core function broken, or core function degraded in a way most users tolerate. Clean workaround. |
| **S4** | Cosmetic, copy, or a rough edge. Nothing is blocked. |

## Priority — when it gets worked (a judgement)

`Priority = severity × reach × absence of workaround × time sensitivity.`

| Pri | Meaning |
|---|---|
| **P0** | Drop everything. Someone works it now. |
| **P1** | This cycle. Scheduled before new work. |
| **P2** | Next cycle. Tracked, not scheduled. |
| **P3** | Backlog. Fixed if it is touched anyway. |

S1 is not automatically P0 — an S1 on a feature nobody has enabled yet is P1. Say so out loud when severity and priority diverge; that gap is the most useful thing triage produces.

## Deduplicate before routing

Check whether it is already known. Search the tracker, recent commits, and the current queue for: the same error string, the same failing endpoint, the same stack frame.

- Same root cause, different symptom → **duplicate**. Link it, add the new symptom as evidence, close.
- Same symptom, different root cause → **not a duplicate**. Say why explicitly so the next person does not re-merge them.
- Recurring flake → classify as `FLAKY`, count the occurrences, and route to test reliability rather than the feature owner.

## Route

Owner is determined by the surface that must change, not by who reported it or who touched it last. If ownership is genuinely unclear, name the two candidates and say which is more likely and why — never leave the owner blank.

## Output — single item

```
TITLE     <one line, symptom-first, no speculation about the cause>
SEV / PRI S2 / P1
STATUS    CONFIRMED | NEEDS-INFO | DUPLICATE of <id> | FLAKY | NOT-A-BUG
COMPONENT <service / module / file>
OWNER     <team or person>
IMPACT    <who is affected, how many, what they cannot do>
REPRO     1. … 2. … 3. →  expected X, got Y
EVIDENCE  <exact error string, log line, or test name — verbatim>
CAUSE     <the suspected cause, explicitly marked as suspected, or "unknown">
NEXT      <one action, one owner. Not a plan — one action.>
```

## Output — batch

Sorted by priority, highest first. One row each, no prose between rows:

```
| Pri | Sev | Title | Component | Owner | Next |
```

Then, below the table, at most three lines: the single item to start with, any pattern connecting several rows (same root cause, same recent deploy), and anything blocked on missing information.

## Never

- Never fix while triaging. If the fix is genuinely one line, say so in `NEXT` and let the owner decide.
- Never invent a severity to match the reporter's tone.
- Never mark `CAUSE` as known without evidence — write "suspected" or "unknown". A confident wrong cause sends the next engineer down a dead end for a day.
- Never silently drop an item from a batch. Everything gets a row, including `NOT-A-BUG`.
- Never paraphrase an error string. Copy it exactly, including the noisy parts.

## Red flags

| Thought | Reality |
|---|---|
| "It looks serious, I'll call it S1" | Severity is defined by damage, not by appearance. Check the table. |
| "I can see the fix, let me just do it" | Different job. Record it in `NEXT`. |
| "Can't reproduce, but I'll classify it anyway" | `NEEDS-INFO` plus a list of what is missing. |
| "This is probably the same as that other one" | Confirm the root cause matches before merging, or you hide a second bug. |
| "Nobody owns this area" | Name two candidates and pick the likelier. Blank owner means nothing happens. |
| "Everything in this batch is P1" | Then nothing is. Force a strict ordering. |
