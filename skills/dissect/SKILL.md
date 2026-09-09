---
name: dissect
description: Review a PR that is too large to read by hand. Recon on Haiku, slice the diff into change-units, one reviewer per slice, then synthesise the findings — either as a markdown comment I paste into the PR, or as a short briefing for me in the terminal. Read-only — nothing is built, nothing is posted.
when_to_use: When I ask for it by name — "/dissect", "dissect PR 74", "run a proper review on this PR" — or I point at a PR and say it is too big for me to review. Not for my own uncommitted work, and not for a PR I could read myself; /code-review is cheaper and better for both. Invoke it as your FIRST action, before reading or diffing anything: phase 1 does all recon through handyman on Haiku, so any orienting you do first is thrown away and was paid for at Opus rates.
disable-model-invocation: true
argument-hint: "[PR url or number] [to creator | to reviewer], or [base...head]"
allowed-tools: EnterPlanMode, ExitPlanMode, AskUserQuestion
---

Target: $ARGUMENTS

## Who the write-up is for

Two output modes. Same review either way — phases 0 through 4 do not change, and
neither does the standard a finding has to meet. Only the phase 5 deliverable
differs. Resolve the mode from `$ARGUMENTS` before phase 0 and name it on the
same line as the file count.

- **`to creator`** — the default. One markdown block I paste into the PR under
  my own words. The audience is the author, who has none of this context.
- **`to reviewer`** — a briefing for me, in the terminal. The audience is me,
  deciding whether to dig in. No markdown block, no paste, no wall.

Anything meaning "for me", "just tell me", or "don't write the comment" selects
reviewer. Anything naming the author, the PR thread, or a paste selects creator.
When `$ARGUMENTS` says neither, use creator and do not ask.

If I ask for the other mode after you have already delivered one, write it from
the findings you are holding. Do not re-run the review; nothing about the code
changed between my two sentences.

**You never write to GitHub in either mode.** No comment, no review, no push, no
edits to code. If I wanted GitHub written to, I would have said so.

**I am not the author.** There is no ledger and no plan of mine to check the
diff against. The PR's own title, body and commit subjects are the only
statement of intent you have, and testing the diff against that statement is
half the job.

**Git.** `git fetch` and `gh` are fine; neither touches the working tree or
HEAD. `pull`, `checkout` and `switch` are not — if you genuinely need the branch
on disk, ask. `gh pr diff` and `gh api` usually mean you don't.

## Phase 0 — Resolve, and check this is even the right tool

Resolve `$ARGUMENTS` to a repo, a PR number, and a `base...head` range. Then:

```
git fetch origin
gh pr view <n> --json title,body,author,baseRefName,headRefName,headRefOid,commits
git diff --numstat <base>...<head> | awk '{a+=$1;d+=$2;n++} END {print n" files, +"a"/-"d}'
```

Print that one line. If it comes back small enough that I could plainly read it
myself, say so and tell me to run `/code-review <n>` instead. Stopping here is a
success. This skill costs several subagents; it has to earn them.

## Phase 0.5 — Is this a second pass?

Ask this before any recon, because the answer collapses phases 1 through 3. A
second pass reviews a **delta**, not a PR.

Look for `plans/pr-<n>-review.md`. Confirm `plans/` is gitignored first with
`git check-ignore -q plans/`; if it is not, stop and tell me. A prior pass may
also be sitting in this conversation rather than on disk — that counts, and it
is better evidence than the file, because it carries the reasoning and not just
the conclusions.

With a prior pass in hand, `git fetch` and test whether its head SHA still
exists: `git cat-file -e <sha>^{commit}`.

- **Still there** — fast-forward. The range is `<old head>...<new head>`.
- **Gone** — force-push, which for this team is normal rather than an accident.
  The branch was rebased and the old SHA is unreachable, so do not try to diff
  against it. Fall back to `<base>...<new head>` and read it as a rewrite: the
  findings still need answering, but "what changed" is no longer a diff you can
  take at face value.

Print one line — which case, the delta's file count, how many findings the last
pass recorded — then go to **Phase 6**. Skip `EnterPlanMode`; the delta pass has
its own gate and is smaller than the ceremony.

No prior pass means a first pass. **Call `EnterPlanMode`** and carry on to phase
1. Phases 0 through 2 are read-only by design, plan mode enforces that at the
tool layer, and it gives the phase 2 gate a real approval prompt instead of a
message that just stops.

## Phase 1 — Recon (cheap, and you read nothing yourself)

Delegate to `handyman`, several in parallel. **The diff never enters your
context.** You are buying a map, not the territory; every hunk you read here you
pay for again when the slice reviewer reads it properly.

Cover:

- **Intent.** PR title, body, commit subjects, linked issues. What does this PR
  claim to do?
- **Shape.** `git diff --numstat <base>...<head>`, full list. Directory
  clustering. Which files are generated, vendored, lockfiles, snapshots.
- **House rules.** `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, the lint and
  tsconfig setup. Return the rules as bullets. If none exists, say so and the
  priority ladder in phase 3 collapses to standard review.
- **The conversation**, if the thread is long: `gh pr view <n> --comments`.
  Digest to what a reviewer needs, not a transcript.

If the repo is GitNexus-indexed, `detect_changes({scope: "compare", base_ref:
"<base>"})` and `impact()` on the changed symbols are the cheapest slicing
signal you will get. Skip silently if the index is stale; do not stop to rebuild
it.

## Phase 2 — Slice, and gate

Group the changed files into **change-units**: sets that have to be understood
together to be understood at all. Not "10 files per reviewer". A slice is a
thing the PR does, and its file set is whatever that thing touched.

Assign a tier per slice by **what could go wrong there**, not by size:

- `handyman` — generated code, lockfiles, snapshots, mechanical renames, config
  bumps. Confirm it is what it claims to be and move on.
- `inspector` — ordinary application code. This is most slices.
- `scientist` — auth, money, data model, migrations, concurrency, anything
  expensive to get wrong. One or two slices, rarely more. A scientist on every
  slice is how this skill becomes more expensive than the thing it replaced.

Print, in under 400 words: the slices, one line each on what each appears to do,
its file count and tier, and which slice you expect the trouble to be in. Then
call **`ExitPlanMode`**.

Do not use `AskUserQuestion` to ask whether the slicing is good; that is what
the gate is. Use it only for a real fork you cannot resolve, before the gate.

## Phase 3 — Fan out, one reviewer per slice

Spawn them concurrently, one message. Every brief carries: the goal, the exact
file list, the range, the house rules from phase 1, the PR's stated intent, and
the return shape. Workers cannot see this conversation.

Tell each reviewer to work the ladder in this order, because a finding high on
it outranks three below it:

1. **Scope** — does this slice do something the PR never said it would?
2. **House rules** — the bullets from phase 1. Blocking when violated.
3. **Correctness** — bugs, edge cases, security, regressions.
4. **Conventions** — only where the file's own neighbours disagree with it.

**No nitpicks, at any tier.** The bar is whether a reviewer would act on it.

**Never pass `model` on the Agent call.** Every agent pins its own model and
effort; the parameter silently overrides that frontmatter.

## Phase 4 — The part only you can do

Read every `spillover` field. The reason this skill exists rather than one big
review pass is that a signature changed in slice A and a caller in slice F still
uses the old shape, and no single reviewer held both. Reconcile them and say
plainly what the PR does as a whole, which is the thing I actually cannot get by
reading hunks.

If a cross-slice suspicion is real but unproven, spend one more `inspector` on
exactly that question with both file sets in scope. One targeted follow-up, not
a second round.

Then judge **scope drift**: does the diff match what the PR body says it is? A
PR that started as one thing and now touches migrations, CI, or a new dependency
has changed shape, and that is worth saying even when each change is defensible.

**Merge state comes from GitHub, never from you.**
`gh pr view <n> --json mergeable,mergeStateStatus`. Report a conflict only when
that says there is one. Do not run `git merge-tree` simulations, do not report
that the branch is behind its base, and do not report that GitHub's file count
differs from the range because the comparison base has moved. A branch that
merges cleanly has no merge finding, however far behind it has drifted.

Verify anything load-bearing yourself before it goes in the comment. A
reviewer's finding is a claim; you are the one signing it.

## Phase 5 — The deliverable

**Save the pass first, whichever mode you are about to write.** Put it in
`plans/pr-<n>-review.md`: the head SHA it was written against, the slice plan,
and every finding with its severity and `path:line`. This is what phase 0.5
reads next time, and a pass that only ever existed in a terminal cannot be
checked against later. Record the findings you *downgraded* too, and why — a
later pass that rediscovers one and calls it blocking has learnt nothing.

Three rules hold in both modes, and they are the load-bearing ones:

- **No speculation.** Nothing that rests on an API, service, or file nobody
  read. If it could not be checked, either leave it out or say in one clause
  that it was not checked.
- **No nitpicks survived this far.** If one did, cut it.
- **Reference code as `` `path/to/file.ts:42` ``**, never a bare line number
  with no path. In creator mode a link to the blob at the head SHA also works.
- **Not `ReportFindings`, in either mode.** Creator mode needs one block I can
  paste and a rendered findings list in my terminal is not that; reviewer mode
  needs something I can read at a glance, which a findings table is not either.
  The schema also has no room for Scope, Files changed or Verified clean, so it
  could only ever duplicate one section of six.

### Mode: to creator (default)

One fenced block, GitHub-flavoured markdown, nothing after it but a one-line
note of anything you deliberately left out.

**Format rules, all of them load-bearing:**

- Use a **four-backtick** outer fence so inner triple-backtick code blocks
  survive the paste.
- **No hard-wrapped lines.** Long bullet lines are correct; a bullet broken
  across source lines is not.
- **Third person throughout.** No `I`, `my`, or `we`. The block gets pasted
  under my own words and those carry the first person. Findings say what the
  code does, not what the reviewer did: "the padded range overlaps adjacent
  weeks by two days", never "I checked and found".
- `ID`, not `id`, in prose.

#### The sections, in this order, with these names

Always `##`, never `###`, and the same names every pass. Two reviews of two
repos should read as the same document.

`/inspect` uses this same spine with fewer sections. Keep the names identical
across both; where they diverge below, the divergence is the point.

**1. `## Verdict`** — one line, `Looks mergeable` or `N blocking`, then two
sentences on what the PR actually does. Nothing else lives here.

**2. `## Files changed`** — a collapsed `<details>` whose summary is
`N files, +X/-Y` over the whole range. The table is always
`| File | +/- | What it does |`, counts always written `+X/-Y` even when one
side is zero, sorted by size descending.

**Roll up clusters into one row.** Generated output, migrations, tests that only
mirror a source file already listed, renamed directories: `37 files, +15079/-25`
and one line describing the lot. Past roughly forty rows a table stops being a
summary and turns back into the diff, which is the thing I could not read in the
first place.

**3. `## Scope`** — does the diff match what the PR body says it is? Name what
it does that it never said it would, and what it claims but does not do. One
line saying it matches is a complete section. This sits above the findings
because it is a judgment about the PR, not about a line.

**4. `## Blocking`** — what breaks, the concrete path to it, and `path:line`.
Mechanism, not adjectives.

**5. `## Worth a look`** — worth a smoke test before merge, but not a reason to
hold it.

**Blocking** means merging it produces a wrong result, a crash, or lost data on a
path this PR actually takes. **Worth a look** means the code is correct as
written but is unexercised, reaches further than it looks, or rests on an
assumption worth confirming.

The test is which sentence you can write. "These inputs reach this branch and
return the wrong total" is blocking. "This could bite if a caller ever passes
null" is worth a look.

**6. `## Verified clean`** — the load-bearing things that were checked and hold
up. This is what makes the rest of the comment worth trusting: it says where the
review actually looked. Name the check, not the effort.

`## Blocking` is always present, `None` when there is nothing. A reader
scrolling a long comment cannot tell a clean bill of health from a section that
was forgotten, and that is the one thing this comment has to be unambiguous
about. Sections 5 and 6 are omitted entirely when empty; they carry no verdict,
so their absence costs nothing.

#### On a second pass, trim

Same names, fewer of them. `## Verdict` and `## Blocking` always. Add
`## Since last pass` directly under the verdict: a status table of the previous
findings, each one fixed, still there, or moved. Drop `## Files changed` unless
files arrived since the last pass, and then list only those. Drop
`## Verified clean` unless something it vouched for has changed underneath it.

Observations about the code, not instructions to a person. This gets forwarded
to the author, so nothing in it should read as a telling-off.

### Mode: to reviewer

Prose in the terminal, for me. No fenced block, no `<details>`, no section
headers, and a table only where a table genuinely is the shortest form. Markdown
past a bullet and a backtick is noise here — nothing is being pasted anywhere.

First person is fine and so is addressing me directly. This is you telling me
what you found, not a document you are handing over.

**Lead with the answer.** If I framed the request around a specific worry —
"make sure the old flows still work", "check the migration is safe" — that worry
gets the first sentence, before the count of anything. Otherwise: how many
blocking, and what the PR actually does, in two lines.

Then only these three things:

- **Each blocking finding** — the claim in one sentence, the mechanism in one
  more, and `path:line`. Not the scenario walkthrough, not the reachability
  argument, not the suggested fix. You still hold all of that; I will ask.
- **Worth a look** — one line each, no elaboration, and cut the lot below about
  three genuinely interesting ones.
- **One closing sentence** on where the review looked and found nothing. This is
  the whole `Verified clean` section compressed, and it has to survive the
  compression: without it I cannot tell a clean area from an unreviewed one.

Leave out `Files changed` — I have `gh pr diff`. Leave out `Scope` unless the
diff actually drifted, and then it is one line.

**The budget is one screen.** Past that you are explaining rather than
reporting. A blocking finding I can act on in two sentences beats a correct one
I have to read twice, and expanding one finding on request is far cheaper than
me reading five that did not need it.

On a second pass, lead with what changed since the last one — fixed, still
there, moved — and only then anything new.

Offer the creator block in a single trailing clause. Do not write it unless I
say so; writing both is the exact wall this mode exists to avoid.

## Phase 6 — The delta pass

Two questions, in this order, and nothing else:

1. Is each previous finding resolved?
2. Did the delta introduce anything new?

### Answering the first

Go through the previous findings **one at a time, before looking at the delta as
a whole.** For each, verify against the **code at the new head**, not against
the patch. A patch is not evidence: a fix can be present in the diff and still
wrong, and a finding can vanish from the diff because the code moved rather than
because anyone addressed it. Read the function as it now stands.

Land on one of four words, and say which:

- **Fixed** — with the `path:line` that closes it, and one clause on how.
- **Still there** — with the `path:line`, still.
- **Moved** — the code was rewritten and the finding now lives somewhere else.
  Give the new location. Common after a rebase.
- **Changed shape** — the fix works but introduced something adjacent. This is
  the one worth the most attention, and the reason a delta pass reads code
  rather than trusting a diff that says "fixed".

A finding you downgraded last pass still gets answered. It was a judgment about
reachability, and the delta may have changed what is reachable.

### Answering the second

Only now look at the delta itself. Slice it the way phase 2 slices a PR, but
expect the answer to be one or two reviewers, not six — the delta is small by
construction, and a delta pass that spawns as many agents as the first pass has
failed at the only thing it was for. Brief each reviewer with the previous
findings as well as the file list, so they can tell a real fix from a
coincidence.

**Files new to the PR since the last pass are the highest-risk part of the run**
— nobody has ever reviewed them. Name that list explicitly and give it a
reviewer even when the rest of the delta is trivial.

If the delta turns out to be a rewrite rather than an increment — most of the
PR's files touched, or the whole approach changed — say so and re-run the full
protocol from phase 1 instead. Pretending a rewrite is a delta is how the second
pass reviews less than the first did.

### The gate

Print the finding-status table and the delta's shape before spawning anything.
That is the gate; it does not need plan mode, and it does not need my approval
unless the delta is big enough that you are proposing a full re-slice.

### The write-up

Phase 5's rules hold, both modes, with one change: `## Since last pass` leads,
directly under `## Verdict`, as a status table of every previous finding. Then
`## Blocking` for anything still there or newly found. Drop `## Files changed`
unless files arrived since the last pass, and then list only those. Drop
`## Verified clean` unless something it vouched for has changed underneath it.

Save this pass over `plans/pr-<n>-review.md` as phase 5 says, with the new head
SHA — including the previous findings and their resolutions, so the file stays a
single running record rather than a snapshot of the latest run.
