---
name: dissect
description: Review a PR that is too large to read by hand. Recon on Haiku, slice the diff into change-units, one reviewer per slice, then synthesise a single markdown comment I paste into GitHub myself. Read-only — nothing is built, nothing is posted.
when_to_use: When I ask for it by name — "/dissect", "dissect PR 74", "run a proper review on this PR" — or I point at a PR and say it is too big for me to review. Not for my own uncommitted work, and not for a PR I could read myself; /code-review is cheaper and better for both. Invoke it as your FIRST action, before reading or diffing anything: phase 1 does all recon through handyman on Haiku, so any orienting you do first is thrown away and was paid for at Opus rates.
disable-model-invocation: true
argument-hint: "[PR url or number], or [base...head]"
allowed-tools: EnterPlanMode, ExitPlanMode, AskUserQuestion
---

Target: $ARGUMENTS

**I am the copy-paster.** The deliverable is one markdown block that I paste
into the PR myself. You never post a comment, never open a review, never push,
never edit code. If I wanted GitHub written to, I would have said so.

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

Then **call `EnterPlanMode`.** Phases 0 through 2 are read-only by design and
plan mode enforces that at the tool layer, and it gives the phase 2 gate a real
approval prompt instead of a message that just stops.

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
- **Prior pass.** `plans/pr-<n>-review.md` (see below). If it exists, its head
  SHA and its findings.
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

## Phase 5 — The comment

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
- Reference code as `` `path/to/file.ts:42` ``, or as a link to the blob at the
  head SHA. Never a bare line number with no path.
- No speculation. Nothing that rests on an API, service, or file nobody read.
  If it could not be checked, either leave it out or say in one clause that it
  was not checked.
- No nitpicks survived this far. If one did, cut it.

### The sections, in this order, with these names

Always `##`, never `###`, and the same names every pass. Two reviews of two
repos should read as the same document.

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

**5. `## Worth a look`** — not wrong, but unexercised, wide-reaching, or worth a
smoke test before merge.

**6. `## Verified clean`** — the load-bearing things that were checked and hold
up. This is what makes the rest of the comment worth trusting: it says where the
review actually looked. Name the check, not the effort.

Omit sections 4, 5 and 6 entirely when they would be empty. Never write a
section heading followed by "none".

### On a second pass, trim

Same names, fewer of them. `## Verdict` and `## Blocking` always. Add
`## Since last pass` directly under the verdict: a status table of the previous
findings, each one fixed, still there, or moved. Drop `## Files changed` unless
files arrived since the last pass, and then list only those. Drop
`## Verified clean` unless something it vouched for has changed underneath it.

Observations about the code, not instructions to a person. This gets forwarded
to the author, so nothing in it should read as a telling-off.

## Second pass, after they push

Save every pass to `plans/pr-<n>-review.md` with the head SHA it was written
against, the slice plan, and the findings. Confirm `plans/` is gitignored first
with `git check-ignore -q plans/`; if it is not, stop and tell me.

On re-run, `git fetch` and then test whether that SHA still exists:
`git cat-file -e <sha>^{commit}`.

- **Still there** — fast-forward. Review `<sha>...<newhead>` only, then check
  each previous finding against current code.
- **Gone** — force-push, which for this team is the normal case, not an
  accident. The branch was rebased and the old SHA is unreachable, so do not try
  to diff against it. Re-slice the whole PR from `base...newhead`.

Either way, **before anything else**, go through the previous findings one at a
time and say whether the current code fixes it, still has it, or moved it. A
rebase often means the work was rewritten rather than patched, so check that a
fix did not quietly become a different bug. Then flag any file that is in the PR
now and was not at the last pass: that list is the highest-risk part of the run.
