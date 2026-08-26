---
name: inspect
description: A second pair of eyes on a PR I have already read myself. One pass, no slicing, tests my read against the code, answers in chat. Runs in-session on a Sonnet or Haiku session and hands off to one inspector on anything dearer. Read-only — nothing is written, nothing is posted.
when_to_use: When I ask for it by name — "/inspect", "inspect PR 74", "cross-check my read on this PR". The size boundary between the three review skills: /code-review for my own uncommitted diff, /inspect for a PR I have read and want tested, /dissect for a PR too big for me to read at all. If I could not read the PR myself, this is the wrong tool and its answer will be thin.
disable-model-invocation: true
argument-hint: "[PR url or number], what I already think, --findings"
---

Target: $ARGUMENTS

A second pair of eyes on a PR I have already read. I have a view; the job is to
test it, not to replace it.

## Who runs it

- **Sonnet or Haiku session** — do it yourself, here, now. That is the whole
  point; a fork would only add a hop to work already priced correctly.
- **Opus or Fable session** — hand it to one `inspector` and relay what comes
  back. Everything below is its brief, plus the target, the diff range, and my
  read. Tell it that where its standing instructions talk about a slice someone
  else picked, it should read that as this whole PR. Reading a small PR at Opus
  rates is the one thing this skill exists to avoid.

Either way it is a single pass. No slicing, no fan-out, no second round.

## Getting the diff

`gh pr diff`, `gh pr view --json title,body,files`, `git fetch`. Never `pull`,
`checkout` or `switch`. If the target is ambiguous, ask before spending
anything; an inspector that cannot ask reports blocked instead of guessing.

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

  Prose by default. Only when I ask for it — `--findings`, or I say to use the
  findings list — put this one section through `ReportFindings` instead:
  `failure_scenario` is the mechanism sentence the rule above already demands.
  Once, in the tool or in prose, never both. The other three sections stay as
  prose regardless; a confirmation of my read and a clean-checked list are not
  findings and have nowhere to sit in that schema.
- **Checked and clean** — where you looked and found nothing. A short list, not
  an account of the effort. This is what makes the section above worth trusting.
- **Couldn't check** — what you would have needed.

No preamble, no closing offer.
