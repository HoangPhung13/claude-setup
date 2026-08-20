---
name: release-notes
description: Draft lean release notes from a PR or a diff, for pasting into a GitHub PR description. Bullets only, one plain sentence each, features and user-visible fixes, nothing else. Takes an audience dial — internal team, client, or external users — which changes the vocabulary, not the length. Read-only; nothing is written to disk and nothing is posted. Use this whenever I ask for release notes, a changelog entry, a PR description, a "what shipped" summary, or notes about a PR or a branch for anyone who is not an engineer, even when I do not use the words "release notes" — the default instinct is to summarise the engineering work, and that is the wrong document.
when_to_use: 'When I point at a PR, a branch, or a diff and ask what to tell people — "release notes for PR 42", "write the PR description", "summarise this branch for the changelog", "what do I tell support about this", "customer-facing notes for this release". Invoke it before reading or diffing anything: the reading you do to orient yourself is reading you will redo through the filter this skill sets, and the first read is the one that anchors you to the code instead of the product.'
argument-hint: "[PR number or URL | base...head], [internal | client | external]"
---

Target: $ARGUMENTS

The unit is not the commit, the file, or the endpoint. The unit is **one change
a person outside the codebase could notice.** Everything else is invisible to
every reader this skill has and does not go in.

## Two rules that override everything below

**Output goes in the chat, in one fenced markdown block, and nowhere else.** No
file, no `plans/`, no Artifact, no `gh pr edit`. I paste it where it is going
myself. Writing it to disk creates a file nobody opens; posting it takes a
decision that is mine.

**Say it once and stop.** One bullet, one change, one sentence. No sub-bullets,
no bold lead-ins, no trailing clause explaining the mechanism. If a bullet needs
a second sentence, it is either two changes or one change you have not
understood yet. The dial below moves this line exactly once, at its far end, and
never anywhere else.

## The dial

Read it off my words. If I gave a stop by name, use it; if I said who is
reading, map it; if I said nothing, use **internal** and do not ask, because
that is the common case and asking costs me a turn.

**1 — internal** *(default)*. Support, sales, a PM, me on a Monday. They know
the product vocabulary and nothing under it. Feature names land unexplained.
Flat and short; this is a lookup table, not a document.

> - You can now export a report as a spreadsheet.

**2 — client**. The person who paid for the work and is checking they got it.
Same one sentence, but name the feature the way it appeared in the scope of work
rather than the way the UI labels it, and lead with the thing they asked for.

> - Reports can now be exported as a spreadsheet, including the custom columns.

**3 — external**. End users or a public changelog. They have no vocabulary from
us at all, so a feature name on its own means nothing. This is the one stop
where a bullet may run to a second short sentence, and only to say what the
thing is for. Take it when the name alone does not carry, not by default.

> - You can now download any report as a spreadsheet. Open it in Excel or Sheets
>   to build your own charts.

Nothing else changes across the stops. The filter, the ordering, the banned
words and the format are the same at all three, because they are about what is
true, not about who is asking.

## Phase 1 — Get the change

Resolve the target with whatever fits:

| Given | Use |
|---|---|
| PR number or URL | `gh pr view <n> --json title,body,commits` then `gh pr diff <n>` |
| `base...head` | `git diff --stat <base>...<head>` then `git diff <base>...<head>` |
| Nothing | current branch against the default branch |

**An infra or config repo cannot produce a bullet on its own.** A Terraform
diff gives you a scheduled Lambda, an event bus, a CORS policy; it never tells
you that the schedule is a daily digest someone reads, and a bullet written from
the resource alone describes plumbing. When the target is infra, or any repo
holding one layer of a feature whose other layers live elsewhere, read the
companion PRs for the same release alongside it. Ask me for them if you cannot
find them; that is a cheap question and the alternative is a page about queues.

Start with `--name-only` and the commit subjects. On anything past a handful of
files, that list tells you which files could possibly carry a user-visible
change; read those in full and skim the rest. A migration, a lockfile, a test
directory and a CI config can be read as one line each, because none of them can
produce a bullet.

If the diff is genuinely large, hand a `handyman` the file list and ask for a
**factual inventory** per file: what appeared, what was removed, which screens,
routes, settings, permissions, jobs or messages are involved. Do not ask it for
a summary. Deciding what a user notices is the entire job of this skill and it
is not delegable; gathering the raw facts is.

## Phase 2 — Filter, in two passes

**Pass one, visibility.** In: something a person can now do that they could not,
something that now behaves differently to them, something visible that is gone.

Out, always, no matter how much work it was: refactors, renames, type changes,
dependency bumps, tests, CI, logging, error handling that nobody sees,
performance work with no felt difference, migrations, feature flags that ship
switched off, and anything behind an internal admin path unless the reader is
the one using it.

**Pass two, significance.** Visibility is a real bar in a repo where most of the
work is invisible, and no bar at all in a UI repo, where every drag handle,
column width and dark-mode class clears it. Pass one will happily hand you forty
true bullets. So ask of every survivor: **if this shipped and nobody announced
it, who would come asking about it?**

Nobody comes asking about a reordering handle, a field that used to render at
half width, or a redirect after save. They come asking about a section of the
app that did not exist last week. If you cannot name the person who would go
looking, the bullet is out, and it is out even though it is true and even though
it took someone a day.

A fix clears pass two only when the bug caught someone who was not looking for
it. "Invoices over 100 lines would not open" is in. "Fixed a null check in the
serialiser" is out, and so is a layout glitch the team found themselves. Write
the symptom rather than the cause, and only when the symptom had a witness.

At **external**, the filter tightens again: anything only staff can reach comes
out entirely, including admin screens that are perfectly in scope at the other
two stops.

The PR title and commit messages are a **hint, not a source.** They were written
by an engineer for engineers, and half of them describe the mechanism. Check
each against the diff before you believe it.

**If nothing survives the filter, say so in one line and write nothing.** A
release note listing a dependency bump is worse than no release note, because it
teaches the reader that these notes are not worth opening.

## Phase 3 — Write it

**One bullet per thing the product has a name for.** Not per commit, not per
file, not per capability. If Tasks is a section of the app, Tasks is one bullet,
however many screens, checklists, templates, reminders and drawers went into
building it; a second bullet is earned only by something a user would call a
different feature.

This rule decides whether the notes are readable, and it is the one under the
most pressure, because a diff hands you the pieces and never the name. The
test: could you have written this bullet from the product's navigation menu
rather than from the file tree? Then you are at the right altitude.

Order by how noticeable it is, not by how hard it was.

**Shape.** Present tense, active, the reader or the thing as the subject. "You
can now …" or "<Thing> now …". **Twenty words, counted rather than estimated**,
at stops 1 and 2 and for the first sentence at stop 3. Commas are fine; four
things a feature does can read clean in eleven words. What will not fit in
twenty is either a second bullet's worth of change or a mechanism you have
started explaining. Name the feature the way the product names it, never the way
the code names it.

**Translation.** The gap between what the diff says and what the reader needs is
the whole skill:

| The diff says | The bullet says |
|---|---|
| Added `POST /reports/export` with CSV serialiser | You can now download a report as a spreadsheet. |
| Implemented debounced search against the new index | Search results now appear as you type. |
| Added `role` column and guard middleware to team routes | Team owners can now control who sees billing. |
| Refactored auth to use refresh tokens | *nothing; nobody notices being signed in* |
| Fixed off-by-one in the pagination cursor | The last item on a page is no longer skipped. |
| Bumped the PDF library to v4 | *nothing* |
| Removed the legacy settings screen | The old settings page is gone; everything moved to Account. |

Notice the right-hand column never mentions where the change lives. That is the
test: if a bullet names a layer, a file, a service or a technology, it was
written for the wrong reader, at any stop on the dial.

**Words that do not appear in the output.** Endpoint, API, route, component,
schema, migration, refactor, middleware, cache, query, token, payload, backend,
frontend, database, deploy, config, hook, service, module.

**Words that also do not appear**, for the opposite reason: seamless, powerful,
robust, enhanced, streamlined, revamped, leverage, unlock, elevate, delightful.
Marketing language is the other way of not saying what changed, and the pull
towards it gets stronger the further along the dial you go. Plain and slightly
flat is correct at all three stops.

### Format

Exactly this, in one fenced markdown block. Two headings and no others. **No
grouping sub-headings**: no bold category lines, no third level, nothing that
buckets bullets by area. A bucket is an empty slot asking to be filled, and five
buckets is how a ten-bullet release turns into forty. **Omit a section that
would be empty**; never write a heading with "none" under it.

```markdown
## What's new
- One sentence.
- One sentence.

## Fixes
- One sentence.
```

**Ten bullets across both sections, hard.** That holds for a whole release
spanning a dozen PRs, because a bigger release means a higher altitude per
bullet and not more of them. Going over is not something to flag to me in a
closing note; it is the signal that you are writing at the wrong altitude, so go
back to the naming rule and merge. If you are genuinely sure I am losing
something I need, give me the ten and say in one line what the eleventh would
have been.

## Phase 4 — Check your own draft

Read the block back as someone who has never seen the repo. Any bullet you
cannot picture happening on a screen is describing the code, and it goes.

Then, on the draft text:

```bash
grep -inE 'endpoint|api|route|component|schema|migration|refactor|middleware|backend|frontend|database|config|payload|serialis|seamless|robust|streamlin|leverage|enhanc' <<< "$NOTES"
```

Hits are not automatically wrong; a product genuinely called API Keys is fine.
Read each one and decide. A hit you cannot justify to a non-engineer is a bullet
to rewrite.

Hand me the block, and one line naming which stop you used so I can tell you to
move it. Nothing else: no preamble, no account of what you left out unless I
asked, no offer to post it.
