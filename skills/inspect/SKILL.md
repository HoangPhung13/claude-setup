---
name: inspect
description: A second pair of eyes on a PR I have already read myself. Runs entirely inside one inspector subagent on Sonnet, tests my read against the code, and answers in chat. Read-only — nothing is written, nothing is posted.
when_to_use: When I ask for it by name — "/inspect", "inspect PR 74", "cross-check my read on this PR". The size boundary between the three review skills: /code-review for my own uncommitted diff, /inspect for a PR I have read and want tested, /dissect for a PR too big for me to read at all. If I could not read the PR myself, this is the wrong tool and its answer will be thin.
disable-model-invocation: true
argument-hint: "[PR url or number], and what I already think"
context: fork
agent: inspector
background: false
---

Target: $ARGUMENTS

A second pair of eyes on a PR I have already read. I have a view; the job is to
test it, not to replace it.

You are reviewing the whole of one small PR. Where your standing instructions
talk about a slice that someone else picked, read that as this PR.

## Getting the diff

`gh pr diff`, `gh pr view --json title,body,files`, `git fetch`. Never `pull`,
`checkout` or `switch`. If the target is ambiguous, say so and stop; you have no
way to ask me.

## Do

- **Read around each hunk, not the hunk alone.** Most wrong findings come from
  reading a diff instead of the function it landed in.
- **Test my read.** If `$ARGUMENTS` carries what I already think, take each
  claim and mark it confirmed, contradicted, or not checkable. This is the
  cross-reference, and it is the whole reason this skill exists rather than a
  plain review.
- **Read the deletions as carefully as the additions.** Removed handling is the
  easiest thing to skim past, and I have already skimmed this PR once.
- **Mechanism or nothing.** These inputs, this branch, this outcome. If you
  cannot write that sentence, you do not have a finding.

## Don't

- **No nitpicks.** Naming, formatting, "consider extracting this". The bar is
  whether I would do something differently.
- **No speculation** about a service, API or file you did not open.
- **Never write, never fix, never post.** Bash is for reading and for the
  repo's own checks.
- **No pasteable GitHub block.** This lands in chat, for me. `/dissect` is the
  one that produces a comment.

## Return, under 300 words

- **On my read** — one line per claim I made. Omit the section if I made none.
- **Worth knowing** — findings, each with `path:line` and the mechanism in a
  sentence or two. "Nothing" is a real answer and the likeliest one on a PR I
  was able to read myself.
- **Checked and clean** — where you looked and found nothing. A short list, not
  an account of the effort. This is what makes the section above worth trusting.
- **Couldn't check** — what you would have needed.

No preamble, no closing offer.
