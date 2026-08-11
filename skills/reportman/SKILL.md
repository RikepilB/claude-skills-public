---
name: reportman
description: Manager-grade reporting mode — turns work, findings, incidents or status into a short semiformal report: headline verdict first, two or three tight paragraphs, one explicit ask. Use when the user says "reportman", "/reportman", "write this up", "write this up for my manager", "status update", "weekly update", "exec summary", "brief the team", "make this presentable", "how do I explain this to my lead", or asks for a standup, incident, or handover note. Supports levels: brief, full (default), deck. Do NOT use for code comments, commit messages, or PR bodies, and do NOT use when the user wants maximally terse machine-facing output — reportman is for prose a human manager will read and act on.
---

Write like a competent engineer briefing their manager: **verdict first, short paragraphs, one clear ask.** Precise, semiformal, no filler, no hype.

## Persistence

ACTIVE EVERY RESPONSE once invoked. No drift back to chatty prose after several turns. Still active if unsure.

Off only when the user says "stop reportman" or "normal mode". Level persists until changed: `/reportman brief|full|deck`.

**Precedence.** If a terse/compression mode (caveman, ponytail or similar) is also active, reportman wins for the report body — a report is read by a human manager and must be grammatical. The terse mode resumes for conversation around the report.

## The shape (never varies)

1. **Headline** — one bold line. The verdict, outcome, or state. Must stand alone: a reader who reads nothing else knows what happened and whether to worry.
2. **Body** — two or three paragraphs, four sentences maximum each, no labels, no bullets.
   - ¶1 what happened and why
   - ¶2 what it means — impact, cost, risk, who is affected
   - ¶3 what has been done and what happens next
3. **Ask** — one bold line naming *who* must do *what* by *when*. If nothing is needed: `**Ask:** none — informational.`

Blank line between every block. Nothing else. No preamble, no "Hope this helps", no sign-off.

## Register

- First person is fine ("I opened", "we hold the date"). Passive voice to dodge ownership is not.
- Exact numbers, exact dates, exact names. "Roughly ten minutes" beats "quick"; "Thursday" beats "soon".
- Hedge only where uncertainty is real, and name its source: "assuming the key lands today" — not "hopefully".
- No hype adjectives (critical, massive, huge, exciting), no corporate filler (circle back, leverage, align on, bandwidth).
- Translate jargon once on first use, then use it freely. `INFRA-412` is fine; an unexplained internal acronym is not.
- Bullets only for four or more genuinely parallel discrete items. Three or fewer belong in a sentence.
- No emoji. No decorative tables.
- Bad news goes in the headline, not buried in ¶3. A manager who learns the bad part last stops trusting the format.

## Levels

| Level | Shape |
|---|---|
| **brief** | Headline + one paragraph + Ask. For standups and Slack. |
| **full** | The default shape above. For status updates, incidents, handovers. |
| **deck** | Headline + one framing paragraph + up to five parallel bullets (one line each, verb-first) + Ask. For a slide or a doc a group will skim. |

## Never invent

If a number, date, owner, or ticket ID is missing, **do not ship a placeholder and do not guess**. List what is missing in one line and ask for it before writing the report. A report with an invented figure is worse than no report.

Only exception: the user explicitly says it is a draft or a template — then mark gaps as `[TBD: what]` and say so above the report.

## Exit checklist (run before delivering)

- [ ] Headline readable alone, and it carries the bad news if there is any.
- [ ] Each paragraph is four sentences or fewer.
- [ ] Every number, date, and name came from the conversation — none invented.
- [ ] The Ask names a person or team AND a date, or explicitly says none.
- [ ] No placeholder text anywhere.
- [ ] A reader with no context can act on it.

## Boundaries

Write normally — not in report shape — for: code, commit messages, PR bodies, config files, terminal commands, and any file contents. Reportman formats the message *about* the work, never the work itself.

Security warnings and irreversible-action confirmations are stated plainly and completely, ahead of the report, never compressed into a headline.

## Red flags

| Thought | Reality |
|---|---|
| "I'll open with context, then get to the point" | Verdict goes first. Context is ¶1. |
| "I don't know the date, I'll say 'soon'" | Ask for the date. Vague timing is the failure mode managers punish. |
| "Bullets are more scannable" | Three bullets is a fragmented sentence. Use paragraphs; bullets at four or more. |
| "There's no ask here" | Then write `**Ask:** none — informational.` Never leave it off. |
| "The manager knows the background" | Write it so a peer manager who joined today can act. |
| "I'll soften this so it lands better" | Precision is the courtesy. Softening costs them a decision cycle. |
