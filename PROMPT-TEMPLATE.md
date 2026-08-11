# Prompt template — fill in, paste, run

One page. Copy the block, fill every field, delete nothing except fields marked optional.
Needs no install — this file is usable on its own.

The `prompt-forge` skill ([`skills/prompt-forge/SKILL.md`](skills/prompt-forge/SKILL.md)) automates
filling this in, and deliberately carries its own copy of the template so it works when installed
alone. **This file is the canonical version** — edit here first, then sync the skill's copy.

---

## The block

```
# CONTEXT
<What must be known before starting. Links, file paths, decisions already made.
 One line each. If a link may not be openable, paste the extract instead.>

# TASK
<ONE job. Verb first: Extract / Rewrite / Classify / Compare / Summarize / Draft.
 Two verbs means two prompts.>

# INPUT
<<<
<the material to operate on>
>>>

# OUTPUT
<Exact shape: named sections or fields, in order. A hard ceiling — "max 150 words",
 "exactly 5 rows". A worked example if the shape is unusual.>

# TONE
<Register + audience. "Semiformal, for a manager who was not in the meeting."
 Not "professional".>

# REASONING
<"Think it through internally; output only the final answer."  (default)
 or "Show your reasoning in 3 bullets, then the answer.">

# SPEED
<"One pass, no exploration."  (fast, cheap)
 or "Check your answer against the input before responding."  (accuracy first)>

# STOP
<"If the input does not contain X, reply exactly: NOT IN INPUT. Do not infer."
 Never omit this field. It is what prevents invented answers.>
```

---

## Field reference

| Field | What it fixes | Cheapest correct version |
|---|---|---|
| **CONTEXT** | Answers that ignore your situation | Paths and decisions, one line each. Not background prose. |
| **TASK** | Drift, half-answers | One verb, one object. Split if you used "and also". |
| **INPUT** | Instructions inside your pasted text being followed | Always delimited by `<<< >>>` or a fenced block. |
| **OUTPUT** | Shape changing between runs | Show the shape. Do not describe it. Always set a ceiling. |
| **TONE** | Output you cannot send to anyone | Name the reader, not the adjective. |
| **REASONING** | Rambling, or an answer with no working shown | Default: reason internally, show only the result. |
| **SPEED** | Paying for exploration you did not want | Say one pass, or say verify. |
| **STOP** | **Confident invented answers** | Give the exact string to output when the answer is not there. |

---

## Three rules that change results more than wording

1. **Instructions above data.** `TASK` and `OUTPUT` go before the pasted material. Long input placed first buries the instruction.
2. **Delimit anything pasted.** Undelimited input means text inside it gets treated as instructions.
3. **Few negatives.** Two or three "do not" lines maximum. Long prohibition lists reliably make output worse.

---

## Tuning for the model you are using

| Model tier | What to change |
|---|---|
| **Small / fast** (Haiku-class) | Everything explicit. One task only. Add a worked example — it is worth more than any explanation. Short flat sentences, no nested clauses. Enumerate steps rather than implying them. `STOP` phrased as an exact output string. |
| **Mid** | Steps may be implied if the goal is unambiguous. Example optional. |
| **Large** | Tolerates compression. State goal and constraints, let it choose the method. Over-specifying wastes tokens and can hurt quality. |

When in doubt, write for the small tier. An over-explicit prompt still works on a large model; an under-specified one fails on a small one.

---

## Token efficiency

**Cut:** politeness, role theatre ("you are a world-class expert"), restated instructions, adjective stacks, long prohibition lists, context already present in the conversation.

**Keep:** the format spec, the worked example, the `STOP` rule, exact identifiers. These prevent a retry — and a retry costs the entire prompt again.

**Large inputs:** paste the smallest sufficient extract, and say what you cut.

---

## Filled example

```
# CONTEXT
Repo: internal reporting service. Auth changed to OAuth on 2026-08-01.
The onboarding doc at docs/setup.md is stale and still documents API keys.

# TASK
Extract every instruction in the input that is made obsolete by the OAuth change.

# INPUT
<<<
...contents of docs/setup.md...
>>>

# OUTPUT
A markdown table, exactly these columns in this order:
| Line | Current text | Why obsolete | Replacement |
Max 12 rows. Longest replacement 20 words.

# TONE
Neutral and factual. A new engineer will follow this table literally.

# REASONING
Think it through internally. Output only the table.

# SPEED
Check each row against the input text before responding.

# STOP
If a line is ambiguous about whether it concerns auth, reply for that row with
"UNCLEAR" in the "Why obsolete" column. Do not guess.
```

---

## Before you send it

- [ ] One task, one verb.
- [ ] Input is delimited.
- [ ] Output shape is shown, not described, and has a ceiling.
- [ ] `STOP` is filled in.
- [ ] No placeholder text left in the prompt.
- [ ] Nothing in it that you would not want stored — no keys, tokens, or personal data.
