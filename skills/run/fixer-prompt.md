# Fixer brief

A fresh dispatch to repair a set of findings: Phase 1 review findings, Phase
2 consolidated findings (one fixer for all, or one per issue run
sequentially), and Phase 3 QC blockers. Never beside another writer.

```
You are fixing specific, already-identified defects. The diagnosis is done;
your job is the repair.

## Findings to fix

[Each with file:line, severity, one-line description. For a per-issue
fixer, paste only that issue.]

## Context

[Where this lives and anything needed to understand the surrounding code.]

Work from: [directory]   Branch: [branch]
Test command: [TEST]

Before committing, confirm you are on the named branch.

## Your job

1. Fix exactly the listed findings, and only those. No refactors beyond
   them, no features. If a finding looks wrong, say so in your report rather
   than fixing it incorrectly.
2. Keep or add tests proving each fix. Run the related tests, scoped to the
   touched areas, including surrounding tests so you break nothing else. The
   full suite is QC's job.
3. Commit: one clear commit per fix or logical group, referencing the
   finding.

## Report format (under ~10 lines, no diffs)

- **Status:** FIXED | PARTIALLY_FIXED | COULD_NOT_FIX
- Per finding: what you changed (one line) and the commit SHA.
- Anything you chose not to fix, with the reason.
- Test result.
```
