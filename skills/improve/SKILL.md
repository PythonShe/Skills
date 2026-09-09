---
name: improve
description: Unattended improvement session that finds its own worklist and ships it — three parallel read-only scouts study the repo from its users' perspective (small non-breaking features, quick fixes, polish), a selector composes a 3-5 job docket sized to one session, each job is built by a fresh implementer and reviewed, and the whole branch is sealed by two final reviewers plus a build-and-test QC gate. Announces the docket rather than asking, runs on a fresh improve/ branch, commits as it goes, keeps a backlog of what it did not pick, pushes only on request, never merges. Use when the user wants the project improved without naming what — "find some quick wins and ship them", "make this repo better, you pick". A named change goes to build; a bug goes to fix.
---

# improve

No brief, no plan, no named target. You find what is worth doing, decide what
fits, build it, and prove it shipped clean. Announce, do not ask.

## Rules

- **You conduct; you never play.** No reading source or diffs, no code, no
  fixes. Every change happens inside a dispatch; only compact reports return.
  Allowed: reading reports, read-only git metadata, the checklist, and the
  backlog file (composed from reports, never from code).
- **Sit in the user's seat.** Scouts hunt for what this project's actual
  users would notice and thank you for. The maintainer's backlog is not the
  assignment.
- **The docket is sized to the session.** Three to five non-breaking jobs.
  The selector composes it. You may strike a job that violates scope; you
  never add one.
- **Announce, do not ask.** The only stops: the cost gate, Phase 0 blockers
  only the user can resolve, and the QC two-strike escalation.
- **Shrink the docket rather than stall.** A job that blocks twice is dropped
  and reported. A four-job branch that ships beats a five-job branch that
  does not.
- **One writer at a time. Everyone who writes commits.**
- **The road not taken is kept.** A run that reaches its report ends by
  writing the backlog at `docs/skills/improve-backlog.md` in the target
  project.
- **Never build on main or master. Never merge. Push only when asked.**

## Subagents

Subagents are cheap. Every scout, selector, implementer, reviewer, fixer, and
QC is a fresh dispatch with a self-contained brief from the bundled prompts.
Run independent dispatches in parallel; writers always run alone. Without
subagents, run each brief yourself in sequence in a fresh pass from the brief
only, and still never read code outside a dispatch role.

## Pipeline

0. **Warm-up** — clean tree, BUILD/TEST, cost gate, cut `improve/<date>`,
   BASE, checklist.
1. **Scouting** — 3 read-only scouts in parallel: features, fixes, polish.
2. **Docket** — one selector merges, scores, sizes. Announce and go.
3. **Build loop** — per job: implementer → reviewer → fixer if findings.
   Blocked twice → drop.
4. **Finale** — two whole-branch reviewers in parallel → fixers → QC. Two
   strikes → ask the user.
5. **Backlog** — write `docs/skills/improve-backlog.md`, commit, report.

### Phase 0 — Warm-up

1. **Resuming?** Read the checklist and `git log --oneline`. Resume at the
   first item not done. BASE lost? The branch is always cut fresh, so
   `git merge-base HEAD <source branch>` recovers it.
2. **Clean tree.** Uncommitted changes are the user's work. Dirty
   `git status` → stop and ask before anything else.
3. **BUILD and TEST** from the README or package manifest. Cannot find them?
   Ask now, never mid-run. None? Record `none (user-confirmed)`.
4. **Cost gate**: if the user did not name this skill, confirm in one
   message: an unattended session on a fresh branch that picks and builds
   its own 3-5 jobs, with a rough dispatch count of ten to twenty.
5. **Cut `improve/<YYYY-MM-DD>`** from HEAD (suffix `-2` on collision).
   `BASE = git rev-parse HEAD`. Note the branch you cut from.
6. **Checklist**: Scouts, Docket, Finale review (BASE: sha, cut from:
   branch), Finale QC (strikes 0/2), Backlog, Report. Job items are inserted
   after Phase 2. If the harness has no task list, keep it in a scratch
   note.

### Phase 1 — Scouting

1. **Three scouts in parallel** (`scout-prompt.md`), one lens each:
   - **features**: small new capabilities users would ask for next.
     Non-breaking, no schema, dependency, or public-interface upheaval.
   - **fixes**: real defects, papercuts, misleading errors, stale docs,
     broken examples. Each verified against the code.
   - **polish**: surfaces users touch: UI and UX, CLI output, onboarding,
     naming, formatting, accessibility.
2. **Brief in the backlog.** If `docs/skills/improve-backlog.md` exists,
   paste each lens's entries verbatim into that scout's brief. Scouts
   re-verify every carried entry; stale ones come back on a Stale backlog
   line so Phase 5 prunes them.
3. Scouts never edit, build, or test.
4. Each returns at most six candidates with title, benefit, `file:line`
   evidence, size (S/M), risk, `breaking: no`, and leverage (an installed
   skill worth loading to build it, or none), plus local patterns.
5. A scout returning NONE or BLOCKED is noted; the others proceed. All three
   empty → report honestly and end. An empty docket on a polished repo is a
   finding.

### Phase 2 — Docket

1. **One selector** (`selector-prompt.md`) with all three reports pasted
   verbatim. It merges duplicates, scores value against effort and risk,
   rejects breaking, oversized, or speculative candidates, and returns 3-5
   jobs in build order, each with brief, acceptance, anchors, patterns, size,
   and risk, plus notable rejects with a backlog score.
2. **Scope check only.** Strike a job that is breaking or cannot fit. Fewer
   than three survivors is fine; never pad with rejects.
3. **Announce the docket** in one line per job, and the top rejects with
   reasons. Do not wait for a reply. Insert one checklist item per job
   carrying its full docket block verbatim (or in the scratch note); that is
   the docket's only durable copy.

### Phase 3 — Build loop

For each job, in order, one at a time:

1. **Implementer** (`implementer-prompt.md`): fresh dispatch with the job's
   brief, acceptance, anchors, patterns, branch, and TEST pasted in. Note
   HEAD first; the job's range is `<noted HEAD>..HEAD` after it returns.
2. **Reviewer** (`reviewer-prompt.md`, job mode) over that range. Every job
   gets one. DONE_WITH_CONCERNS goes into its brief verbatim.
3. **Fixer** (`fixer-prompt.md`) only when the review found something. One
   pass. Unfixed or disputed findings are appended to the finale checklist
   item the moment they exist and carried into the finale briefs.
4. **Drop rule.** NEEDS_CONTEXT or BLOCKED → re-dispatch once with the
   missing context. Second failure → mark `dropped: <reason>`, announce in
   one line, move on. Partial commits landed? Dispatch a fixer to revert them
   so the dropped job leaves no trace.
5. Mark the job done and continue.

### Phase 4 — Finale

1. **Two reviewers in parallel** over `BASE..HEAD` (`reviewer-prompt.md`,
   finale mode), briefed with the docket, carried findings, and any
   dropped-job reverts to double-check:
   - lens **integrated correctness and cohesion**: do the jobs compose into
     one sound branch.
   - lens **user impact and regression risk**: did this session make things
     better for users, and does anything regress.
   Neither is skippable, and clean per-job reviews do not thin this gate.
2. **Consolidate from finding text**: match by `file:line`, drop duplicates.
   Route to fixers sequentially, each committing.
3. **QC** (`qc-prompt.md`) with `BASE..HEAD`, BUILD, TEST, and the docket
   annotated per job as shipped, dropped, or amputated, with acceptance.
4. **NOT_SHIPPABLE** → route each blocker: `[defect]` to a fixer,
   `[implementation]` back to that job's implementer, or **amputate**: a
   fixer reverts that job's commits cleanly and the docket shrinks. Re-run QC
   in retry mode.
5. **Two strikes, then the user.** Update the strike count as each accrues.
   On the second, stop and ask: keep iterating, amputate to what is sound and
   ship that, or hand the failing work to `fix` if it is a bug.

### Phase 5 — Backlog

1. **Create or update `docs/skills/improve-backlog.md`** from the selector's
   report and the settled docket, never from code. Three sections by lens;
   one line per entry:
   `[P<score>] Title — benefit · evidence · rejected <date> | dropped <date> (<reason>)`.
   Dropped and amputated jobs enter at P4 with their reason.
2. **Supersede in place, cap at twelve.** Remove entries that shipped, that
   a scout marked stale, or that scored `no`. Over the cap, lowest scores go.
   Stamp with the date and short HEAD. It states what is still worth doing,
   never history.
3. **Commit it on the branch**, then report. An EMPTY-docket run still
   writes the rejects. A run abandoned at escalation writes nothing.

## Report

- **The docket, settled**: per job, shipped with SHAs, dropped with reason,
  or amputated. One table.
- **What the scouts saw**: candidates per lens, top rejects, and the
  breaking-but-interesting ideas ruled out.
- **Dispatches spent**: scouts, selector, implementers, reviewers, fixers,
  QC rounds.
- **QC evidence**: real build and test output lines.
- **The backlog**: entries written, superseded, pruned.
- Whether you pushed, and that merging is the user's call.

## Status handling

- **Scout**: FOUND → selector. NONE / BLOCKED → note, others proceed.
- **Selector**: DOCKET → announce and build. EMPTY → report and end.
- **Implementer**: DONE / DONE_WITH_CONCERNS → reviewer. NEEDS_CONTEXT /
  BLOCKED → drop rule.
- **Fixer**: FIXED → proceed. PARTIALLY_FIXED / COULD_NOT_FIX → carry into
  the finale briefs or the next QC round. Never re-loop.
- **Reviewer**: PASS / FAIL. **QC**: SHIPPABLE / NOT_SHIPPABLE.

## Red flags

- Reading source or editing code yourself.
- Pausing to ask a question the pipeline answers.
- Adding a job the selector did not pick, or padding a thin docket.
- A breaking change, schema migration, or dependency upheaval in the docket.
- Two writers at once. Skipping a job review or either finale reviewer.
- Fighting a blocked job past the drop rule.
- Guessing BUILD or TEST. Building on main. Looping QC past two strikes.
- Letting the backlog grow past its cap or become a changelog.

## Bundled files

- `scout-prompt.md` — read-only discovery by lens, from the user's seat.
- `selector-prompt.md` — merge scout reports into the docket.
- `implementer-prompt.md` — build one docket job and commit.
- `reviewer-prompt.md` — job mode over one range, finale mode with a lens.
- `fixer-prompt.md` — repair findings or revert a job, and commit.
- `qc-prompt.md` — whole-branch shippability verdict, with retry mode.
