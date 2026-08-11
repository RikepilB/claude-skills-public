---
name: save-context
description: Save the state of a conversation to a durable file before clearing, compacting, or closing it — a handoff with a paste-ready resume block, plus durable memories for later sessions and other agents. Use when the user says "save context", "/save-context", "save this session", "export this", "write a handoff", "I'm going to clear", "before we compact", "wrap this up", "remember this for next time", "carry this to a new conversation", or is about to switch machines, models, or agents. Redacts secrets before writing. Do NOT use to read existing context for orientation (that is a catch-up job), and note it cannot itself delete or clear conversations — it prepares the artifact and tells the user how.
---

A conversation is volatile. The file you write is not. Save the state, then the conversation is safe to clear.

## Where it goes

**Always inside the current project directory, never the home directory or a system temp path.** A machine may deny writes outside the repo, and a handoff that lives outside the project is one nobody finds. Use the first of these that applies, so a project keeps one convention:

1. `docs/handoff/` — if that directory already exists.
2. `context/` — otherwise, create it at the repo root.

If the project directory is not writable, stop and say so rather than silently writing elsewhere. Offer to print the file contents in the conversation so the user can save it themselves — a handoff the user pastes into a file is still a handoff; one written to a path they will never look at is not.

Write `docs/handoff/<YYYY-MM-DD>-<short-slug>/HANDOFF.md` (or the `context/` equivalent). One folder per session. An index file at the root of that directory — `HANDOFF.md` or `README.md` — carries a rolling current state plus a permanent list of sessions.

**Append, never overwrite.** Session folders are immutable once written. Only the index's `## Current state` is ever replaced. If a past session's record is wrong, add a new session that corrects it — do not edit history.

## Redact before writing (do this first, not last)

The file may be committed, synced, or read on another machine. Strip before writing:

- API keys, tokens, passwords, connection strings, `.env` contents
- Internal hostnames, IPs, and private URLs, unless the repo already contains them
- Customer names, personal data, anything under NDA
- Full stack traces containing paths that expose account or machine names

Replace each with `[redacted: what it was]` so the next reader knows something existed. If unsure whether a value is sensitive, redact it — the cost of over-redacting is one question, the cost of under-redacting is permanent.

## Session file — fixed sections

```markdown
# <Session name> — <YYYY-MM-DD>

## Goal
<What this session set out to do. One or two sentences.>

## Current state
<Where things actually stand right now. Honest — "half migrated, tests failing"
 is more useful than "in progress".>

## Done
<One concrete line per completed thing: file, command, PR, decision.
 NOT a narration of the conversation.>

## Files in flight
<Paths currently being edited, and what is half-done in each.>

## Decisions
<Each decision + the reason. The reason is the part that stops it being
 re-litigated in three weeks.>

## Failed attempts
<What was tried and did not work, and why. This is the highest-value section —
 it is the only thing a fresh session cannot re-derive.>

## Next steps
<Ordered. First item should be startable without re-reading anything else.>

## Resume prompt
<A paste-ready block. See below.>
```

## The resume prompt (the part that makes it worth writing)

End every session file with a fenced block the user can paste into a brand-new conversation to be fully oriented in one message. Write it addressed to the next agent, self-contained, no references to "the previous conversation":

```
Project: <name> at <path>
Goal: <goal>
State: <2-3 sentences on where things stand>
Read first: <2-4 file paths, most important first>
Constraints: <anything that must not be broken or changed>
Do not retry: <the failed approaches, one line each>
Start with: <the single first action>
```

Keep it under roughly 200 words. It replaces the conversation, so it must be readable in one pass.

## Durable memories (facts that outlive this project)

Some facts should survive beyond the session file — preferences, conventions, standing corrections, environment quirks. Those go in the project's own long-term instruction file (`CLAUDE.md`, `AGENTS.md`, or a `MEMORY.md` next to it), not in the session folder, because a session folder is history and is never read by default.

Write a memory only if all three hold:

1. It will still be true next month.
2. It is not already derivable from the code, the README, or the commit history.
3. Getting it wrong would cost real time.

Format each as one line: **the fact — why it is true — what to do about it.** Convert relative dates to absolute ones ("since 2026-08-10", never "last week"). Delete memories that turn out to be wrong; a stale memory is worse than a missing one because it is trusted.

Everything else stays in the session file. Do not promote conversational detail into standing instructions — that is how instruction files rot.

## Then, and only then, clear

**This skill cannot delete, clear, or compact a conversation, and cannot export a transcript.** Those are user actions. After the file is written and the user has confirmed it looks right, tell them plainly what to do next:

1. Export the full transcript if the harness supports it, into the same session folder as `transcript.md`, for deep dives later.
2. Verify the session file and the resume prompt are on disk.
3. Then clear, compact, or delete the conversation.

Never tell the user it is safe to clear before the file exists on disk. If the write failed, say so and stop.

## Red flags

| Thought | Reality |
|---|---|
| "I'll paste the whole conversation in" | One line per finished thing. The transcript is the archive; this is the digest. |
| "I'll overwrite the index with the new state" | Only `## Current state` is replaced. The session list is permanent. |
| "Failed attempts aren't worth recording" | They are the only content a fresh session cannot rediscover. Always record them. |
| "I'll fix that old session file" | Session folders are immutable. Write a new one that corrects it. |
| "There's no handoff directory, so I'll skip it" | Create `context/`. Never drop the state because the convention is missing. |
| "The user said clear, so we're done" | The file must exist on disk first. Confirm, then clear. |
| "This detail is interesting, into MEMORY.md it goes" | Memories are facts that outlive the project. Interesting is not durable. |
