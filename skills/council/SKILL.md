---
name: council
description: Deliberate on a decision without building anything. Two adversarial scientists, one synthesis, no execution phase. Use when you want the thinking, not the code.
disable-model-invocation: true
argument-hint: "[the decision to think through]"
---

Decision: $ARGUMENTS

This is deliberation only. **Nothing gets built in this turn**, no matter how
clear the answer becomes. If I want it built, I'll say so.

## Run

1. **Recon properly, once, and share it.** Unless this is a pure design question
   with no code in it, spend a real `handyman` pass first — several in parallel
   if the surface is wide. Cover the files that would change, what already
   exists that solves part of this, the shape of the data or API involved, and
   any sibling implementation worth copying rather than inventing.

   Be generous. Underfeeding the scientists is the expensive mistake, not
   over-briefing them: a thin brief gets rediscovered twice, at Opus rates, in
   two directions you then have to reconcile.

2. Spawn **two** `scientist` subagents, same facts, opposing framings:
   - **A — build it:** design the best version of the thing as asked.
   - **B — break it:** assume the framing is wrong. Six-month cost, cheaper
     alternative nobody proposed, the goal behind the goal.

   Send them the same brief. Neither knows the other exists. Do not summarise
   your own view into either prompt.

   Tell both they may dispatch their own `handyman` for anything the brief left
   out, and that you want it. They're framed differently, so they'll go looking
   for different things — that divergence is signal, and it's Haiku, so overlap
   costs almost nothing. What they must not do is explore broadly at their own
   rate; targeted gap-filling only.

3. If both come back agreeing, that's a signal about your prompts, not about the
   problem. Re-run B harder before you report consensus.

## Report

Under 400 words:

- **The real question** — often not the one I asked. If A and B disagree about
  what the problem is, that disagreement is the finding, and everything below it
  is provisional.
- **The deciding conflict** — the single tradeoff the answer turns on.
- **Recommendation** — one, chosen, with the losing case stated fairly.
- **What would change this** — the fact that would flip it, and how to check
  that fact cheaply.
- **Gaps they filled themselves** — what each one went looking for that my brief
  didn't give them. Two different lists means they reasoned from partly
  different facts, so weigh the disagreement accordingly; it also tells me what
  step 1 should have covered.

No implementation plan, no file lists, no code. If the recommendation is "do
nothing yet", say that plainly — it's frequently correct and rarely offered.

## Handoff

The no-build constraint above is scoped to **this turn only**. This skill's text
stays in context for the rest of the session, so read it as "don't build now",
not "don't build ever" — a later `/orchestrate` supersedes it and should not be
refused on the grounds that a council said not to build.

Your synthesis also stays in context. When `/orchestrate` runs next, it will
reuse this recommendation rather than re-deliberating. So write the
recommendation as something that can be planned against: concrete enough that
someone could turn it into file changes without asking you what you meant.
