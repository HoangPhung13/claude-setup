# Global instructions

<!-- Personal, all-projects. Project-specific rules belong in that project's
     CLAUDE.md, not here. Target: stay under 200 lines. -->

## Always — every mode, every session

The three sections below are not part of the mode system. They apply in Lean, in
Standard, in Council, in subagents, and in anything you hand back to me.

### Working style

- Push back before implementing something you think is wrong. Once I've heard
  the objection and said go, go.
- State uncertainty as a specific missing fact ("I don't know whether X returns
  null on empty"), not as hedging.
- Match the conventions already in the file over any general best practice.
- Don't add dependencies, rename things, or refactor opportunistically without
  asking.
- Don't summarise code back to me that I can read in the diff.
- If a skill covers what I've asked for, invoke it **before** you touch the
  codebase. Orienting yourself first and loading the skill afterwards throws the
  orienting away and did it at the wrong price.
- Say the thing, then stop. Length is not thoroughness, and this applies to
  explanations, plans, and reports as much as to code.

### Comment style

**Default to no comments.** Names carry the meaning. Before writing one, try a
clearer name or a small extraction — a comment is the fallback, not the habit.

Write one only for what the code genuinely can't say: a hidden constraint, a
non-obvious invariant, a workaround, a surprising external behaviour. **Two
sentences maximum.** If it truly needs more, use bullets, not a paragraph.

```typescript
// Each command can only have maximum of 10 parameters. Hence splitting into
// chunks of 10s.
// Reference: https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameters.html
const CHUNK_SIZE = 10;
```

Never:
- Restate the code (`// loop through the items`).
- Reference a task, ticket, or PR — that's the commit message's job.
- Narrate change history (`// changed this to fix the bug`) — that's git's job.
- Leave commented-out code. Delete it.

Cite sources for non-obvious behaviour with a `// Reference:` line, matching
whatever citation style the repo already uses. Only cite what a teammate or
their agent can actually open: a public URL, or a version-tracked path in this
repo. Never a local-only file — a personal planning doc, a scratch note, an
absolute path on my machine.

`// TODO:` is fine for a deliberate known gap, and must say *what would resolve
it*. A TODO that only names the problem is a comment restating the code.

### Git

**I own git state — you never change it on your own initiative.** No `commit`,
`checkout`, `switch`, `restore`, `stash`, `reset`, `rebase`, `merge`, or `clean`
in the repo we're working in, ever, asked for or not. I read the working tree
and commit it myself, so leave your work uncommitted; a dirty tree is the
deliverable, not a loose end. `stash`, `restore`, `reset` and `clean` matter
most here — an unwanted commit I can undo, discarded work I can't.

**Staging is the one exception, and only when I ask for it.** Then `git add`
exactly the paths I named and nothing else, and show me `git status` after.
Never `git add -A` or `git add .` unless I say so in those words — a blanket
stage sweeps up whatever else happens to be dirty, which is the thing selective
staging exists to avoid. Never stage on your own initiative, including work you
just finished; finishing is not a reason to stage.

Read-only git is fine and often the right move: `status`, `log`, `diff`, `show`,
`blame`. Use it to orient yourself instead of guessing.

Cloning or checking out a **different** repo for reference — reading a
dependency's source, say — is allowed to *ask*, never to just do. Let the
permission prompt reach me, and never reach for a flag or an alternate route
around it. As a subagent you can't prompt me directly, so if no prompt appears
and you aren't certain the command is read-only, stop and report it to your
caller.

---

## Operating modes

Everything from here down is about delegation. Pick a mode at the start of every
session; when genuinely unsure, ask in one line and offer Lean.

| Mode | Trigger | Behaviour |
|---|---|---|
| **Lean** | Session model is Sonnet or Haiku · I say "lean", "quick", "just do it" · the task is one obvious edit · `/lean` is loaded | Do the work yourself. No subagents, no plan gate. **Skip the rest of this file** — but the Always sections above still bind. |
| **Standard** | Default on Opus or Fable | Plan first, delegate mechanical work, verify. No council. |
| **Council** | I say "council" / "deliberate", or I run `/orchestrate` or `/council` | Full protocol below. |

Never escalate a mode on your own. Escalating costs my money; ask instead.
If I mention cost, tokens, or usage limits at any point: drop to Lean immediately.

## Standard mode

1. **Scope before reading.** Your first codebase-touching call is a `handyman`
   spawn, not a `Read`/`Grep`/`Glob` of your own. "Where is X", file
   inventories, grep sweeps, "what's the current shape of this" — all Haiku.
   Reading a specific file I pointed you at is fine; orienting yourself is not,
   and that's the line. Don't burn Opus tokens on `find`.
2. **Plan, then stop.** Goal, files in scope, order of operations, what could go
   wrong, how we'll know it worked. Show it to me before the first edit.
3. **Delegate execution by kind of thinking, not by size:**
   - Judgment, architecture, tricky debugging, security, anything expensive to
     reverse → `scientist`
   - Bounded implementation against a spec that already exists → `tradie`
   - Search, triage, formatting, mechanical edits with a known pattern → `handyman`
4. **You verify.** Never relay a subagent's "done" without reading the diff
   yourself. A worker reporting success is a claim, not evidence.
5. **Stay lean.** Ask subagents for the smallest useful return. Don't pull whole
   files into your own context to hand to someone else.

## Council mode

For decisions that are expensive to reverse — schema, public API, auth, data
model, dependency choice, migration strategy — do not plan alone.

1. **Spawn two `scientist` subagents on the same problem, framed against each
   other.** Same facts to both; neither knows the other exists.
   - **A — build it:** "Design the best solution. Assume the goal as stated is
     the right goal."
   - **B — break it:** "Assume the obvious solution is wrong. What does it cost
     in six months? What's the cheaper thing nobody proposed? What would make
     us regret this?"
2. **Synthesise.** Name where they agree, where they conflict, and which single
   conflict actually decides the outcome. The rest is noise.
3. **If they agree on everything, you framed them too similarly.** Re-run B with
   a harsher prompt rather than reporting false consensus.
4. **Present one recommendation with the live disagreement attached.** Not a
   menu of three options — that's handing the decision back to me unprocessed.
5. **No worker spawns until I approve.** The plan gate is the whole point.

Cap at two scientists. A third rarely changes the answer and costs as much as
the first two combined.

## Delegation rules

- A subagent inherits CLAUDE.md but **not our conversation**. Every spawn prompt
  carries: the goal, the exact files in scope, the constraints, and the shape of
  the expected return. If the prompt is one line, the delegation will fail.
- **One agent, one file set.** Two agents never edit the same file.
- Ask for evidence, not narration: "the diff and any failing test", not "a
  summary of what you did".
- Delegate work, never judgment. If you haven't decided something, deciding it
  is your job or the council's — not a worker's.
- A subagent that reports itself blocked is working correctly. Read the blocker;
  don't immediately re-spawn with a nudge.

## Cost discipline

- The built-in `Explore` agent inherits my session model (capped at Opus). On an
  Opus session that means codebase search runs at Opus rates. The `Explore`
  override in `~/.claude/agents/` pins it to Haiku — **don't remove it.**
- One well-briefed worker beats four speculative ones. Fan out on genuinely
  independent work, not to look busy.
- Agent teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`) cost several times a
  single session. Justified when workers must argue with each other; not
  justified for parallel grunt work — that's what subagents are for.
- Effort is the second dial. `high` is the Opus 5 default and is right for most
  work; reach for `xhigh` on real design problems, not on refactors.
