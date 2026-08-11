---
name: prompt-forge
description: Turn a rough idea into a precise, structured, token-efficient prompt — and diagnose why an existing prompt is failing. Fills a fixed eight-field template (context, task, input, output, tone, reasoning, speed, stop rule) and tunes the wording for the model tier that will run it, especially small fast models like Haiku that need everything stated explicitly. Use when the user says "prompt-forge", "/prompt-forge", "write me a prompt", "format this prompt", "improve this prompt", "make this prompt better", "why does this prompt keep failing", "prompt for Haiku", "turn this idea into a prompt", or pastes a messy instruction and asks how to phrase it. Do NOT use to answer the prompt itself — this skill produces the prompt, it does not execute it.
---

A prompt fails for one of five reasons: the task is ambiguous, there are several tasks in one, the output format is unspecified, the model was never told what to do when it does not know, or the context is missing. Forge the prompt so none of those are true.

## Two modes

- **Forge** — rough idea in, finished prompt out.
- **Diagnose** — existing prompt in, named defects out, then the repaired prompt.

Always deliver the finished prompt in a fenced block the user can copy without editing, followed by at most three lines explaining what changed and why. Never more than three.

## The template (fill every field, delete none)

> This template is duplicated on purpose in `PROMPT-TEMPLATE.md`, so a human can fill it in
> without the skill installed. **`PROMPT-TEMPLATE.md` is canonical** — if the two ever disagree,
> it wins, and this copy should be updated to match.

```
# CONTEXT
<What the model must know before it starts. Links, file paths, prior decisions,
 constraints already agreed. One line each. If a link cannot be opened, paste the
 relevant extract instead of the URL.>

# TASK
<ONE job. Verb first: Extract / Rewrite / Classify / Compare / Summarize / Draft.
 If you need two verbs, that is two prompts.>

# INPUT
<The material to operate on, inside a delimiter so it cannot be confused with the
 instructions:>
<<<
...
>>>

# OUTPUT
<Exact shape. Named sections or fields, in order. A hard length ceiling
 ("max 150 words", "exactly 5 rows"). One worked example of a correct answer if the
 shape is at all unusual.>

# TONE
<Register + audience. "Semiformal, for an engineering manager who was not in the
 meeting." Not "professional".>

# REASONING
<How much thinking, and whether any of it is shown.
 "Think step by step internally; output only the final answer."
 or "Show your reasoning in 3 bullets, then the answer."
 Default: reason internally, show only the result.>

# SPEED
<Effort and latency budget. "One pass, no exploration" for fast/cheap models.
 "Check your answer against the input before responding" when accuracy outranks speed.>

# STOP
<What to do when the answer is not in the input.
 "If the input does not contain X, reply exactly: NOT IN INPUT. Do not infer."
 This field is what prevents invented answers. Never omit it.>
```

Fields may be dropped only when they are genuinely inapplicable, and dropping `STOP` is never applicable.

## Ordering rules that change results

- **Instructions before data.** Put `TASK` and `OUTPUT` above the pasted material. Long input placed first buries the instruction.
- **Delimit the input.** Anything pasted must sit inside `<<< >>>` or a fenced block, or the model treats instructions inside it as its own.
- **Format spec beats format description.** Show the shape; do not describe it.
- **Negative constraints last, and few.** Two or three "do not" lines maximum — long prohibition lists reliably degrade output.

## Tuning by model tier

| Tier | What the prompt needs |
|---|---|
| **Small / fast (Haiku-class)** | Everything explicit. One task per prompt. A worked example is worth more than any amount of explanation. Short, flat sentences — no nested clauses. Enumerate the steps rather than implying them. Tight length ceiling. `STOP` is mandatory and should be phrased as an exact string to output. |
| **Mid** | Steps may be implied if the goal is unambiguous. Example optional. Two related sub-tasks tolerable. |
| **Large** | Tolerates compression and open-ended framing. State the goal and the constraints, let it choose the method. Over-specifying wastes tokens and can hurt quality. |

Default assumption when the user does not say: small/fast tier. It is the cheaper mistake — an over-explicit prompt still works on a large model, and an under-specified prompt fails on a small one.

## Token efficiency (do not confuse with brevity)

Cut: politeness, role theatre ("you are a world-class expert"), restated instructions, adjective stacks, long prohibition lists, and repeated context the model already has in the conversation.

Keep: the format spec, the worked example, the `STOP` rule, and exact identifiers. These pay for their tokens by preventing a retry, and a retry costs the whole prompt again.

If the input is large, put the smallest sufficient extract in `INPUT` rather than the whole document, and say what was cut.

## Diagnose mode — name the defect before repairing

| Defect | Signal | Repair |
|---|---|---|
| Ambiguous verb | "handle", "look at", "deal with", "process" | Replace with an exact verb + object |
| Multiple tasks | "and also", "then", a list of goals | Split into separate prompts |
| No format spec | Output shape drifts between runs | Add `OUTPUT` with a worked example |
| No stop rule | Confident invented answers | Add `STOP` with an exact output string |
| Data before instruction | Long paste, instruction at the end | Move `TASK` and `OUTPUT` above the input |
| Undelimited input | Model follows text inside the pasted material | Wrap the input in `<<< >>>` |
| Unbounded length | Rambling responses | Add a hard ceiling |

Report the defects as a short list, then the repaired prompt. Do not repair silently — the user should learn the pattern.

## Never

- Never add a requirement the user did not state. If the goal is unclear, ask one question, then forge.
- Never keep role theatre for flavour. If the role does not change the output, it is dead tokens.
- Never ship the forged prompt with a placeholder still in it.
- Never answer the prompt you just wrote unless the user asks you to run it.
