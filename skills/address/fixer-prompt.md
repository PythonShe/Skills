# Fixer brief

One fresh dispatch per verified directive, or one for the whole queue when
items are few or interrelated. Fixers write to the branch, so they run
sequentially. The verifier's directive is authoritative even where it differs
from what the reviewer literally asked (PARTIAL verdicts).

For a re-fix after a panel or QC finding, paste the finding in place of the
directive and name the F-id it relates to.

```
You are implementing an already-verified fix on a pull-request branch. A
read-only verifier confirmed the problem and wrote the directive below. The
directive is authoritative; where it differs from the reviewer's suggestion,
follow the directive.

## Fix directive

[Paste: what to change, where, the acceptance check. Several items: each
with its F-id.]

## Original feedback (context)

[The ledger entry or entries: F-id, author, location, demand.]

Work from: [directory]   Branch: [branch]
Test: [TEST]
Confirm you are on the named branch before committing.

## Your job

1. Implement exactly the directive. No refactors beyond it, no bonus
   features. If the directive proves wrong once you are in the code, report
   that instead of improvising a different fix.
2. Keep or add the test the acceptance check names. Run the tests related to
   your change (scope [TEST] to the touched files when the runner allows),
   including surrounding tests in those areas. The full suite is QC's job.
3. **Commit**: one clear commit per item in the repo's convention,
   referencing the item (e.g. `fix: guard empty session (PR #42 review, F3)`).

## Report format (under ~10 lines)

- **Status:** FIXED | PARTIALLY_FIXED | COULD_NOT_FIX
- Per item: commit SHA + one line on what changed, phrased for the thread
  reply ("Fixed in <sha> — …").
- Test result (summary line).
- Anything not fixed, and why.
```
