---
name: delegation-brief
description: How to write a spawn prompt that a subagent can succeed on first try. Load before delegating to scientist, tradie, or handyman, or before spawning an agent-team teammate.
when_to_use: Whenever you are about to spawn a subagent or teammate, or when a subagent came back with work that missed the point.
user-invocable: false
---

A subagent starts fresh. It inherits CLAUDE.md and the working directory — it
does **not** inherit this conversation, the files already read, or the reasoning
that led here. Everything it needs must be in the spawn prompt.

## Every spawn prompt has four parts

1. **Goal** — the outcome, in one sentence, in terms of the end state rather
   than the activity. "The auth middleware rejects expired tokens with 401" beats
   "look at the auth middleware".
2. **Scope** — the exact file paths in play, and explicitly what is off-limits.
   Without a boundary, an agent widens until it runs out of turns.
3. **Constraints** — the decisions already made. Conventions, the chosen
   approach, what not to touch, what was already tried and failed. Say *why* on
   the non-obvious ones or they'll be re-litigated.
4. **Return shape** — what you want back and how long. "The diff and any failing
   test" gets a diff. "Report on what you did" gets an essay.

## The failure modes, and the fix

| Symptom | Cause | Fix |
|---|---|---|
| Came back with an essay | No return shape given | Specify sections and a length cap |
| Rewrote things you didn't ask about | No scope boundary | Name the files, name what's off-limits |
| Re-proposed an approach you rejected | Constraint omitted as "obvious" | State rejected options and why |
| Two agents clobbered a file | Overlapping scope | One agent, one file set. Always |
| Stopped early, work half done | Task too large for one hop | Split into units with a clear deliverable each |
| Confidently wrong about the codebase | Asked to infer, not to look | Give paths, or send `handyman` first |

## Sizing

Right-sized is one self-contained deliverable: a function, a test file, a
review, an answer. Too small and coordination costs more than the work; too
large and the agent drifts for a long time before you find out.

## Before you spawn, check

- Would a person with only this text and the repo know what to do? If not, the
  agent won't either.
- Are you delegating a decision you haven't made? Don't — decide it or send it
  to a scientist as a decision, not as a task.
- Is this cheaper to just do yourself? Spawn overhead is real. Under roughly two
  minutes of your own work, do it yourself.
