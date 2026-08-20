---
name: orchestrate
description: Plan a PR as a series of commits, then execute one commit at a time — handyman recon, tradie build, verify, hand back for my review and my commit. Stops at every commit boundary. Reuses a /council synthesis if one is in this conversation, and defers to /council if the decision is expensive to reverse.
when_to_use: 'When I ask for it in words — "orchestrate X", "plan this as commits", "plan and build X", "break this into a PR", or I otherwise name commits as the unit of work. Also when I say to carry on with a commit and a plans/*-ledger.md exists. Invoke it as your FIRST action, before reading or grepping anything: phase 1 does all recon through handyman on Haiku, so any orienting you do first is thrown away and was paid for at Opus rates. You do not need to understand the task before invoking — working that out is what the skill is for. Fire on my language only; never invoke because you judged a task big enough to deserve the protocol, which is escalating a mode on your own and costs me money.'
argument-hint: "[what you want built], or [commit N] to resume"
allowed-tools: EnterPlanMode, ExitPlanMode, AskUserQuestion
---

Task: $ARGUMENTS

This is the execution engine. Deliberation lives in `/council`; phase 2 decides
whether you need it, already have it, or can skip it.

The unit of work here is **a commit**, not a task. One plan describes a PR as a
series of commits; you execute them one at a time and stop after each so I can
review, amend, and commit myself. **You never commit.** The tree you hand back
is dirty on purpose.

## Phase 0 — Resume, or start

**Look for an existing ledger first:** `plans/*-ledger.md` in the repo. If
`$ARGUMENTS` names a commit ("commit 2", "next commit", "carry on"), or a ledger
with unfinished commits plainly covers what I'm asking for, this is a **resume**.
Read it, skip to phase 4, and start at the first commit not marked `done`. Do not
re-plan, do not re-gate, do not call `EnterPlanMode`. Re-gating a plan I already
approved wastes a turn and invites you to quietly redesign it.

If more than one unfinished ledger could match, name them in one line and ask
which. Don't guess, and don't assume the most recently modified one is mine — I
run several of these at once.

Otherwise this is a **new plan**. **Call `EnterPlanMode` before anything else.**
Phases 1 through 3 are read-only by design, and plan mode enforces that at the
tool layer instead of trusting you to remember. It also gives the phase 3 gate a
real approval prompt rather than a message that just stops.

## Phase 1 — Ground yourself (cheap)

Delegate to `handyman` — several in parallel if the surface is wide. Do not read
the codebase yourself yet.

Cover at least: the files that would change, what already exists that solves
part of this, the conventions in the files you'd be touching, and any sibling
implementation worth copying rather than inventing. If I named another repo,
send a handyman there too.

Be generous here. This is the cheapest context you will ever buy, and every gap
you leave gets rediscovered later by a tradie or a scientist at several times
the price.

If this reveals the task is trivial or already done, say so and stop. Ending
here is a success, not a failure of the protocol.

## Phase 2 — Do we already have a decision?

Three branches. Pick one and say which, in one line, before continuing.

**A — a `/council` synthesis for this task is already in the conversation.**
Use it. Do not re-deliberate. Any `/council` instruction about not building
supersedes here — that constraint was scoped to its own turn. Go to phase 3 and
write the plan from the recommendation.

**B — no council, and the decision is expensive to reverse.** Schema, public API,
auth, data model, dependency choice, migration strategy. Stop and tell me to run
`/council <the decision>` first, naming the specific thing that needs deciding.
One line, no plan attached. I'd rather spend a command than unwind a migration.

**C — no council, and the approach is already settled.** Decided in conversation,
or obvious, or mechanical. Proceed. Say which, so I can disagree if I think
you've mislabelled a B as a C.

## Phase 3 — Plan the commit series, and gate

Write, for me, in under 500 words:

1. **The decision this rests on**, in one line, and where it came from — the
   council synthesis, our conversation, or your own call under branch C.
2. **The commit ledger.** The PR broken into commits, in order. Per commit:
   - a subject line, written as the actual commit subject I'll use
   - one line of intent — what it makes true that wasn't true before
   - the files it touches
   - how it gets verified
   - whether it can be split across parallel tradies (see below)
3. **What could go wrong**, and which commit is the point of no return.

Sizing rule: each commit is independently reviewable and leaves the tree green.
If a commit can't leave the tree green on its own, say so and say why — that's
sometimes correct, but it should be a decision, not an accident.

**Parallelism belongs inside a commit, never across them.** Commits are
sequential and freely revisit the same files — commit 1 adds the function,
commit 3 wires it in. Within one commit, if the work splits into genuinely
disjoint file sets, run a tradie per set concurrently. Two tradies never hold
the same file at the same time. Building a wall, several tradies; the floor is
another day.

If you came from a council under branch A, do not re-argue the recommendation.
Restate it in one line and spend the words on the ledger.

No spawning, no edits, no "I'll get started on the uncontroversial part". Plan
mode blocks the writes; this line is about not trying.

### The gate

**Print the plan inline in chat first.** If it only exists in a plan file, I get
an approval prompt for something I haven't read, and "review" reads like a menu
option rather than an action. Plan in the message, then the prompt.

Then call **`ExitPlanMode`**. That is the gate. It is the native approval path:
it holds the session read-only until I answer, and unlike a multiple-choice
question it never auto-resolves if I walk away from the keyboard.

Do **not** use `AskUserQuestion` to ask whether the plan is good or whether to
proceed. That is precisely what `ExitPlanMode` does, and asking twice is worse
than asking once.

`AskUserQuestion` has one correct use here, **before** `ExitPlanMode`: a genuine
fork inside the plan where I have to pick and you can't. Two viable commit
orderings, a tradeoff with no dominant answer, a boundary you can't infer. Ask
it as a concrete choice — "cascade the delete or soft-delete and reconcile
nightly?" — never as a reference to "the plan", which I can't see yet.

Branch B in phase 2 is also a legitimate `AskUserQuestion`: *this looks expensive
to reverse* → **Run /council first** · **Proceed anyway** · **Let me reframe it**.

If plan mode is unavailable, fall back to a single `AskUserQuestion` —
**Approve** · **Adjust scope** · **Rethink** · **Plan only** — and say you're
falling back. **Approve** → phase 4. **Adjust scope** → revise and re-gate.
**Rethink** → stop, name the decision, tell me to run `/council`. **Plan only**
→ write the ledger and stop.

**One structural constraint.** `AskUserQuestion`, `EnterPlanMode`, and
`ExitPlanMode` are all stripped from subagents, so a worker can never gate on me
— it reports blocked and you ask. Never add `context: fork` to this skill; it
would move the whole flow into a subagent and silently delete the gate.

### On approval, write the ledger to disk

`plans/<slug>-ledger.md`, in the repo's gitignored `plans/` directory.

**You name it.** Summarise the PR into a short kebab-case slug — two to four
words, specific enough that I can tell it apart from the others in that
directory six weeks from now. `stripe-webhook-retries`, not `billing` or
`refactor`. Don't derive it from the branch: I reuse a branch across several
PRs when the branch name still makes sense. Check `plans/` first and pick
another slug if yours is taken, rather than overwriting.

Open the file with the PR's one-line goal, so the slug isn't the only clue about
what it is. Then one section per commit: subject, intent, files, verification,
status (`pending` · `handed back` · `done`), and an empty **Handoff**
subsection. This survives compaction and new sessions; the chat transcript does
not.

**Confirm `plans/` is actually ignored before writing** — `git check-ignore -q
plans/`. If it isn't, stop and tell me. I review and commit by hand, and a
ledger surfacing in `git status` mid-review is exactly the noise this workflow
exists to avoid.

## Phase 4 — The commit loop

Approving the plan exits plan mode, which is your signal to start. Not before.

Run these five steps for **one** commit, then stop. Do not begin the next commit
because the current one went well.

**1 — Recon (`handyman`, cheap).** Before every commit, including the first.
Ask for: the exact current contents and shape of the files this commit touches,
what's already there that overlaps, and `git log`/`git diff` since the last
commit in the ledger.

This step is load-bearing, not ceremony. Between commits I amend the work and
commit it myself, sometimes from a different session you can't see. **Your
memory of what the last tradie did is not the state of the tree.** Re-ground on
what's actually on disk, and if it contradicts the ledger, say so before
building anything.

**2 — Brief.** Turn the ledger entry, the recon, and the accumulated **Handoff**
notes from earlier commits into a tradie brief. Every brief carries: goal, exact
files in scope and what's off-limits, constraints and conventions, relevant
pitfalls inherited from previous commits, and the expected return shape. Workers
do not see this conversation — if it isn't in the brief, it doesn't exist.

**3 — Build (`tradie`).** One tradie, or several if the ledger marked this commit
splittable and the file sets are disjoint.

**You do not write the commit yourself.** Every file change in this step goes
through a tradie, including changes you could type from memory. The brief is
cheap because you did the planning; your reads and writes across the commit's
file set are not, and the wider the commit's radius the worse that trade gets.
If you catch yourself editing "just this one file first", you've already skipped
the spawn.

If something turns out to need judgment mid-flight — the spec is wrong, the
approach doesn't fit what's actually in the file — stop the tradie. Decide it
yourself if it's within the approved plan, bring it back to me if it isn't.
Never let a tradie improvise a design.

**When I comment mid-flight**, I'm watching the changes land and reacting. Route
my comment to the cheapest place that can still act on it:

- A tradie is still working, or the next one is about to be briefed → fold it
  into that brief. You're writing the brief anyway; this is close to free.
- Nothing is in flight and it's a couple of lines → just do it yourself.
- It changes what the commit is for, or reaches files outside its scope → stop.
  That's a ledger change, not a comment, and I should see the new shape before
  you build to it.

Either way it goes in the **Handoff** notes at step 5. A redirect I gave in
passing is invisible to the next tradie otherwise, and I won't remember I gave
it.

**4 — Verify, yourself.** Read the actual diff. Run the project's own checks. A
tradie's "done" is a claim; the passing test is the evidence. Do not relay a
report you haven't checked.

**5 — Hand back, and stop.** Give me, in this order:

- **Commit N of M — `<subject>`**
- **Changed** — one line per file. No code summary; I'm about to read the diff.
- **Verified** — the exact commands you ran and their output. If you ran
  nothing, say "nothing run" and why.
- **Proposed commit message** — **the subject line alone**, in a fenced block,
  ready to paste. This is mine to edit, so write it as a claim about the change,
  not a narration of the session. Add a body only when the subject genuinely
  cannot carry the scope; commits are atomic by construction here, so that
  should be rare. A body that restates the subject at greater length, or walks
  through what changed file by file, is noise — I'm about to read the diff.
- **Pitfalls for the next commit** — ambiguities you resolved and which way,
  workarounds left in place, anything deliberately deferred to a later commit,
  anything I redirected you on mid-flight, anything you found that the ledger got
  wrong. "Nothing" is a valid answer, and suspicious more than twice in a row.
- **Open / blocked** — anything unfinished, or "nothing".

Then write the pitfalls into the ledger's **Handoff** section for this commit,
mark it `handed back`, and **stop**. No "shall I continue?" — I'll say.

## Phase 5 — Corrections, while you're stopped

While you're stopped I'm reading the diff, and I'll often want changes before I
commit. **I say them as plain comments in this thread** — "change X", "actually
make it Y", "drop that bit". That is the normal path and it needs no command
from me: treat any comment arriving after a handback as a correction pass on
that commit. Don't wait for `/amend`, and don't suggest I run it.

A correction pass is not new work. No `EnterPlanMode`, no re-gate, no re-plan,
no widening the commit beyond what I named.

**Who does the edit** — the decision is already made, so this is only about cost:

- A few lines, in files already in your context → do it yourself.
- Anything wider — several files, a rename, a change you'd have to re-read the
  files to make → one `tradie` with a three-sentence brief. You have the context
  to write it in three sentences; it's your tokens on the edit that are worth
  avoiding, and a correction with a wide radius is still a wide radius.
- Needs a decision I haven't made, or reaches outside this commit's scope → stop
  and name it. That's a ledger change or a `/council`, not a correction.
  Amendments that quietly grow are how a commit stops matching the diff I
  reviewed.

Exactly what I asked for: no drive-by fixes, no reformatting, no renames I didn't
name. If you spot something else, one line at the end. Re-run the project's own
checks — the commit was green when you handed it back, it has to still be green.
Never commit, and never stage unless I ask.

Then append to this commit's **Handoff** section what I asked for and what
changed, one line each — that section is the only reason the next tradie will
know it happened. Reissue the **proposed commit message** if the correction
changed what the commit claims. Return short: changed files one line each, the
verification command and its result, the ledger line. No code summary; I'm
looking at the diff.

I may also edit or commit outside this session entirely, so when I say move on,
mark this commit `done` and go back to phase 4 step 1 — the recon pass is what
tells you which of those actually happened.

## Finishing

When the last commit is marked done, say so in one line, list the commit
subjects in order, and name anything in the ledger that was dropped or deferred.
Then delete the ledger file, or tell me why it should stay.
