# Sharing skills and plugins with your team, securely

Three ways to distribute, in increasing order of power and of risk. Pick the least powerful one that does the job — that is the whole security strategy in one sentence.

---

## First: know what you are handing someone

This distinction drives every other decision on this page.

| | **A skill** | **A plugin** |
|---|---|---|
| What it is | A markdown file with frontmatter | A package: skills + slash commands + subagents + **hooks** + MCP server configs |
| What it does | Text injected into the model's context. Influences behaviour. | All of the above, **plus it can execute code on the machine** |
| Worst case | The model is steered into doing something you did not want — including running a command it suggests | Code runs automatically at session start, on every prompt, or after every tool call, with your user's permissions |
| Review effort | Read the markdown. It is all there. | Read the markdown **and** every script it references |

A real example from a plugin installed on the author's machine — its `.claude-plugin/plugin.json` declares:

```json
"hooks": {
  "SessionStart": [{ "hooks": [{ "type": "command",
    "command": "node \"${CLAUDE_PLUGIN_ROOT}/src/hooks/caveman-activate.js\"" }] }]
}
```

That runs on every single session start, before anyone types anything. It is a benign plugin. The point is that **installing a plugin is consenting to run its author's code**, and that consent is easy to give without noticing.

Skills carry no such mechanism. That is why the recommendation below is: share skills, and be deliberate about plugins.

---

## Route 1 — commit skills into the repo (default; use this unless you have a reason not to)

Put them in the project itself:

```
your-repo/
  .claude/
    skills/
      reportman/SKILL.md
      triage/SKILL.md
```

Commit. Open a PR. Teammates pull, and the skills are there.

**Why this is the secure option, and not a compromise:**

- Distribution is your existing code review. A skill arrives the same way a function does — as a diff someone approved.
- Access control is your existing repo permissions. No new system, no new sharing surface, no separate list of who can see what.
- It is scoped. The skill exists in the project it was written for, not on everyone's machine in every context.
- It is versioned and revertible. A bad skill is `git revert`, and you can see who added it and when.
- It travels correctly. New hire clones the repo, has the team's skills, no onboarding step.

**Make the diff readable.** In the PR description, say what the skill changes about the agent's behaviour, not just that it exists. `SKILL.md` files are prose and reviewers skim prose. Call out anything that instructs the agent to run commands, read outside the repo, or contact a network endpoint.

---

## Route 2 — an internal plugin marketplace (many repos, one install)

When the same skills belong in twenty repos, stop copying and publish a marketplace. A marketplace is **just a git repo** with a manifest at `.claude-plugin/marketplace.json`:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "acme-internal",
  "description": "Internal Claude skills for the platform team.",
  "owner": { "name": "Platform Team", "url": "https://github.com/acme" },
  "plugins": [
    {
      "name": "acme-standards",
      "description": "House reporting, triage, and handoff conventions.",
      "source": "./",
      "category": "productivity"
    }
  ]
}
```

The plugin itself needs `.claude-plugin/plugin.json` alongside its content:

```
acme-standards/
  .claude-plugin/
    marketplace.json
    plugin.json
  skills/
    reportman/SKILL.md
    triage/SKILL.md
  commands/          ← optional slash commands
  agents/            ← optional subagents
```

A minimal `plugin.json` with **no hooks and no MCP servers** is the safe shape:

```json
{
  "name": "acme-standards",
  "description": "House reporting, triage, and handoff conventions.",
  "author": { "name": "Platform Team" }
}
```

Teammates then add it once:

```
/plugin marketplace add acme/claude-standards
```

Claude Code records it against the org and repo, and everyone installs from the same source. Private repos work — access is whatever GitHub already grants that person, so an ex-teammate loses the marketplace when they lose the repo.

**Rules that keep this safe:**

- **Ship skills-only plugins by default.** No `hooks`, no `mcpServers` in `plugin.json` unless there is a concrete need. A skills-only plugin has the same risk profile as Route 1 with better distribution.
- **The moment a plugin gains a hook, it becomes code.** It needs a code reviewer, not a docs reviewer. Hooks run automatically and unattended — that is their purpose and their risk.
- **Branch protection with required review on the marketplace repo.** Anyone who can push to it can run code on every teammate's machine at their next session start. Treat write access to that repo as production access, because that is what it is.
- **Pin to a tag, not a moving branch,** for anything with hooks. Tracking `main` means an upstream change alters behaviour on every machine with no one approving it.
- **Announce updates.** "We bumped the marketplace, here is the diff" — so an unexpected behaviour change is traceable to a decision rather than mysterious.

---

## Route 3 — workspace or org-level distribution

Claude team and enterprise plans have admin-managed distribution — skills and connectors published centrally, without every person installing anything. The exact surface and which plan gates it varies, so **check with whoever administers your workspace** rather than assuming from this page.

Where it exists it is the right answer for organisation-wide standards, because the control point is an admin rather than a convention. The security questions to ask your admin are the same three regardless of the UI:

1. Who can publish, and does publishing require a second approver?
2. Does the published artifact contain hooks, MCP servers, or anything that executes?
3. How do people find out when it changes?

---

## Security rules that apply to all three routes

**Never put a secret in a skill, a plugin, or a marketplace repo.** Not an API key, not a token, not a connection string, not an internal URL that acts as a credential. These files get copied, forked, cached, and pasted into chats. Reference environment variables; never literal values. Run a secret scanner in CI on the repo — a pre-commit `gitleaks` hook takes minutes to add and catches the mistake before it is permanent.

**Review every third-party skill before it reaches anyone else.** A skill is prose that steers an agent, so a hostile one does not need to look like code to be dangerous. Read it fully. Specifically look for: instructions to run shell commands, to read files outside the project, to send data to a URL, to ignore or override the user's other instructions, or to suppress its own reporting. Any of those in a skill you did not write is a stop, not a note.

**Fork rather than track, for anything external.** Copy a public skill into your own repo and pin it. Then an upstream change is a diff you choose to pull, not a surprise. This costs you nothing and removes an entire class of supply-chain risk.

**Never redistribute a plugin you have not read end to end,** including scripts under paths the manifest references. "It came from a colleague" is not review. Neither is "it has stars".

**Treat write access to a shared marketplace as production access.** Branch protection, required review, no direct pushes, and a small owners list.

---

## Review checklist before accepting a contributed skill

- [ ] Does the `description:` frontmatter describe what it actually does? That line is the trigger — a misleading one makes the skill fire in situations nobody expected.
- [ ] Does the body instruct the agent to run commands, read outside the project, or reach the network? If yes, is that stated in the description and justified?
- [ ] Does it try to override the user's standing instructions, or tell the agent to skip confirmations?
- [ ] Any secret, internal hostname, personal path, or customer name in it?
- [ ] Does the folder name match the frontmatter `name:`?
- [ ] For a plugin: does `plugin.json` declare `hooks` or `mcpServers`? If yes — has an engineer read every referenced script?
- [ ] Is it pinned to a tag or commit rather than a moving branch?

---

## What to actually do

Most teams need Route 1 and nothing else. Commit the skills into the repo, review them like code, and stop there.

Move to Route 2 when copy-paste across repos becomes the bottleneck — and keep those plugins skills-only. Add hooks only when there is a specific job they do, and route that change through an engineer who reads the script.

Route 3 when it is an organisational standard rather than a team preference.
