---
name: run
description: Executes a written implementation plan through a pipeline of subagents — groups related tasks, builds each group with a fresh implementer plus one review-and-fix pass, then gates the whole branch behind three parallel skeptical reviewers with different lenses and an evidence-based build-and-test QC gate. The controller never reads source or edits code; it dispatches, consolidates, and decides. Use when a plan or task list exists and the user wants it built — "execute this plan", "build out the plan", "run the plan", or resuming an interrupted run. Producing the plan goes to plan; one scoped change with no plan goes to build. Commits as it goes, pushes only on request, never merges.
---

# run

A plan needs building. You direct implementers who write the code, reviewers
who try to break it, and fixers who repair what reviewers find. You dispatch,
consolidate, commit, and decide. You never read source or edit files.

## Rules

- **You conduct; you never play.** No reading source or diffs, no code, no
  fixes. Allowed: reading the plan, read-only git metadata (`merge-base`,
  `rev-parse`, `log --oneline`, `status`), and the checklist.
- **Sequential groups, fresh agents.** One group at a time, each with its own
  fresh implementer. Never two implementers at once.
- **Adversarial review.** Reviewers assume the work is wrong until they have
  read the code and proven otherwise.
- **Everyone who changes code commits.** Reviewers and QC change nothing.
- **Feature branch only. Never merge. Push only when asked.**
- **Never guess BUILD or TEST.**
- **Strike budget, then the user.** Three QC failures → stop and ask.

## Subagents

Subagents are cheap. Every implementer, reviewer, fixer, and QC is a fresh
dispatch with a self-contained brief from the bundled prompts; subagents never
read the plan file, you paste the task text in. Reviewers run in parallel;
writers always run alone. Without subagents, run each brief yourself in
sequence in a fresh pass from the brief only, and still never read code
outside a dispatch role.

## Pipeline

0. **Setup** — read the plan, group tasks, branch, BASE, BUILD/TEST,
   checklist, cost gate.
1. **Per group** — implementer → reviewer → fixer (skipped on clean PASS).
2. **Panel** — three lensed reviewers over the whole branch, in parallel →
   consolidate → sequential fixers.
3. **QC** — build and tests. MERGEABLE → report. NOT_MERGEABLE → route
   blockers, retry. Third strike → ask the user.

### Phase 0 — Setup

1. **Resuming?** Read the checklist and `git log --oneline`. Re-derive the
   groups from the plan and resume at the first group or phase not done,
   from its last commit. Read the strike count from the QC item.
2. **Read the plan once.** Extract every task with its full text and
   surrounding context. You will paste this into dispatches.
3. **Group related tasks** (see Grouping). If the plan already has groups,
   use them.
4. **Branch**: confirm a feature branch, not main or master. If not, create
   `run/<slug>` or get consent.
5. **BASE** = `git merge-base HEAD <main or base branch>`. Every whole-branch
   review is `BASE..HEAD`.
6. **BUILD and TEST** from the plan header first, then the README or
   manifest. Cannot find them? Ask. None? Record `none (user-confirmed)`.
7. **Cost gate**: if the user did not name this skill, confirm in one
   message: the group count and rough dispatch count.
8. **Checklist**: one item per group, plus Panel, Panel fixes, QC
   (strikes 0/3).

### Phase 1 — Build each group

For each group, in order:

1. **Implementer** (`implementer-prompt.md`): fresh dispatch with the full
   task text, context, branch, and TEST. Note HEAD first; the group's range
   is `<noted HEAD>..HEAD` after it returns.
2. **Reviewer** (`reviewer-prompt.md`, lens `overall correctness and spec
   compliance`) over that range. Concerns from the implementer go in
   verbatim.
3. **Fixer** (`fixer-prompt.md`) only if the reviewer found something. One
   pass, no re-review. Exception: a finding that is both critical and
   complex gets one extra reviewer-plus-fixer round on that issue alone.
   PARTIALLY_FIXED or COULD_NOT_FIX → record it in the group's checklist
   item and carry it into the panel briefs. Never re-loop Phase 1.
4. Mark the group done and move on.

Phase 1 is a light gate. The deep scrutiny comes over the integrated whole.

### Phase 2 — Panel

Only after every group passed Phase 1.

1. **Three reviewers in parallel** over `BASE..HEAD`, one lens each, chosen
   for this work (see Choosing lenses). Include carried findings in every
   brief.
2. **Consolidate from finding text only**: match by `file:line` and
   description, drop duplicates, discard anything not a real defect. Do not
   open code to dedupe; if two findings cannot be merged from their text,
   keep both.
3. **Fix**: one fixer for everything when issues are few or related, else
   one fixer per issue, run sequentially. Each commits.

### Phase 3 — QC

1. **QC** (`qc-prompt.md`) with `BASE..HEAD`, BUILD, TEST, and the plan's
   requirements. It runs build and tests and returns MERGEABLE or
   NOT_MERGEABLE with typed blockers.
2. **MERGEABLE** → report. Push only if asked. Never merge.
3. **NOT_MERGEABLE** → route each blocker: `[defect]` to a fixer,
   sequentially; `[implementation]` back to that group's implementer, or to
   the user if the plan itself is at fault. Re-run QC with the retry section
   filled.
4. **Three strikes, then the user.** Update the count on each NOT_MERGEABLE.
   On the third, stop and ask the user how to proceed.

## Grouping

Group tasks that are tightly coupled so one implementer builds them in one
coherent pass: same files or module, one depends on another's output, or
splitting would force a second agent to re-learn the first's context. Keep
groups independent where you can. Small enough to hold in context, large
enough to stand alone. Derive grouping from the plan, not from reading code.

## Choosing lenses

Pick three that fit this work so the reviewers cover different failures:

- backend or logic-heavy: spec compliance · correctness and edge cases ·
  security.
- refactor: behavior preservation · simplicity · test coverage.
- data or integration: schema and contracts · error handling · performance.

## Report

- What was built, per group, with commit SHAs.
- Dispatches spent.
- QC evidence: real build and test output lines.
- Left open: unfixed, disputed, or deliberately out of scope.
- Whether you pushed, and that merging is the user's call.

## Status handling

- **Implementer**: DONE → reviewer. DONE_WITH_CONCERNS → reviewer, concerns
  pasted in. NEEDS_CONTEXT → supply it, re-dispatch. BLOCKED → supply
  context, or split the group smaller; never re-dispatch unchanged. Plan
  itself wrong → stop and tell the user.
- **Fixer**: FIXED → proceed. PARTIALLY_FIXED / COULD_NOT_FIX → Phase 1:
  carry into the panel brief. Phase 3: counts toward strikes, re-route next
  round.
- **Reviewer**: PASS / FAIL. **QC**: MERGEABLE / NOT_MERGEABLE.

## Red flags

- Reading source or editing code yourself.
- Two implementers or two fixers at once.
- A reviewer that trusts the implementer's report.
- An unresolved range or a guessed BUILD or TEST handed to a reviewer or QC.
- Skipping the panel because Phase 1 "already reviewed".
- Looping QC past three strikes. Merging the branch.

## Bundled files

- `implementer-prompt.md` — build one group and commit.
- `reviewer-prompt.md` — skeptical reviewer, broad mode for Phase 1, one
  lens each for the panel.
- `fixer-prompt.md` — repair listed findings and commit.
- `qc-prompt.md` — whole-branch MERGEABLE / NOT_MERGEABLE verdict.
