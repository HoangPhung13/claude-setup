<!-- Scope: Default output style only. Out of scope under Design, which says
     so in its style file. Loads every session regardless. -->

# Coding rules

Applies when the active output style is Default. Under Design this file has
loaded but does not apply.

## Comment style

**Default to no comments.** Names carry the meaning. Before writing one, try a
clearer name or a small extraction — a comment is the fallback, not the habit.

Write one only for what the code genuinely can't say: a hidden constraint, a
non-obvious invariant, a workaround, a surprising external behaviour. **Two
sentences maximum.** If it truly needs more, use bullets, not a paragraph.

```typescript
// Each command can only have maximum of 10 parameters. Hence splitting into
// chunks of 10s.
// Reference: https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameters.html
const CHUNK_SIZE = 10;
```

Never:
- Restate the code (`// loop through the items`).
- Reference a task, ticket, or PR — that's the commit message's job.
- Narrate change history (`// changed this to fix the bug`) — that's git's job.
- Leave commented-out code. Delete it.

Cite sources for non-obvious behaviour with a `// Reference:` line, matching
whatever citation style the repo already uses. Only cite what a teammate or
their agent can actually open: a public URL, or a version-tracked path in this
repo. Never a local-only file — a personal planning doc, a scratch note, an
absolute path on my machine.

`// TODO:` is fine for a deliberate known gap, and must say *what would resolve
it*. A TODO that only names the problem is a comment restating the code.
