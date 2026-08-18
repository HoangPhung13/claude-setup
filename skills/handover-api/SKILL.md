---
name: handover-api
description: Write the API→frontend handover markdown at the end of an API session — the delta a frontend Claude Code session needs to consume what just shipped. Fixed section names, types copied verbatim from source, no UI prescription, no artifact. Use this whenever I am on the API side and ask for a handover, a handoff, a frontend contract, or "document this for the frontend" — even if I phrase it as just writing a markdown file, because the default instinct is to publish an artifact and invent a UI, and both are wrong. This is the API-side direction only; a handover written from a frontend repo is a different document and not this skill.
when_to_use: 'When I finish work in an API repo and ask you to document it for the frontend — "write a handover", "handover MD for frontend", "document the contract for the FE session". Invoke it before reading or diffing anything: phase 1 does the delta sweep through handyman on Haiku, so orienting first throws that away at Opus rates.'
argument-hint: "[what shipped — feature, or PR/branch]"
---

Shipped: $ARGUMENTS

**The reader is another Claude Code session, not a person.** It will `cat` this
file as the first thing it does in a fresh frontend repo, with no memory of this
one. That reader wants the contract dense and verbatim; it does not want prose
it has to interpret. Write for the machine and the human skims it fine — the
reverse is not true.

It is also **only ever read by me and my own sessions**. Nobody is being
onboarded by it. So no overview, no executive summary, no explaining the
codebase or the business reason — the reader has the repo and the git log. Just
the contract.

## Two rules that override everything else in this file

**Never publish this.** No `Artifact`, no HTML, no styled page, no
`SendUserFile`. It is a markdown file in `plans/`, read with `cat` by a session
on this same machine. Publishing buys a rendering nobody opens and charges me
for the privilege.

**You have never seen the frontend.** Not the screens, not the component
library, not the routing, not what the user is trying to do on the page. So
every sentence you write about the UI is either wrong or — worse — followed.
State the constraint and stop; the frontend decides what to do about it. This
is the single most common way this document goes bad, and there is a whole
section on it below.

## Phase 1 — Recon, and you read nothing yourself

The handover is a **delta**, not a description of the API. Everything the
frontend already knows is a line I pay for twice.

**Delegate to `handyman` — several in parallel if the surface is wide. Do not
grep or read the codebase yourself.** This phase is bulk transcription of route
signatures and schema bodies: high volume, no judgment, and the cheapest context
you will ever buy. Every gap you leave here gets rediscovered while you are
mid-document at several times the price.

Cover at least:

- `git diff --stat <base>...HEAD`, and the full list of touched routes,
  controllers, DTOs, validators, schemas, enums, migrations.
- The **verbatim** text of every route decorator, Zod schema, and enum in that
  set, with `file:line`. Not a summary — the actual source, because it is going
  into the doc unmodified.
- Anything **deleted or renamed**, and every remaining caller of it. This is the
  highest-value part of the document and the easiest to miss, because nothing in
  the new code points at it.
- Error codes and their trigger conditions, wherever this repo keeps them.

### When you may skip the sweep

Only for what you actually built in this session and can still see. That is a
real exception — if you wrote the controller an hour ago, re-reading it through
a subagent is theatre.

But weigh it honestly, because the cost of getting this wrong is asymmetric: a
skipped sweep saves a Haiku spawn, and a hallucinated field name costs me a
frontend built against a contract that doesn't exist. So:

- **Wrote it this session, still in context** → use it. Still open the file to
  copy types verbatim; never transcribe a schema from memory.
- **Wrote it this session but context has been summarised**, or you're
  reconstructing from your own notes rather than reading it → sweep. A
  summarised memory of a schema is a paraphrase, and paraphrases drift.
- **Anything you did not write** — pre-existing endpoints in scope, the base
  branch's shape, who still calls a route you deleted → sweep. No exceptions;
  this is the majority of a handover for anything beyond a one-endpoint change.

Uncertain which side of the line you're on is itself the answer: sweep. If you
find yourself opening a third file to check something, the spawn was due two
files ago.

Confirm `plans/` is gitignored: `git check-ignore -q plans/`. If it is not, stop
and tell me — this file is a session baton, not repo documentation, and it
should not turn up in a PR.

## Phase 2 — Write it

`plans/<slug>-frontend-handover.md`, always that suffix, always `plans/` and
never `.claude/plans/`. Both are in use today and the split is the reason I
can't find the last one.

Past a couple of files' worth of reading, brief a `tradie` with the phase 1
output and this template. You holding the design is what makes the brief cheap;
it does not make your output tokens on a 250-line document cheap. Pass the
verbatim schema text through in the brief — a tradie sent to go and find the
types itself will paraphrase them.

**Types are copied, never retyped.** A Zod schema paraphrased from memory drifts
from the code silently, and the frontend builds against the paraphrase. Paste
the source and cite `file:line` next to it.

### The template

Same section names every time, in this order, always `##` and never `###`. Two
handovers from two repos should read as one document — that is most of the value
of having a skill at all, and it is what lets me find the section I want without
reading the file. Don't rename a section because it fits this feature better;
don't add one because this feature felt special. **Omit any section that would
be empty**; never write a heading followed by "none".

```markdown
# <Feature> — Frontend Handover

Against `<branch>` @ `<short-sha>`.

## What changed
Bullets, eight at most. Each one says what the frontend must now do
differently. Not what I built — what changed for the caller.

## Breaking
Routes and fields deleted or renamed. Table: `| Was | Now | Why |`.
Stop-calling-these goes here, at the top, where it cannot be missed.

## Endpoints
Table first: `| Method | Path | Purpose |`. Then per-endpoint detail *only*
where the shape isn't obvious from Types — an unusual status code, a
side effect, a field that means something non-literal.

## Types
Verbatim from source, each block tagged with its `file:line`. Request and
response shapes, enums, discriminated unions. This is the section the frontend
copies wholesale, so it has to be exact.

## Errors
Table: `| Status | Code | When it fires |`. Include the app error code if the
repo has them — the number is what the frontend switches on.

## Behaviour the types don't show
Lifecycle and state transitions, async gaps, ordering guarantees, idempotency,
rate limits, pagination and filter semantics, which nullable fields are only
populated in which states. This is where a constraint with a UI consequence
belongs: state the constraint, not the consequence.

## Not built
Deliberate gaps, each with what would resolve it. A gap named without its
resolution is just a shrug.
```

Auth, base URLs and environments go in only when **they changed**. Restating
unchanged infrastructure in a delta doc is how 724-line handovers happen.

Under 250 lines. Past that, ask whether this is really one handover — a doc
covering two features is two docs, and the frontend will read one of them.

## The UI line, precisely

The distinction that matters: **what a field is for** is contract, and belongs
in the doc. **How to render it** is not yours.

Legitimate — the name doesn't carry the purpose, so say it:

> `tenant` is embedded on each invitation so the recipient can tell who invited
> them without a second call.

Not yours, in every case:

| Don't write | Write instead |
|---|---|
| "Show a spinner while the sync runs" | "`status` stays `syncing` until terminal; no push channel, so poll `GET /syncs/:id` — rate limit 30/min" |
| "Use an accordion with a section per property" | *nothing* — that is a layout decision |
| "Colour the badge red on failure" | "`status` is `idle \| syncing \| failed`; `failedAt` and `failureReason` are set only on `failed`" |
| "Disable submit until the dates validate" | "`POST` returns 400 (code 2025) when `startsAt` is after `endsAt`" |
| "Add a confirmation dialog before deleting" | "`DELETE` is hard and not reversible; there is no restore endpoint" |

Notice what happens in the right-hand column: it is *more* useful, because it
names the actual mechanism. The frontend can build the spinner from "no push
channel, poll at 30/min"; it cannot build anything from "show a spinner".

Never write: a component or element name, a colour, a screen or page name, a
layout, "the user will/should see", "you should display", or a suggested UI
section of any kind.

## Phase 3 — Verify, then hand me the path

You wrote a contract. A wrong one is worse than none, because the frontend
builds against it and finds out in integration.

Check every path, field name, enum value and error code against the source —
not against your memory of writing it. Spot-check the load-bearing ones
yourself; hand a `handyman` the full list to confirm each one exists, since that
is exactly the mechanical sweep it is for. A subagent reporting the doc is
accurate is a claim — the fields I care about, you verify. Then run the leak check on your own
draft:

```bash
grep -inE 'spinner|modal|accordion|dropdown|tooltip|button|badge|toast|banner|component|screen|page|layout|colou?r|\bred\b|\bgreen\b|\bgrey\b|\bgray\b|tailwind|css|the user (will|should|sees)|you (should|can) (show|display|render)' plans/<slug>-frontend-handover.md
```

Hits are not automatically wrong — "page" in `?page=2` is fine, and so is a
field genuinely called `colour`. Read each one and decide. A hit you can't
justify as contract is a line to delete.

Then give me **the path and three lines**: what the frontend must change first,
anything I flagged as not built, and the SHA it was written against. Do not
paste the document back into the chat. I am about to open it in the other
session; echoing it here charges me twice for the same tokens.
