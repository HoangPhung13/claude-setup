---
name: design-brief
description: Clarify a design brief & check it against the design non-negotiables and the project's design system before any Figma operation runs. The plan gate for design mode — the equivalent of orchestrate's phase 1–3 for a screen or flow. Produces a brief file, stops for my go, then hands to the Figma operation skills.
when_to_use: 'Under the Design output style, whenever a Figma write is about to happen and no settled brief exists in this conversation — "design the X screen", "build the Y flow in Figma", "port the tokens to Figma", a Figma URL with "make this…". Also when I say /design-brief by name. Invoke it before reading anything: phase 1 does recon through handyman on Haiku. Never skip it because the ask seems small — a small ask with no brief is how a session jumps straight to building.'
argument-hint: "[what to design], or [resume <brief-slug>]"
allowed-tools: AskUserQuestion
---

## Two rules that override everything else in this file

**No `use_figma` write until the brief file exists & I've said go.** Reads —
`get_metadata`, `get_screenshot`, `get_variable_defs`, `search_design_system`
— are fine during recon.

**The brief records decisions, not options.** A field you couldn't settle is
written as "unknown, assumed X" — never left blank, never padded with a menu.

## Phase 0 — Resume, or start

Derive `<project-slug>` from the working directory's basename. If
`~/.claude/design/<project-slug>/` has a brief matching the ask, or I named
one, read it & continue from its recorded phase. Otherwise start.

## Phase 1 — Recon, through handyman

One `handyman` spawn, in parallel with in-session Figma reads if a Figma URL
or file was given.

Handyman locates & reads the project's design context wherever it lives —
`.claude/rules/design.md` if the project has one, else design sections of
CLAUDE.md / AGENTS.md, `docs/`, a theme or tokens directory, a
`design-system.md` — and returns: the token names & values in play, type
roles & their families, state rules, product rules that constrain design, any
existing screen or component this overlaps.

In-session, yourself: `get_metadata` & `get_screenshot` of the target node,
`get_variable_defs` / `search_design_system` to learn what the Figma library
already has.

**Branch:** if the project has no Figma library yet — no variables, no
components — record that. Phase 3 routes differently.

## Phase 2 — The brief, then stop

Write `~/.claude/design/<project-slug>/<brief-slug>-brief.md` with every field
answered or marked "unknown, assumed X":

- **Surface** — which screen/flow, entry point, platform(s) & breakpoints.
- **Problem** — what the user can't do or gets wrong today. One sentence.
- **States** — every state this surface renders, checked against
  `rules/design.md` (surface states: empty, loading, error, partial, long
  content; pressable states: idle, focused, pressed, disabled, hover where
  pointer) & against the project's own state rules. List them; anything
  missing is a flag.
- **Type roles** — which role carries each text tier, by role name. A tier
  with no role is a type-scale decision → flag.
- **Tokens** — which tokens from the project's system are in play. A value
  not in the system is a flag, not a decision.
- **Rhythm & layout** — anything off the 4-pt grid or fixed-size, with its
  reason. Empty is the expected answer.
- **Constraints** — what's locked (product philosophy, existing components,
  platform chrome) & what's open.
- **Done looks like** — the frames/components that will exist, named, & what
  they'll be reviewed against.
- **Handoff** — which operation skill (phase 3) & the section order.
- **Flags** — everything above that's a flag, in one list. If any flag fights
  locked product philosophy or the design system, say so & defer to
  `/council` rather than resolve it here.

Then use `AskUserQuestion` only for genuine forks, present the brief path &
the flags, and **stop for my go.** Do not proceed on silence.

## Phase 3 — Handoff

On go, name the operation skill by content:

- Composed screen / modal / drawer / multi-section view → `figma-generate-design`
  (with `figma-use`).
- Tokens, variables, components, variant sets → `figma-generate-library`
  (with `figma-use`).
- Surgical edits to existing nodes → `figma-use` alone.
- **No-library branch:** the first job is the library, not the screen. Route
  to `figma-generate-library` from the project's theme/token source, with the
  project's reference docs as what the library is checked against. The screen
  brief waits until the library exists.

Spawn `draughtsman` per brief section, carrying the brief's Tokens, Type roles &
States tables verbatim in the spawn prompt — subagents don't see the
conversation or the style. After each return, `get_screenshot` the
**enclosing container** — sub-section, section, or page — never the built
node itself (a node render crops its own overflow & hides neighbours), &
check against "Done looks like" yourself. Read the worker's overflow &
sibling audit lines; a return without both goes back. Update the brief file's recorded phase &
the page's `Notes` frame as sections land.

## Finishing

Brief file marked done, Notes frame current, list of frames/components
created with node ids. No summary of the design back to me — I can open the
file.

**Figma quota line.** After every brief, one line: calls this brief (mine
+ every draughtsman's reported count) & the day's running total against
the plan's daily cap, e.g. "Figma: 58 this brief · 141/200 today". The
tally lives in the brief file's phase line so it survives a `/clear`.
