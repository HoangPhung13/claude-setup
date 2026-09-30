<!-- Scope: every output style. Primary under Design. Loads every session. -->

# Design non-negotiables

These hold for any product, any platform, in Figma and in code. A screen
built in code obeys them the same as a frame in Figma. Break one only with a
reason, written where the break is.

## Rhythm

- **Everything is a multiple of 4.** Spacing, size & radius all snap to the
  4-unit grid. An off-grid value is a decision with a reason next to it, not
  a rounding error.
- **Fix the container, not the scale.** Type metrics can push a box off-grid
  — tall ascenders, platform padding. Adjust the container, don't fudge the
  type scale to compensate.

## Layout

- **Responsive first.** A fixed height or width is an escape hatch, never
  the first-class solution, and it needs a reason.
- **Anything with text or an emoji must be able to grow.** Text scales with
  the OS setting, so a hard height clips descenders at 1.2× and worse above.
- **Size in this order: intrinsic, then min-constraint, then literal.** A
  literal pixel value is the last resort, not the first instinct.
- **A spec's size is the size at 1×.** A design document giving a size
  states the 1× value — it's not a constraint on the box.
- **Static height stays at the minimum the content genuinely needs.**

## Tokens

- **No raw values.** Colour, spacing, radius & type all come from semantic
  tokens, in Figma variables and in code alike.
- **Light & dark are one system in two lightnesses, not two moods.** Parity
  is a hard requirement, checked per surface.

## Colour

- **Colour is never the sole signal.** Pair it with shape, icon, or label.
- **Palettes stay colour-blind-safe.** Text contrast meets WCAG AA in both
  lightnesses, checked with a tool, not eyeballed.

## Type

- **Hierarchy comes from type roles & spacing**, never ad-hoc bolding or
  sizing. A missing tier is a type-scale decision, not a one-off.
- **Weight is baked into the family, not applied as a style** — platforms
  don't reliably synthesise it.

## States

- **Every surface is designed in all its states** — empty, loading, error,
  partial, long content — before the happy path is polished.
- **Every pressable has four states**: idle, focused, pressed, disabled. Add
  hover where a pointer exists.
- **Test content is long, wrapped & translated-length by default.** Never
  the shortest label.

## Input

- **Tap is the baseline for every action.** Gestures & animation are
  additive delight, never the only path & never load-bearing. Reduced-motion
  is respected.
- **Touch targets are at least 44 pt**, whatever the visual size inside them.

## Platform

- **Default to native components.** Never hand-draw chrome the OS provides.
- **Chrome carries brand as tint.** The content canvas is fully owned.
