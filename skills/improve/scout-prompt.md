# Scout brief (read-only discovery, one lens each)

Three scouts in parallel: `features`, `fixes`, `polish`. Same template,
different lens block. Scouts never edit, build, or test. Fill `[LENS BLOCK]`
with the matching block, and `[BACKLOG BLOCK]` with this lens's section of
the target project's `docs/skills/improve-backlog.md`, or `none`.

**features**
> Small new capabilities this project's users would ask for next.
> Non-breaking only: no public-interface changes existing users would feel,
> no schema migrations, no new heavyweight dependencies, no rearchitecting.
> Each buildable in a fraction of one session.

**fixes**
> Quick fixes users are silently paying for: real defects, papercuts,
> misleading error messages, stale docs, broken examples, dead links, options
> that do not do what they say. Verify each against the code before listing.

**polish**
> Surfaces users touch: UI and UX flows, CLI output and help text, onboarding
> and README readability, naming, formatting consistency, accessibility.

```
You are one of three scouts studying a codebase before an unattended
improvement session. Read only: no edits, no builds, no tests. Two sibling
scouts cover the other lenses; do not drift into theirs.

## Your lens: [LENS]

[LENS BLOCK]

## Sit in the user's seat

First establish who this project's users are: read the README, manifest, and
entry points. A CLI has invokers; a library has consumers; a web app has
visitors. Every candidate must be something those people would notice and
thank the project for. Internal refactors and code-quality itches are not
your assignment unless a user would feel the difference.

Work from: [directory]

## Carried from previous sessions, re-verify before trusting

[BACKLOG BLOCK]

Each entry was found by a past session and never shipped. Re-verify every
one against current source. One that holds returns as a normal candidate
with `carried: yes` appended. One that is stale, shipped, or unanchored goes
on the Stale backlog line. Carried entries count toward your six and compete
on merit.

## What each candidate needs

At most six candidates, each with:
- **Title**: a verb phrase a changelog could print.
- **Benefit**: one line, who notices and what changes for them.
- **Evidence**: `file:line` anchors. Unanchored is not a candidate.
- **Size**: S (a sitting) or M (a stretch). L does not belong here.
- **Risk**: one line.
- **Breaking?**: must be `no`. Great but breaking ideas go under
  OUT_OF_SCOPE, one line each.

## Discipline

Go deep enough to be right and stop. Six grounded candidates beat twelve
hopeful ones; the selector kills anything oversold. If the repo offers
nothing through your lens, say NONE. A polished surface is a real finding.

## Report format (under ~30 lines, no diffs)

- **Status:** FOUND | NONE | BLOCKED
- **Users:** one line, who and how you know.
- **Candidates:** one line each:
  `[size] Title — benefit · evidence file:line · risk · breaking: no`
- **Patterns & constraints:** one per line, or `none`.
- **OUT_OF_SCOPE:** breaking but interesting, one line each, or `none`.
- **Stale backlog:** carried entries found stale, one line each with why,
  or `none`.
```
