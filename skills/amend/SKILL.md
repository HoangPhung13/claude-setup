---
name: amend
description: Correction pass on a commit the orchestrator just handed back — makes the changes I ask for, records them in the ledger, never commits and never re-plans. Runs in the orchestrator session, not a new one.
when_to_use: 'When a commit has been handed back by /orchestrate, is still uncommitted, and I ask for corrections before I commit it — "change X", "actually make it Y", "drop that bit". Do not invoke while a tradie is still working: mid-flight comments belong to the running orchestrator, which folds them into the next brief. Do not invoke once I have committed the work — that is new work, not an amendment.'
argument-hint: "[what to change]"
---

Changes: $ARGUMENTS

A commit has been handed back and I haven't committed it yet. I've read the diff
and I want corrections first. This is a correction pass, not new work: no
re-planning, no re-gating, no `EnterPlanMode`, no widening the commit.

**Stay in this session.** Switching to a cheaper one costs me more in
re-explaining than it saves, so the savings have to come from *who does the
edit*, not from where I'm sitting.

## Orient

You should already have the context. If you don't — new session, or compaction
took it — read `plans/*-ledger.md` for the entry marked `handed back`, then
`git status` and `git diff`. The diff is the subject; don't go re-exploring the
codebase around it.

If nothing is marked `handed back`, say so and ask which commit I mean. Don't
guess from the most recent one.

If the tree is clean and the last commit matches that entry, I've already
committed it. Say so before touching anything — what you write now lands as new
uncommitted work on top, which may not be what I meant.

## Who does the edit

The decision is already made; this is only about cost.

- **A few lines, in files already in context** → do it yourself. Writing a brief
  would cost more than the edit.
- **Larger, or spread across several files** → one `tradie` with a short brief.
  You have the context to write it in three sentences, and it's your output
  tokens on a multi-file edit that are worth avoiding.
- **Needs a decision I haven't made, or reaches outside this commit's scope** →
  stop and name it. That's the orchestrator's call or `/council`'s. Amendments
  that quietly grow are how a commit stops matching the diff I reviewed.

## Rules

- Exactly what I asked for. No drive-by fixes, no reformatting, no renames I
  didn't name. If you spot something else, one line at the end.
- Match the conventions already in the file.
- Re-run the project's own checks. The commit was green when you handed it back;
  it has to still be green.
- Never commit, and never stage unless I ask. CLAUDE.md has the rule; restating
  it because the name suggests otherwise. This skill has nothing to do with
  `git commit --amend`.

## Record it

Append to that commit's **Handoff** section in the ledger: what I asked for and
what changed, one line each. That section is the only reason the next tradie
will know any of this happened, and I won't remember to tell it.

If the correction changed what the commit claims, reissue the **proposed commit
message**.

## Return

Short. Changed files, one line each. The verification command and its result.
The line you wrote to the ledger. No summary of the code — I'm looking at the
diff.
