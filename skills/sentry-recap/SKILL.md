---
name: sentry-recap
description: Recap a Sentry org's recurring unresolved issues as three tables (pressing errors, optimisation flagged, potential noise), sorted by Sentry's own priority & recommended rank, with a one-line likely cause read from the code for each pressing error. Read-only. Nothing changes in Sentry, nothing is written to disk, nothing is committed. Use this whenever I ask what's going on in Sentry, for a Sentry or error recap, a daily or weekly error digest, "what's recurring", "what should we fix first", or a triage of a Sentry org or project, even when I don't say "recap".
when_to_use: 'When I name a Sentry org or project & want the state of it, not one issue: "recap sentry for the-eternal", "what is recurring in prod", "daily sentry digest", "triage our errors". For one known issue use sentry:sentry-debug-issue instead. Invoke it before querying Sentry: the first unfiltered query is the one that anchors the recap to raw frequency instead of priority.'
argument-hint: "org=<slug> [env=production] [period=30d] [repos=<project>=<repo>@<branch>,...]"
---

Arguments: $ARGUMENTS

The unit is **one Sentry issue, one table row.** Group nothing. Grouping hides
which issue to open, and every row links to the issue it describes.

## Parameters

Read them off the arguments, then off my words. Ask only for `org` if neither
gives it.

| Parameter | Default | Meaning |
|---|---|---|
| `org` | none | Sentry org slug |
| `env` | `production` | Sentry environment. `all` drops the filter |
| `period` | `30d` | Window for unresolved issues |
| `repos` | discover | `<sentry-project>=<repo dir>@<branch>`, comma separated. Several projects may share one repo |

## Read-only, always

This runs unattended in the cloud as well as on my machine. So:

- Never call Sentry's `update_issue` or Seer analysis. Archiving & resolving are my call.
- Never `checkout`, `switch`, `pull`, `commit` or `push`. `git fetch` is fine. It moves remote refs only, never my working tree.
- Output goes in the chat. No file, no artifact, no PR.

## 1. Pull issues by priority

Sentry tags each issue High, Medium or Low. Its recommended sort ranks issues
within that. Together they are Sentry's suggested order, so use them as the
sort & never re-sort by event count.

Run three `search_issues` calls in parallel, one per priority:

```
query: is:unresolved environment:<env> issue.priority:<high|medium|low>
sort: recommended   period: <period>   limit: 100
```

An issue's rank is its position in its own priority's result, shown as
`High #3`. Drop `environment:` when `env=all`.

## 2. Get counts for the environment

`search_issues` event counts are totals across every environment. For error
issues, get per-environment counts with one `search_errors` call:

```
fields: issue, environment, count()   sort: -count()   period: <period>
```

Use the `<env>` count for error rows. Performance issues (N+1, consecutive
HTTP, app hangs) aren't in the errors dataset. Keep their totals & say so in
one line under the tables.

## 3. Sort each issue into a table

**Optimisation flagged** is any performance-category issue: N+1 query, N+1 API
call, consecutive HTTP or DB, slow DB query, large payload, app hang, slow
frame. They're inefficiency, not failure, even when Sentry ranks one High.

**Potential noise** is an error that isn't the code's fault:

- Script injected by a browser extension (`__firefox__`, `window.ethereum`, `chrome-extension://` frames)
- Expected user outcome: wrong or expired verification code, expired link, "try again later"
- Transient device networking: `Network request failed`, timeouts, offline
- Device-side limits, e.g. FCM `TOO_MANY_REGISTRATIONS`
- Malformed client requests, e.g. multipart boundary missing
- Harmless double-initialisation that only needs a guard

Noise that's escalating or hitting many users goes to pressing anyway. Volume
turns a shrug into a bug.

**Pressing errors** is everything else.

Mark a row **New** when the issue was first seen in the last 24 hours. On a
daily run that's what changed since yesterday.

## 4. Find a likely cause for pressing errors

For each pressing row, up to the first 10 by rank:

1. `get_sentry_resource` on the issue. Take the top first-party frame's file & line.
2. Resolve the repo & branch (below) & read that spot in the code.
3. Write one line: the likely cause & `file:line`. If the frame is all third-party or the code doesn't explain it, say what the error says & stop. A guess written as a cause sends me to the wrong file.

Suggest no fix. A fix is a follow-up run on its own, usually with
sentry:sentry-debug-issue.

### Resolving repo & branch

- **Repo:** take it from `repos`. Without `repos`, look in the current directory & its sibling directories for a git repo whose name or `origin` URL contains the Sentry project slug. If none match, skip the code lookup for that project & say so in one line.
- **Branch:** take it from `repos`. Otherwise use `main` if `origin/main` exists, then the repo's default branch (`git symbolic-ref refs/remotes/origin/HEAD`).
- **Reading:** run `git fetch origin <branch>`. If the checkout is already on `<branch>`, read the working tree & note when it's behind `origin/<branch>`. Otherwise read the remote ref without touching the tree: `git show origin/<branch>:<path>` & `git grep <pattern> origin/<branch>`. My local trees are often dirty or on a feature branch, and a cloud clone starts on the default branch, which may not be `<branch>`.

## Output

Open with one sentence: how many unresolved issues are in `<env>` over
`<period>`, & how Sentry split them by priority. If one priority holds every
error, say so, because then the rank is the only ordering signal. Then use the
three tables in this order, with these columns:

```markdown
## Pressing errors

| Rank | Issue | Events / users | Last seen | What's wrong |
|---|---|---|---|---|
| High #1 | **New** [API-1M](<issue url>) | 572 / 36 | 1d | Play `acknowledge` rejects the token (`services/subscriptions.ts:1230`) |

## Optimisation flagged

| Rank | Issue | Where | Events / users | Last seen | Flag |
|---|---|---|---|---|---|

## Potential noise

| Rank | Issue | Events | Why it's noise | Action |
|---|---|---|---|---|
```

- Sort every table by priority (High, then Medium, then Low), then by rank.
- Last seen is short: `20m`, `6h`, `3d`.
- Action is one of: add to `ignoreErrors`, report as a warning, don't report, archive, add a guard.
- Leave out an empty table & say in one line that there's nothing there.
- Finish with the Sentry search URL for the query, then stop. No summary paragraph.
