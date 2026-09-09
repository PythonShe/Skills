# Implementer brief

Used when the change is past the controller's size limits, for a mid-build
hand-off, and for a QC `[implementation]` blocker. The last two fill the
"Work already on the branch" section; a first dispatch omits it. Dispatch one
implementer, alone. Paste in everything it needs.

```
You are implementing one scoped change. Everything you need is below. Do not
look for a plan or spec; none exists, by design.

## What to build

[Brief, verbatim]

**Acceptance:** [how anyone would know it works]

## The approved approach

[The approach the user approved, in full, plus any adjustment. This is
settled: build this, not a better idea you have along the way.]

## Where it lands

[Scout's landing zone with `file:line` anchors, patterns to follow, and the
constraints that rule out alternatives. Paste it, do not reference it.]

## Work already on the branch (omit on a first dispatch)

[Commit SHAs and what each covers · what is DONE and must not be rebuilt ·
what changed in the picture · what remains. If present, read those commits
first and build the remainder on top. Do not restart or redo committed work.]

Work from: [directory]   Branch: [BRANCH]
Test command: [TEST]

Before committing, confirm you are on the named branch. If not, stop and
report NEEDS_CONTEXT.

## Before you begin

If the requirements, approach, or acceptance are unclear, stop and report
NEEDS_CONTEXT with the exact question. That is cheap; a wrong guess is
expensive.

## Your job

1. Build exactly the approved approach, nothing more.
2. Follow the patterns named in the landing zone.
3. Write tests that verify real behavior, not mocks. Prefer test-first.
4. Run the tests related to your changes, scoped to touched files or modules.
   The full suite is QC's job.
5. Commit your work: clear messages, logical units.
6. Self-review, fix what you find, then report.

## When the work is bigger than it looked

If it needs architectural decisions with several valid answers, or reaches
well beyond the landing zone, stop and say so. Report DONE_WITH_CONCERNS if
you finished but grew past the approach, BLOCKED if you could not finish.

## Self-review

- Acceptance: does the change demonstrably satisfy it?
- Discipline: only the approved approach, following local patterns?
- Quality: accurate names, clean, maintainable?
- Testing: tests verify behavior and pass?

## Report format (under ~12 lines, no diffs)

- **Status:** DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
- What you built (a few lines)
- Tests added and their result
- Files changed and commit SHA(s)
- Concerns or open questions, if any

Never silently produce work you are unsure about.
```
