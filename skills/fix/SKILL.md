---
name: fix
description: Hunts a bug to its root cause and lands a verified fix — two independent investigators trace the mechanism blind to each other, the controller cross-examines their reports as the confidence gate, then a fixer commits a deliberately failing repro test before the fix, a skeptical reviewer checks cause and only cause, and a mechanical verifier proves red to green by reverting the fix. Use when the user reports a reproducible bug, regression, crash, or failing test — "fix this bug", "why is this test failing", "find the root cause". Commits as it goes, never merges, pushes only on request. Code that works but should be better is not a bug; name the change and use build. Review feedback or failing CI on an open PR goes to address.
---

# fix

Something is broken. You dispatch, compare, and decide. You never read source
or form your own theory: a judge who investigates becomes a third, biased rival.

## Rules

- **You judge; you never play.** No reading source files or diffs, no
  hypotheses, no fixes. Allowed: assembling the bug brief, read-only git
  metadata (`log --oneline`, `status`, `rev-parse`), keeping the checklist.
  Without subagents you must take the dispatch roles yourself; keep them as
  separate passes and judge only from the written reports.
- **No fix without an evidence-backed root cause.** A traced mechanism: the
  defect, the chain from defect to symptom, and why fixing it kills the
  symptom. "Plausible" is not a root cause.
- **Investigators stay blind to each other.** Independent convergence is the
  confidence gate. The scout's ranking goes to investigator A only.
- **Evidence decides.** Quoted output or it did not happen.
- **Failing test first.** The repro test must fail for the diagnosed reason
  before the fix exists, and pass after. Committed separately, test first.
- **Strike budgets, then the user.** One rebuttal round for a deadlock, three
  strikes for the repair loop. Never loop silently past a budget.
- **Feature branch only.** Never merge. Push only when asked.

## Subagents

Subagents are cheap. Every investigator, fixer, reviewer, and verifier is a
fresh dispatch with a self-contained brief from the bundled prompt files. Run
independent dispatches in parallel. Without subagents, run each brief yourself
in sequence, in a fresh pass that starts from the brief only. Run investigator
B first (brief only), write its report down, then run A with the scout's
ranking. You cannot un-know B's findings, so say in the final report that the
investigations were sequential, not blind, and lean harder on the repair
gates.

## Pipeline

0. **Intake** — bug brief, branch, BUILD/TEST, checklist, cost gate.
1. **Investigation** — scout, then investigator A (primed) and B (blind) in
   parallel.
2. **Cross-examination** — mechanisms agree → repair. Disagree → one rebuttal.
   Still split → ask the user.
3. **Repair loop** — fixer commits red test, then green fix → reviewer.
4. **Verification** — red leg, green leg, suite, build. Three strikes → ask.

### Phase 0 — Intake

1. **Resuming?** Read the checklist and `git log --oneline`. Resume at the
   phase that stopped. Read the strike count from the Repair item.
2. **Bug brief**, the one document every dispatch receives:
   - **Symptom**: exact error text or failing test name, verbatim.
   - **Expected**: what should happen. Ask the user if not obvious.
   - **Repro**: command or steps. Unknown? Say so; finding one becomes the
     investigators' first deliverable.
   - **History**: when it last worked, suspected changes. Capture
     `git log --oneline -15`.
   - **Environment**: versions, platform, flags, anything unusual.
   Ask the user for anything missing rather than letting agents guess.
3. **BUILD and TEST** from the README or package manifest. Cannot find them?
   Ask. Project has none? Record `none (user-confirmed)` and pass that token
   to the fixer and verifier.
4. **Clean tree.** Uncommitted changes are the user's work, and this skill's
   investigators and verifier reset the working tree. Dirty `git status` →
   stop and ask before anything else.
5. **Branch**: confirm you are on a feature branch, not main or master. If
   not, create `fix/<slug>` or get consent.
6. **Trivial?** If the user's own report already names an obvious one-line
   fix, say so and confirm with the user before running the full pipeline.
7. **Cost gate**: if the user did not name this skill, confirm in one message:
   the brief and that a two-investigator hunt is about to run.
8. **Checklist**: Scout, Investigator A, Investigator B, Cross-examination,
   Repair (strikes 0/3), Verify, Report. If the harness has no task list,
   keep it in a scratch note.

### Phase 1 — Investigation

1. **Scout** (`scout-prompt.md`): fast triage, not a deep trace. Returns
   ranked candidate fault surfaces, most to least likely, one line each.
   Dispatch investigator B at the same time; it does not wait on the scout.
2. **Investigator B** (`investigator-prompt.md`, blind): the bug brief only.
   No scout ranking, no hints beyond the brief.
3. **Investigator A** (`investigator-prompt.md`, primed): the bug brief plus
   the scout's ranking as search priorities, never scope. Dispatch as soon as
   the scout returns.
4. Both return the same shape: root cause as a mechanism at `file:line`,
   evidence chain, repro, proposed minimal fix, confidence. Or NO_ROOT_CAUSE
   with what they ruled out.

### Phase 2 — Cross-examination

Enter only with both reports in hand.

1. **Compare from the text alone.** Agreement means the same mechanism: same
   causal story, overlapping locations, compatible fixes. Not similar prose.
   Match → gate passed. Carry the sharper repro and fix proposal forward.
2. **Both empty-handed** (NO_ROOT_CAUSE or only LOW confidence from both):
   do not force a verdict. Ask the user, presenting what each learned and what
   would unblock the hunt.
3. **One mechanism, one empty hand**: dispatch a fresh skeptic
   (`investigator-prompt.md` in rebuttal mode, lone-mechanism audit).
   Confirmed → passed. Refuted → treat as empty-handed. This uses the rebuttal
   round.
4. **Two mechanisms in conflict**: one rebuttal round, a fresh skeptic with
   both reports pasted in, told to test the claims against the code.
5. **Decide from the round.** SAME_MECHANISM, A_CORRECT, or B_CORRECT →
   converged, proceed. BOTH_WRONG → ask the user, offering the skeptic's new
   hypothesis if it produced one.

### Phase 3 — Repair loop

1. **Fixer** (`fixer-prompt.md`): receives the agreed root cause, evidence,
   fix proposal, and repro. Writes the minimal repro test, proves it red,
   commits the test. Applies the minimal fix, proves repro and suite green,
   commits the fix separately.
   - FIXED → reviewer.
   - COULD_NOT_FIX → a strike. Re-dispatch with more context, or escalate on
     the third.
   - CAUSE_DISPUTED (test would not go red, or code contradicts the
     diagnosis): a strike, and it voids the cross-exam verdict. Take the
     fixer's evidence back to Phase 2. Use the rebuttal round if unused,
     otherwise ask the user.
2. **Reviewer** (`reviewer-prompt.md`): a skeptic on the fix diff. Three
   questions, all must pass: kills the cause, kills only the cause, the test
   pins the bug.
3. FAIL → a strike. Hand findings to a fresh fixer pass and re-review.
   PASS → Phase 4.
4. **Strike budget.** One shared counter. A strike is COULD_NOT_FIX,
   CAUSE_DISPUTED, reviewer FAIL, or verifier NOT_VERIFIED. Update the
   checklist the moment one accrues. On the third, stop and ask the user:
   three failed repairs usually mean the root cause is wrong or the problem is
   architectural.

### Phase 4 — Verification

1. **Verifier** (`verifier-prompt.md`): fresh eyes, mechanical evidence, four
   legs. Red: with every fix commit reverted in the working tree, the repro
   test fails for the diagnosed reason. Green: at HEAD it passes. Suite: full
   TEST green. Build: BUILD clean. Hand it TEST_SHA, every fix commit SHA
   across re-fix passes, the fixer's recorded red proof, and the exact repro
   test command.
2. NOT_VERIFIED → a strike. Route evidence back to the fixer, or to Phase 2
   if it contradicts the root cause.
3. VERIFIED → report.

## Report

- **Root cause**: the mechanism, one short paragraph.
- **Evidence**: the chain that proved it.
- **The fix**: files touched, test commit and fix commit SHAs.
- **Verification**: four legs with quoted output.
- **Concordance**: converged on the first pass, after rebuttal, or
  user-decided.

Commits stay on the branch. Never merge. Remind the user the history contains
a deliberately red test commit, so a squash-merge keeps CI and bisect clean.

## Red flags

- Reading source or proposing your own root cause.
- Handing the scout's ranking to investigator B, or letting A treat it as
  scope.
- Calling similar prose "agreement".
- Cross-examining with one report in hand.
- "Quick fix for now, root cause later."
- A fix without a red repro test first, or a test written after the fix to
  match it.
- A fixer that special-cases the symptom away.
- Skipping the reviewer because the fix is "obviously right", or the verifier
  because the reviewer passed. One reads, the other runs.
- A fourth swing after three strikes. Merging the branch.

## Bundled files

- `scout-prompt.md` — fast triage, ranked candidate fault surfaces.
- `investigator-prompt.md` — systematic root-cause hunt; rebuttal mode for
  deadlocks.
- `fixer-prompt.md` — red to green, two-commit repair.
- `reviewer-prompt.md` — cause, only cause, test pins the bug.
- `verifier-prompt.md` — mechanical four-leg proof with a revert-based red leg.
