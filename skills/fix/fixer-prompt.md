# Fixer brief (red to green, two commits)

Dispatched once the root cause is settled. The red repro test is committed
before the fix exists, so the verifier can prove red to green from history.

For a re-fix pass after a reviewer FAIL or verifier NOT_VERIFIED, add a
"Findings to address" section with the gate's findings pasted in. The red test
usually already exists; extend it only if the findings show it pins the bug
too loosely.

```
You are fixing a diagnosed bug. The investigation is done and was
cross-examined between two independent investigators. Your job is the repair,
test-first.

## The agreed root cause
[Paste: mechanism with file:line, evidence chain, proposed minimal fix]

## Bug brief
[Paste: symptom, expected, repro]

Work from: [directory]   Branch: [branch]
Build: [BUILD]   Test: [TEST]
Confirm you are on the named branch before committing.

## Discipline, in this exact order
1. **Repro test first.** Write the minimal test that captures the bug. Assert
   the expected behavior, not the absence of a crash. Run it: it must FAIL,
   for the diagnosed reason. Quote the failing output.
   - If it will not go red, STOP. Fix nothing. Report CAUSE_DISPUTED with what
     you observed. A passing repro means the diagnosis is suspect.
2. **Commit the test** alone: `test: red repro for [bug]`. This commit is
   deliberately red.
3. **Minimal fix.** Change exactly what the mechanism demands. No refactors,
   no cleanups, no defensive special-cases that hide the symptom. If the fix
   keeps growing beyond the mechanism, stop and report.
4. **Prove green.** The repro test passes; the full test suite passes. Quote
   both results. If TEST is `none (user-confirmed)`, the repro test must
   still be runnable on its own; say how.
5. **Commit the fix** separately: `fix: [root cause one-liner]`.

## Report format (compact)

- **Status:** FIXED | COULD_NOT_FIX | CAUSE_DISPUTED
- **Commits:** TEST_SHA, FIX_SHA. On a re-fix pass, every new fix commit SHA.
- **Repro test command:** the exact single-test command, verbatim.
- **Red proof:** the failing line before the fix. **Green proof:** repro and
  suite result lines after.
- What you changed, one line per file.
- Anything that surprised you; that goes to the reviewer.
```
