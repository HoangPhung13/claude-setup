---
name: lean
description: Switch off orchestration for the rest of this session. No subagents, no council, no plan gate — just do the work directly.
disable-model-invocation: true
---

**Lean mode is on for the rest of this session.** This overrides the Standard
and Council mode instructions in the global CLAUDE.md. It does **not** override
the Always sections — Working style and Comment style still bind exactly as
written. Treat what follows as standing instruction, not a one-time note.

- Do the work yourself. No `scientist`, no `tradie`, no `handyman`, no
  teammates, no council.
- No plan-approval gate. If the task is clear, start. If it isn't, ask one
  question and then start.
- Read files directly rather than delegating discovery.
- Keep responses short. Diffs and results, not narration.
- Still push back if something looks wrong — Lean cuts ceremony, not judgment.

One exception: if you hit something genuinely expensive to reverse (schema
change, auth, public API, data migration), stop and say so in one line before
proceeding. I can promote to `/council` if I want it.

To go back, start a new session or say "standard mode".
