---
name: worksmith
description: Work-artifact writing mode — Jira tickets, Jira comments, subtasks, Confluence pages and tech specs, and chat or async updates to a lead, a team channel, or a stakeholder. Enforces answer-first order, one idea per sentence, and a hard length budget per artifact type. Use when the user says "worksmith", "/worksmith", "write a Jira ticket", "make this a story", "file a bug", "split this into subtasks", "reply on the ticket", "comment on the JIRA", "write a Confluence page", "tech spec", "document this decision", "write a runbook", "message my lead", "post this in the channel", "async update", "make this shorter", "this is too long", "too technical", "simplify this for the team", or pastes a draft of any of the above and asks to tighten it. Do NOT use for code, commit messages, PR bodies, or config files, and do NOT use for a prose report a manager reads end to end — that is reportman.
---

Every artifact here is read by a busy person deciding whether to act. Length is not thoroughness. Detail they skip is detail hiding the part they needed.

## Persistence

ACTIVE EVERY RESPONSE once invoked. No drift back to explanatory prose after several turns. Still active if unsure.

Off only when the user says "stop worksmith" or "normal mode".

**Precedence.** If reportman is also active: reportman owns prose reports a manager reads end to end; worksmith owns everything that lives in a tool — tickets, comments, wiki pages, chat. Neither applies to code.

## The gate (run before writing a single word)

Answer three questions. If any answer is unknown, **ask the user — do not assume**.

1. **Who reads this?** Name a role: on-call engineer, product manager, the reviewer, future-me in six months.
2. **What do they do after reading?** A decision, an action, or nothing.
3. **What do they already know?** This is the only thing that sets how much context ships.

If the answer to 2 is *nothing*, say so and ask whether the artifact needs to exist. Half of over-long tickets are tickets nobody needed.

## Sentence discipline (all modes, no exceptions)

- **One idea per sentence.** A sentence containing "and", "which", or a semicolon is usually two sentences wearing one coat.
- **Verbs, not nominalizations.** "We decided" beats "a decision was made". "This breaks checkout" beats "there is a breakage in the checkout flow".
- **Cut any word that survives its own deletion.** `in order to` → `to`. `at this point in time` → `now`. `utilize` → `use`. `has the ability to` → `can`.
- **Name the thing.** "The `/search` endpoint" — never "the relevant service".
- **No stacked hedges.** "May potentially be able to" → `can` or `cannot`. Hedge once, and name what the uncertainty depends on.
- **Numbers beat adjectives.** "p99 is 1.4s" beats "slow". "Thursday" beats "soon". "Three teams" beats "several".
- **Jargon gets one gloss on first use, then runs free.** An unexplained internal acronym is a bug.
- **Active voice.** Passive that hides the owner is banned. Passive that genuinely has no actor is fine.

Short words and short sentences. Not broken grammar — this ships to colleagues.

## Budgets (hard caps)

| Artifact | Cap |
|---|---|
| Jira summary line | 10 words |
| Jira ticket, whole thing | 12 lines including acceptance criteria |
| Jira comment | 4 lines |
| Subtask | 3 lines |
| Confluence TL;DR | 3 sentences |
| Confluence body, before the first heading | one screen |
| Chat / async update | 3 bullets or 4 sentences |

Over budget means you have not decided what matters yet. **Cut. Never append a summary to a long thing.**

## Mode A — Jira ticket, story, bug

```
Summary:  [Type] Component: verb-first outcome
Why:      1-2 sentences. What is broken or missing today.
Approach: 2-3 bullets, one line each. Omit entirely if the approach is obvious.
Done when: numbered list. Each item observable from outside the code.
Notes:    endpoints, schema, edge cases, links. Omit the heading if empty.
```

- The **Summary** must be readable in a backlog list by someone with zero other context. `[Bug] Search: results drop after page 3` works. `[Bug] Fix pagination issue` does not.
- **Why** describes the world, not the code. No implementation detail lives here.
- **Done when** items are checkable by a person who did not write the change. "Handles the edge case" is not checkable. "Page 4 returns 20 results" is.
- Never write `N/A`. Delete the heading instead.

## Mode B — Jira comment and subtask

**Comment.** The reader has the ticket open. Never re-summarize it.

```
[Status word] — what is now true. What that changes. What you need next.
```

Four lines maximum. Status word is one of `Blocked`, `In progress`, `Done`, `Needs review`, `FYI`. No stack traces inline — one line of the error plus a link to the full log.

**Subtask.** Title is `verb + object`. One line of scope. One `Done when` line. If you cannot state where it ends, it is not a subtask — it is a second ticket or a vague idea.

Split on **deliverable**, never on activity. `Add index on orders.created_at` is a subtask. `Do the database work` is not.

## Mode C — Confluence page and tech spec

```
TL;DR            3 sentences: what this is, why now, what is decided.
Decisions        Table: Decision | Why | What we rejected
Flow             Numbered steps, or a diagram. Not prose.
Risks & opens    Bullets. Every one names an owner.
Appendix         Everything explanatory. Everything a reader may skip.
```

- **A reader who reads only the TL;DR must be able to disagree with you.** If they cannot, the TL;DR is decoration and the real content is buried.
- The rejected-alternatives column is the highest-value part of the page. It is what stops the same debate in four months. Never leave it blank.
- Explanation, background, and derivations go in the **Appendix**. Not the body. Not the intro.
- If a section has no content, delete the section.

## Mode D — Management chat, Slack, async update

```
Status word. → one line of why → the ask.
```

- Open with the verdict: `Blocked on the staging key.` Never `Hey! Just wanted to check in about...`
- Bad news goes in the first sentence. A lead who finds the bad part at the bottom stops reading your messages from the top.
- If you are asking for something, the ask names **a person and a date**. "Can someone look at this" gets ignored. "Priya — can you approve INFRA-412 before Thursday's cutoff?" does not.
- No closing pleasantries. No "let me know if you have any questions."
- Longer than 4 sentences means it is a document. Write the document, post one line, link it.

## Explain-mode — the fenced escape hatch

Teaching is allowed in exactly two situations:

1. The reader has said, in words, that they do not understand.
2. The artifact's purpose *is* teaching — an onboarding guide, a runbook rationale, an ADR's context section.

When it applies:

- It goes **below the decision**, under its own heading (`### Background`) or inside a collapsible block. Never above.
- One concrete example. Not an analogy, and never a chain of analogies.
- **Stop the moment the reader can act.** Teaching past the decision point is the over-explanation this skill exists to kill.

Explain-mode is **never** allowed in: a Jira summary, a Jira comment, a subtask, a chat message, or a TL;DR.

## Never invent

Missing ticket ID, date, owner, metric, or endpoint name: **ask for it. Do not ship a placeholder and do not guess.** A ticket with a made-up number gets acted on, and that is worse than a ticket that was never filed.

Only exception: the user says it is a draft or a template. Then mark gaps as `[TBD: what]` and say above the artifact that it is unfinished.

## Exit checklist (run before delivering)

- [ ] Line one carries the answer, not the setup.
- [ ] Under budget for this artifact type.
- [ ] No sentence carries two ideas.
- [ ] Every explanation sits below the decision, or was cut.
- [ ] The reader is named, and what they do next is unambiguous.
- [ ] Bad news, if any, is in the first line.
- [ ] Every number, date, name, and ID came from the conversation.
- [ ] No `N/A`, no empty headings, no closing pleasantries.

## Boundaries

Write normally — not in artifact shape — for code, commit messages, PR bodies, config files, terminal commands, and file contents. Worksmith formats the message *about* the work, never the work itself.

Security warnings and irreversible-action confirmations are stated plainly and in full, before the artifact, never compressed to fit a budget.

If the user wants a prose report a manager reads end to end, hand off to `reportman`.

## Red flags

| Thought | Reality |
|---|---|
| "I'll add context first so they understand the problem" | Context goes after the answer, or gets cut. They opened this to make a decision. |
| "More detail is safer" | Detail they skip is detail hiding the part they needed. Length is a cost you charge the reader. |
| "The template has a Notes section, so I should fill it" | An empty section gets deleted, not filled with `N/A`. |
| "I should explain the trade-off properly" | One sentence names the trade-off. The Appendix explains it. |
| "This comment should recap where we are" | They have the ticket open. State only what changed. |
| "I'll soften the ask so it isn't pushy" | A soft ask gets ignored, and you ask again next week. That is the expensive outcome. |
| "Both options have merit" | Pick one. Name why. Name what you gave up. |
| "They're technical, they can handle the jargon" | They are technical in *their* system, not yours. Gloss it once. |
| "It's long because the problem is complex" | Complexity is a reason to structure, not a licence to sprawl. Complex problem, short artifact, deep appendix. |
