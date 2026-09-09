---
name: build
description: Builds one scoped change end to end — scouts the codebase, gets the user's approval on an approach, implements it on a branch, runs two fresh reviews, fixes findings in one pass, and seals with a build-and-test QC gate. Use when the user names one concrete change and wants it built now with no spec or plan first — "add a --json flag", "wire this endpoint", "make the sidebar collapsible", "just build it". Commits as it goes, pushes only on request, never merges. A bug to diagnose goes to fix; an unnamed "just improve things" goes to improve.
---

# build

Recon, a decision, the work, a check. Fast, never careless.

## Rules

- **Nothing is written before the user approves an approach.** The gate in
  Phase 2 is the only approval in this flow. Do not skip it. The one
  exception is the below-the-floor shortcut in Phase 0, which you announce.
- **Never review your own code.** Reviews and QC run in a fresh context: a
  subagent if the harness has them, otherwise a fresh pass that reads only
  the brief and the diff, never your working notes.
- **One writer at a time.** One branch, one seat editing. Never two
  implementers, never a fixer beside an implementer, never you editing while a
  subagent runs.
- **Everyone who changes code commits.** Commits are the progress record and
  the resume trail.
- **Never build on main or master** without explicit consent. Never merge.
  Push only when asked.
- **Never guess BUILD or TEST.** Ask if you cannot find them.
- **Wrong room is a valid outcome.** A change that needs a design decision or
  a plan is reported as such at the gate, not forced through.

## Subagents

Subagents are cheap. Use them for every scout, reviewer, fixer, and QC
dispatch. Each gets a self-contained brief from the bundled prompt files, with
every placeholder filled with real text. Run independent dispatches in
parallel. If the harness has no subagents, run the same brief yourself in
sequence, and treat "fresh context" as: start from the brief and the diff only.

## Pipeline

0. **Setup** — brief, scope check, BRANCH, BUILD/TEST, cut branch, BASE.
1. **Recon** — 1-2 read-only scouts. Landing zone, approaches, size call.
2. **Gate** — report and ask the user to pick. Nothing written before this.
3. **Build** — you for tiny changes, else one implementer. Never both.
4. **Review** — two reviewers in parallel on BASE..HEAD, different lenses.
5. **Fix** — one pass. Skip on clean PASS.
6. **QC** — build, tests, acceptance. Two strikes, then ask the user.

### Phase 0 — Setup

1. **Resuming?** Read your checklist and `git log --oneline`. Resume at the
   first phase not done. If BASE is lost, ask the user for the start commit.
2. **Brief**: restate the request in one or two lines, plus **acceptance**:
   how anyone would know it works. Every dispatch receives this verbatim.
   Ask now if the request is ambiguous in a way that changes what gets built.
3. **Scope check**:
   - Trivial with one obvious answer (rename, typo, one-line config): just do
     it, say you skipped the pipeline, stop.
   - Spans several modules, needs a new subsystem, or rests on a design
     decision the user has not made: say so and stop. Point to `plan`.
   - Otherwise proceed. Unsure? Let the scout's size call decide at the gate.
4. **BRANCH**: use a named branch if the user gave one, the current branch if
   it is already a feature branch, otherwise plan `build/<slug>`. Check
   `git status`: uncommitted changes are the user's work. Surface them and
   resolve with the user before cutting the branch. Never touch them.
5. **BUILD and TEST** from the README or package manifest. Cannot find them?
   Ask. Project has none? Record `none (user-confirmed)` and pass that to QC.
6. **Cost gate**: if the user did not name this skill, ask once, in one
   message: the brief, the branch, and that recon is about to run.
7. **Cut the branch**, then `BASE = git rev-parse HEAD`. Every review is
   `BASE..HEAD`.
8. **Checklist**: Recon, Gate, Build, Review, Fix, QC (strikes 0/2). Write
   BASE into the Review item and the build seat into the Build item. If the
   harness has no task list, keep the checklist in a scratch note.

### Phase 1 — Recon

1. **One scout** (`scout-prompt.md`, read-only) by default. Returns the
   landing zone with `file:line` anchors, patterns and constraints, 1-3
   approaches with a recommendation, a size call (S/M/L with a file and line
   estimate), and open questions only the user can answer.
2. **A second scout in parallel** for a genuinely separate surface: an
   external one (a library API, a protocol, a version's behavior; this one
   may read docs and search the web) or an unrelated part of the codebase.
3. Scouts never edit and never run builds or tests.
4. Scout returns BLOCKED or empty? Do not guess. Take what it found to the
   user at the gate and ask for the missing pointer.

### Phase 2 — Gate

1. **Report compactly**: findings, approaches with your recommendation and
   why, size call, what will change in 3-6 bullets, which seat builds, branch.
2. **Ask the user once.** Approaches are the options, recommended first.
   Batch recon's open questions into the same message.
3. **Size L or an unmade design decision**: offer "stop, this needs a spec or
   plan" as an option. The user choosing it ends the run successfully.
4. **Nothing is written before this clears.** If the user adjusts the
   approach, restate the adjusted version and build that.

### Phase 3 — Build

**You implement it yourself only when all hold:** settled approach with no
open questions; roughly 3 files and 150 lines or fewer; one module; no new
subsystem, dependency, schema, or public interface. Otherwise dispatch **one
implementer** (`implementer-prompt.md`). Record `seat: controller` or
`seat: implementer` in the checklist.

- **You play**: follow the patterns recon named, write tests that verify real
  behavior, run TEST scoped to what you touched, commit.
- **Implementer**: give it the brief, approved approach, landing zone,
  BRANCH, and TEST pasted in. It commits its own work.
- **Stop and hand off**: if you are playing and the work outgrows the limits
  above, stop, commit what is coherent, and dispatch an implementer with the
  prompt's "Work already on the branch" section filled. Flip the seat to
  `implementer`. Sunk effort is not a reason to keep going.

### Phase 4 — Review

Dispatch **two reviewers in parallel** (`reviewer-prompt.md`), scope
`BASE..HEAD`, each with a different lens:

- **Correctness**: defects, edge cases, broken contracts, security.
- **Intent**: unmet acceptance, scope drift, wrong problem solved, local
  convention mismatch.

Both receive the brief, acceptance, approved approach, and any implementer
concerns verbatim. Each returns PASS or FAIL with findings at `file:line` and
a severity. Merge the findings, dropping duplicates. Always run this phase.

### Phase 5 — Fix

1. No findings: skip.
2. **Who fixes**: you, when the seat is `controller` or when every finding is
   `minor`. A fixer subagent (`fixer-prompt.md`) for any `critical` or
   `important` finding against implementer-written code.
3. **One pass.** No re-review. Disputed or unfixed findings go into the QC
   brief verbatim.
4. Whoever fixes, commits.

### Phase 6 — QC

1. Dispatch QC (`qc-prompt.md`) with `BASE..HEAD`, BUILD, TEST, the brief
   with acceptance, and carried-forward findings. It runs the build and tests,
   verifies acceptance in the code, and scans for ship-blockers.
2. **SHIPPABLE**: report. Push only if asked. Never merge.
3. **NOT_SHIPPABLE**: route each blocker. `[defect]` to a fixer or to you by
   the Phase 5 rule. `[implementation]` back to an implementer, or to the user
   if the approved approach itself is wrong. Re-run QC with the retry section
   filled.
4. **Two strikes, then the user.** Update the strike count on each
   NOT_SHIPPABLE. On the second, stop and ask the user, offering to hand the
   work to `plan` and `execute` or, if what fails is a bug, to `fix`.

## Report

- What changed and why, in a few lines.
- Files and commit SHAs.
- Seats used: how many dispatches, and whether you played or dispatched.
- QC evidence: real build and test output lines.
- Left open: unfixed, disputed, or deliberately out of scope.
- Whether you pushed, and that merging is the user's call.

## Status handling

- **Scout**: FOUND → gate. BLOCKED → gate, with what it found.
- **Implementer**: DONE → review. DONE_WITH_CONCERNS → review, concerns
  pasted in. NEEDS_CONTEXT → supply it, re-dispatch. BLOCKED → supply context
  or return to the user; a wrong approach is a gate decision.
- **Fixer**: FIXED → QC. PARTIALLY_FIXED / COULD_NOT_FIX → QC, remainder
  carried in. Never re-loop Phase 5.
- **Reviewer**: PASS / FAIL. **QC**: SHIPPABLE / NOT_SHIPPABLE.

## Red flags

- Editing a file before the gate cleared.
- Reviewing your own work, or skipping review because it is "obviously right".
- Two writers on the branch at once.
- Playing on past the size limits instead of handing off.
- Guessing BUILD or TEST. Building on main. Looping QC past two strikes.
- Merging, or pushing without being asked.

## Bundled files

- `scout-prompt.md` — read-only recon: landing zone, approaches, size call.
- `implementer-prompt.md` — build the approved approach and commit.
- `reviewer-prompt.md` — one lens per dispatch over `BASE..HEAD`.
- `fixer-prompt.md` — repair listed findings and commit.
- `qc-prompt.md` — evidence-based SHIPPABLE / NOT_SHIPPABLE verdict.
