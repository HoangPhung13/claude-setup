---
name: amend
description: Fallback entry into the orchestrator's correction pass on a handed-back commit, for when the session no longer has the context. Normally I just comment in the thread instead and the orchestrator handles it without this skill.
when_to_use: 'Only when I type /amend by name. A plain comment after a handback is NOT a trigger — that is /orchestrate phase 5, which you follow directly without loading anything. Never suggest this skill to me; if I have commented on a handed-back commit, just do the correction pass.'
argument-hint: "[what to change]"
---

Changes: $ARGUMENTS

A commit has been handed back and I haven't committed it yet. I've read the diff
and I want corrections first.

**The rules live in `/orchestrate` phase 5 — Corrections.** Follow that section.
This skill exists only to carry the context back when the session lost it, so
the one thing to do here that phase 5 assumes is already done:

## Orient

Read `plans/*-ledger.md` for the entry marked `handed back`, then `git status`
and `git diff`. The diff is the subject; don't go re-exploring the codebase
around it.

If nothing is marked `handed back`, say so and ask which commit I mean. Don't
guess from the most recent one.

If the tree is clean and the last commit matches that entry, I've already
committed it. Say so before touching anything — what you write now lands as new
uncommitted work on top, which may not be what I meant.

**Stay in this session.** Switching to a cheaper one costs me more in
re-explaining than it saves, so the savings come from *who does the edit* —
phase 5's tradie rule — not from where I'm sitting.

This has nothing to do with `git commit --amend`. Never commit, and never stage
unless I ask.
