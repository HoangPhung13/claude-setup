---
name: orchestrate
description: Plan a PR as a series of commits, then execute one commit at a time — handyman recon, tradie build, verify, hand back for my review and my commit. Stops at every commit boundary. Also runs review rounds against work already committed, as a triage-and-fixup-map loop. Reuses a /council synthesis if one is in this conversation, and defers to /council if the decision is expensive to reverse.
when_to_use: 'When I ask for it in words — "orchestrate X", "plan this as commits", "plan and build X", "break this into a PR", or I otherwise name commits as the unit of work. Also when I say to carry on with a commit and a plans/*-ledger.md exists, and when a code review comes back on work already committed on this branch — that is phase 6, and it fires even if the ledger is finished or gone. Invoke it as your FIRST action, before reading or grepping anything: phase 1 does all recon through handyman on Haiku, so any orienting you do first is thrown away and was paid for at Opus rates. You do not need to understand the task before invoking — working that out is what the skill is for. Fire on my language only; never invoke because you judged a task big enough to deserve the protocol, which is escalating a mode on your own and costs me money.'
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

**First, the house rules, and this one you read yourself.** Before any handyman
is briefed, `find` every `CLAUDE.md` and `AGENTS.md` under each repo root in
scope and read them verbatim:

```
find <root> \( -name node_modules -o -name vendor -o -name .git \) -prune -o \
  \( -name CLAUDE.md -o -name AGENTS.md \) -print
```

A subagent inherits only the rules this session had loaded when it was spawned,
which is this repo's hierarchy and nothing else. Pointing a handyman at another
repo buys it file access, not that repo's conventions, and a nested file under
`packages/*` is missed the same way even here whenever nothing has caused you to
load it yet. The recon lanes need these as much as the tradies do, so a lane
that goes out before this step is a lane working blind. Reference:
https://code.claude.com/docs/en/sub-agents.md

Setting `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` would load added repos
at startup instead. It is deliberately not set: it puts every repo's rules into
every session and every lane, with no way to tell afterwards which set a tradie
actually followed.

Delegate to `handyman` — several in parallel if the surface is wide. Do not read
the codebase yourself yet.

Cover at least: the files that would change, what already exists that solves
part of this, the conventions in the files you'd be touching, and any sibling
implementation worth copying rather than inventing. If I named another repo,
send a handyman there too, briefed on that repo's rules rather than this one's.

**Anything outside this repo gets read, not remembered.** The dependency's
actual source under `node_modules` or `vendor`, the version the lockfile
actually pins, the real response shape from a log line or a recorded fixture,
the vendor's current docs. If a commit will rest on how something behaves and
you know that only from training, it is a phase 1 gap, and it surfaces as a
tradie guessing wrong two commits later. Where you genuinely could not check,
write `unverified` next to the claim in the plan rather than smoothing over it.

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

**Name the branch this would build on**, in one line, from `git branch
--show-current`. If it is `main` or `master`, say so and hold at the gate: this
protocol produces a PR's worth of commits, and branching before commit 1 is
cheaper than untangling after commit 4. Branching is mine to do; ask for it,
never run it.

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

Open the file with the PR's one-line goal, so the slug isn't the only clue
about what it is, and the branch it was planned against.

Then a **status table**, before the detail: one row per commit with number,
subject, status, and a few words on the outcome. That table is the first thing
a resuming session reads and the only part of the ledger I'll skim mid-review,
so it has to be current rather than tidy. Update it at every status change, not
at the end.

Then one section per commit: subject, intent, files, verification, status, and
an empty **Handoff** subsection. This survives compaction and new sessions; the
chat transcript does not.

Record the resolved rules files too: their paths and mtimes, not their contents.
A resuming session inherits the set instead of rediscovering it, and re-reads
them at brief time so an edited rules file is never served stale.

**Statuses:** `pending` · `in progress` · `handed back` · `blocked` · `done`.
Write `in progress` when you start phase 4 step 1, not when you finish it. A
session that dies mid-commit otherwise leaves a `pending` row that lies about
the tree, and the next session rebuilds work already on disk. `blocked` carries
its reason in the outcome column.

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

Mark the commit `in progress` in the status table before you brief anyone.
The row is how a later session tells "not started" from "half built".

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

**The house rules go in verbatim** — the phase 1 file nearest this commit's file
set, not every file you found and not a paraphrase. One tradie, one repo: a
commit spanning two repos is two briefs, each carrying only its own repo's
rules. Where a repo's rules and my global `~/.claude/CLAUDE.md` disagree on
something local to that repo, the repo wins, and the brief says so.

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
move it to `handed back` in both the section and the status table, and **stop**.
If you are stopping short instead, the status is `blocked` and the row says why. No "shall I continue?" — I'll say.

## Phase 5 — Corrections, while you're stopped

This phase is about **one uncommitted commit** you just handed back. If the work
is already committed, or the changes span commits, that is phase 6 — stop here
and read it.

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

## Phase 6 — Review rounds, on committed work

A reviewer comes back — me, `/code-review`, `/inspect`, a PR thread — with
findings against work that is **already committed**. This is not phase 5 and not
a new plan. It is its own loop, and it is the one place in this skill where the
deliverable is a claim about *history* rather than a dirty tree.

**Which phase am I in.** Phase 5 is one uncommitted commit and a comment from
me. Phase 6 is a list of findings against commits already in the log. If the
tree is clean, or the ledger entries are `done`, or the findings span more than
one commit, you are in 6. The failure mode is drifting into phase 5 behaviour
because nothing else fits, and absorbing decisions phase 6 would have gated.

You cannot declare this from my message alone — every tell requires looking. So
step 1 opens with the cheap check and the declaration falls out of it.

**1 — Recon, every round, non-negotiable.** Two parts, in order.

First the cheap check, yourself, in one call: `git status` and `git log
--oneline` from the branch's fork point. Declare the phase in one line from what
they show, then continue.

Then the full recon, through a `handyman`: the current contents of each file a
finding names, and the commits touching them. Do this **every round**. I rebase,
squash, reword and reorder between your turns — shas move, content moves with
them, and the review's line numbers are stale the moment I touch the branch.
Never carry a sha, a line number, or "commit 3 is the one that added X" across a
turn without re-resolving it against the tree.

**2 — Triage into verdicts, and gate.** No edits yet. One row per finding: what
it claims, where it actually lives now, and a verdict —

- **accept** — real, in scope, and you know the fix.
- **decline** — wrong, already handled, or out of scope. One line of why.
- **decide** — a genuine fork the reviewer surfaced. Both options in one line
  each, plus your recommendation.

Then **stop and gate**. `AskUserQuestion` for the decides — that is exactly what
it is for, a concrete choice I have to make. Do not absorb a fork by picking the
sensible-looking option; a reviewer finding a fork is the same class of event as
phase 2 branch B, and if the answer is expensive to reverse, say so and send me
to `/council` instead of resolving it in a fixup.

Also gate anything that is **not a fixup**: a new migration, a directory move, a
new dependency, a schema change, or a finding that only makes sense as new work.
Name it at the gate as a new commit or a ledger change. Folding one of those
into someone else's commit is the specific thing this phase exists to prevent.

**3 — Attribute, from git. Two lookups, not one.** For every accepted finding:

**A — who introduced it. Resolve at line level, never file level.** Take the
line numbers as they stand at HEAD, then ask git about *those lines*:

```
git show HEAD:<file> | grep -n '<the signature or literal you are changing>'
git log --oneline -L <start>,<end>:<file>
```

`git log -S'<symbol>'` is the equivalent when the thing moved between files. The
ledger and your memory are not sources.

**No file-scoped query may pick a target** — not `git log -1 -- <file>`, not
`git log --oneline -- <file>`. Those answer "what last touched this file", and
on a branch whose commits revisit the same files by design that is almost never
the commit owning your lines. Worse, it is often right *by coincidence*, so it
survives review looking as though it was checked. **Attribution is the
deliverable most likely to be wrong and the one I cannot check by reading the
diff** — a fixup aimed at the wrong sha lands the change in the wrong commit.

**B — has anything since touched the same lines.** `git log --oneline
<sha>..HEAD -- <file>`, then read the hunks of whatever it returns. The
file-scoped query is deliberate here and does not contradict the warning above:
it is a cheap over-approximation of "has this moved", which you then narrow by
reading. Finding the introducer is a different question and needs the precise
tool.

**The target is the latest commit the fix touches or depends on, not the
earliest one that could claim it.** A fixup has to apply at its target *and*
leave that commit green. If a later commit rewrote those lines, removed a field,
or moved the block, a fixup aimed at the introducer either conflicts on rebase or
silently reinstates state a later commit deliberately took out. Attribution being
correct is not sufficient — this is the failure that survives a correct answer to
lookup A.

So: retarget to the latest commit that touched the lines, or land on top. A
finding that spans commits, or attributes to none, lands on top too — say so
explicitly rather than picking the nearest sha.

**When `-L` lands outside the branch** — the initial import, or anything before
the fork point — the finding is **unattributed**, however much a file-scoped
query wants to hand you a branch sha. Say that plainly, then pick the target on
one of two stated grounds: the latest commit the fix *depends on*, or the commit
whose stated purpose owns this kind of change. Name which ground you used. The
file-level coincidence never gets to choose.

**Then check the neighbouring lines before you commit to a target.** A fixup
editing a line that sits inside a *later* commit's hunk context will conflict on
autosquash, even when `-L` says nothing ever touched your line. Look at what the
later commits changed within a few lines either side; if one of them is adjacent,
target that later commit instead. This is the one case where the latest commit
beats the strictly correct one — cheap to spot, expensive to discover during the
rebase.

**4 — Build by concern, through tradies.** Phase 4 step 3's rule holds: you do
not write it yourself.

**Partition by disjoint file sets, not by target commit.** Commit groups share
files as a rule, not an exception — commits are sequential slices of one feature,
so grouping on them makes the overlap check fire constantly and serialises a
round that didn't need it. Cut the lanes by concern instead — repositories,
domain, services, the cross-cutting move — so the file sets genuinely don't
intersect, and phase 4's "two tradies never hold the same file" holds for free.

A tradie does not need to know target shas; hunk-level attribution happens at
map time in step 5. Each brief carries the findings in that lane, the file list,
the house rules covering those files verbatim as in phase 4 step 2, and the
constraint that it fixes those findings and nothing else. No lane spans two git
repos.

**5 — Hand back the fixup map, and stop.** The output is not a commit message:

- **Verdicts** — accepted / declined / decided, one line each. Declines carry
  their reason; round 2 re-raises anything you declined silently.
- **Fixup map — a markdown table**, one row per target commit, columns in this
  order:

  | Target | Files | Basis |
  | --- | --- | --- |
  | sha as of *this round's* recon + its subject | what lands there | the step 3 line-level answer |

  **Target** carries the sha and subject together, so I can eyeball that the sha
  still matches the commit I think it is. **Files** is what lands there; **a file
  whose hunks split across commits is named by hunk, not by path** —
  "`ticket-type-repository.ts` — the update-guard hunk only". A bare path means
  the whole file. **Basis** is the `git log -L` answer that chose this target,
  or "unattributed" plus the ground you picked instead — never left blank, and
  never a file-scoped query.

- **Commands — one fenced `bash` block per table row, in apply order,
  immediately under the table.** Not a column: a fenced block is runnable in my
  terminal and a table cell is not. Each block stages exactly that row's paths
  and fixes up exactly that row's sha, nothing else:

  ```bash
  git add <paths from that row> && git commit --fixup <sha>
  ```

  **A split file gets `git add -p <file>` in its own block, with a comment
  naming the hunks for that sha**, before its fixup line — never a flat path,
  which stages the other commit's hunk into this one and does it silently. You
  do not run any of it, you do not `--autosquash`, you do not rebase. This phase
  touches history more than any other; that makes the git rule tighter here, not
  looser.

- **Verified** — the commands you ran and their output, at the tip. Say plainly
  that this is the tip only: whether each commit is still green *after* the
  autosquash cannot be checked without rewriting history, which is mine to do.
  Step 3's retarget rule is what stands in for that check — if you skipped it,
  say so here.
- **Unattributed** — anything landing as a new commit on top, and why. If a row's
  Basis says unattributed, it still belongs here with the reasoning, not only in
  the table cell.

Record the round in the ledger under `## Review round N` — findings, verdicts,
and where each landed. If the ledger is already deleted, say so and put the map
in the chat only; don't resurrect it.

Then stop. Another round of findings means another round of this loop, starting
at step 1, because by then I will have rebased.

## Finishing

When the last commit is marked done, say so in one line, list the commit
subjects in order, and name anything in the ledger that was dropped or deferred.
Then **keep the ledger** until the branch is merged — review comes back after
the last commit is done, and phase 6 records its rounds there. Ask before
deleting it; don't delete it as a tidy-up.
