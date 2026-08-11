# Install

Three routes. Pick by what the machine lets you do.

---

## Route A — clone (fastest, needs git access to this repo)

```bash
git clone <this-repo-url> claude-skills-public
```

Then copy the skills you want to a target directory (see [Where they go](#where-they-go)):

```bash
cp -r claude-skills-public/skills/reportman ~/.claude/skills/
```

---

## Route B — copy-paste, no clone (works anywhere you can read a web page)

Nothing is downloaded. You read one page and retype/paste it.

1. Open [`ALL-SKILLS.md`](ALL-SKILLS.md). It contains every skill file on one page, each preceded by a `FILE:` line giving its exact destination path.
2. For each skill you want:
   - Create the folder `<target>/skills/<name>/`
   - Create `SKILL.md` inside it
   - Paste everything between that skill's `FILE:` line and the next one
3. Restart your Claude session.

Paste the file **exactly**, including the `---` frontmatter fences at the top. Those three dashes are not decoration — without them the file is treated as ordinary markdown and the skill will never load.

If your editor is configured to strip trailing whitespace or reformat markdown on save, turn that off for these files or check the frontmatter survived.

---

## Route C — ask the assistant to write them

If you have a Claude session with file-write access on the target machine, paste the contents of `ALL-SKILLS.md` into it and say:

> Create each of these files at the exact path given in its FILE: line. Content verbatim, no edits, no summarizing.

Verify afterwards with the check below. Do not skip the verification — "I created the files" is not the same as the files being loadable.

---

## Where they go

| Target | Path | Scope |
|---|---|---|
| **Personal** | `~/.claude/skills/<name>/SKILL.md` | Every project on this machine. On Windows: `C:\Users\<you>\.claude\skills\<name>\SKILL.md` |
| **Project** | `<repo>/.claude/skills/<name>/SKILL.md` | That repo only. Travels with the repo, works for anyone who clones it. |

**If you cannot write to your home directory, use the project path.** It is not a fallback — it is the better option when the skill is tied to one codebase, and it is the one that survives a machine rebuild.

Final layout either way:

```
<target>/
  skills/
    reportman/SKILL.md
    triage/SKILL.md
    save-context/SKILL.md
    prompt-forge/SKILL.md
    browser-copilot/SKILL.md
```

One folder per skill. The folder name should match the `name:` in the frontmatter.

---

## Verify it actually loaded

Do not trust the file being on disk. Start a **new** session and test the trigger:

| Skill | Say this | Expect |
|---|---|---|
| `reportman` | "reportman — write up that the staging deploy is blocked on a missing key" | Bold headline, two or three short paragraphs, a bold `**Ask:**` line |
| `triage` | "triage this: intermittent 500s on /checkout, started after yesterday's deploy" | A `SEV / PRI` block with `STATUS`, `OWNER`, `NEXT` |
| `save-context` | "save context" | Offers to write a handoff file, and tells you it cannot clear the conversation itself |
| `prompt-forge` | "turn this into a prompt: summarize meeting notes" | A fenced prompt with `CONTEXT / TASK / INPUT / OUTPUT / TONE / REASONING / SPEED / STOP` |
| `browser-copilot` | "use my open tabs to answer X" | Enumerates tabs once as a numbered manifest, then reads |

**If nothing happens:** the frontmatter is the usual cause. Check that the file starts with `---` on line 1, has `name:` and `description:`, and closes with `---`. Second most common cause: you did not start a new session — skills load at session start.

---

## Uninstall

Delete the skill's folder. There is no state, no config entry, and no cache — the file being gone is the whole uninstall.

---

## Changing them

These are meant to be edited. The one line to be careful with is `description:` in the frontmatter — that is what the model matches against to decide whether to load the skill. Rewrite the body freely; touch the description deliberately, and re-run the verification above afterwards.

If you fork a skill for your own voice, keep the red-flags table. It is the part that stops the skill drifting back into generic behaviour after a dozen turns.
