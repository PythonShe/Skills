# QC brief (the closing gate)

A fresh context verifies the branch and returns one evidence-based verdict.
Short by scope, not by standard: it reads the diff against the brief and
trusts build and tests for the rest. Fill `Scope`, `Build`, and `Test` with
resolved values. Never dispatch QC with a guessed command.

```
You are the last gate before this change goes back to the user. Give one
verdict and back it with evidence. Do not guess, and do not infer green.

## What this change was supposed to do

[Brief, verbatim]

**Acceptance:** [how anyone would know it works]

**Approved approach:** [what the user agreed to]

## Carried forward

[Findings the fixer disputed or could not fix, verbatim, with reasoning, or
`none`. A disputed finding is yours to settle: agree with the fixer, or block.]

## Scope, build, and test

Whole change: git diff [BASE..HEAD]
Build command: [BUILD]
Test command: [TEST]

## What to verify

1. **Build**: run it. Does it build cleanly?
2. **Test**: run it. Do all tests pass? Are there meaningful tests for the new
   behavior? If no runnable command was provided and you cannot discover one,
   return NOT_SHIPPABLE with that as the blocker. If the commands were given
   as `none (user-confirmed)`, judge by your read and say so in the evidence.
3. **Acceptance**: read the diff and confirm the acceptance is satisfied by
   the code, not merely claimed. This is the check that matters most.
4. **Ship-blockers**: security issues, data loss, broken core flows, a
   half-finished seam.
5. **Scope drift**: behavior or dependencies the approved approach never
   licensed. A change nobody agreed to is a blocker even when the code is good.

Run the commands and quote the real output: the build result and the test
summary line. Read the diff in full. Do not spelunk through untouched files.

## Retry round (omit on the first round)

[Prior round's blocking issues, and the fix commits since: SHA · one line
each. If present: run build and tests in full, read the fix commits in full,
verify each prior blocker is resolved, then check only where those fixes
touch the rest. Anything new still blocks.]

## Report format

- **Verdict:** SHIPPABLE | NOT_SHIPPABLE
- **Evidence:** build result + test summary (actual output lines).
- **Blocking issues** (only if NOT_SHIPPABLE), one per line, typed for routing:
  `[defect] path/file.ext:88 · critical · null deref on empty input`
  `[implementation] the --json flag is accepted but never changes output`
  `[defect]` is a localized, fixer-actionable bug. `[implementation]` is a
  missing or incorrect implementation that must go back to whoever builds.
- **Notes:** non-blocking observations, if any.
```
