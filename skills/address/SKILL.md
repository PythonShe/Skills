---
name: address
description: Works an open GitHub pull request's review feedback to closure through the gh CLI — a clerk harvests every formal review, inline thread, conversation comment, and failing CI check into a ledger; one read-only verifier tests each item against the code and returns a verdict with a fix directive, a pushback draft, or a clarifying question; fixers change only what verification confirmed; a build-and-test QC gate (plus a three-lens panel when the fixes are big or risky) seals the branch; then every thread gets a drafted reply and pushing and posting wait behind one user approval. Use when the user wants PR feedback handled — "address the review comments", "work through the PR feedback", "respond to my reviewers", "fix what CI flagged on the PR". Needs an open PR and an authenticated gh. Never merges, never resolves threads, never force-pushes. A bug with no PR attached goes to fix.
---

# address

The PR is open and the critics have answered. Review feedback is a pile of
claims, not a work order: some are right, some are wrong for this codebase,
some are too vague to act on. You dispatch the checks, route the verdicts,
and gate what goes outward. You never read code yourself.

## Rules

- **You conduct; you never play.** No reading source, diffs, or CI logs, no
  editing code. Allowed: the clerk's ledger, read-only PR metadata (`gh pr
  view`, `gh pr checks`), read-only git metadata (`merge-base`, `rev-parse`,
  `log --oneline`, `status`, `diff --stat`), `gh pr checkout` at setup, and
  after the publish gate clears, the single `git push` and the approved
  reply posts. Without subagents, take each
  role yourself as a separate pass that starts from its brief only.
- **Evidence outranks everyone.** An item earns a fix only when the verifier
  confirmed it in the code, and earns pushback only when the verifier refuted
  it with `file:line` evidence. Sycophancy (implementing a wrong suggestion)
  and defensiveness (rejecting a right one) are the two failures to prevent.
- **One verifier, whole ledger.** Related items get one coherent verdict, and
  its build and test runs never collide in the worktree.
- **Writers run alone; readers run together.** Fixers are sequential. Panel
  reviewers only read, so they run in parallel.
- **Everyone who changes code commits.** Each fixer commits its own work,
  referencing its F-id. Those SHAs feed the replies. You push once.
- **Outward actions pass one gate.** Push and replies go through one user
  approval after the final verdict. Never merge, resolve threads, approve or
  dismiss reviews, or force-push (rewriting the branch breaks the line
  anchors review comments hang on).

## Pipeline

0. **Setup** — cost gate, resolve PR, checkout, BASE + BUILD/TEST, clerk.
1. **Verify** — one read-only verifier judges the whole ledger.
2. **Fix** — sequential fixers: blocking, then simple, then complex.
3. **Final verdict** — panel only when fixes are big or risky, then QC.
   Three strikes → ask the user.
4. **Publish gate** — draft replies, one approval, push, then post.

### Phase 0 — Setup

1. **Resuming?** Read the checklist and `git log --oneline`. There is no
   state file: re-dispatch the clerk (answered and resolved threads drop out
   on their own), keep recorded verdicts, resume at the first phase not done.
2. **Resolve the PR.** Use the number or URL given, otherwise `gh pr view
   --json number,title,headRefName,baseRefName,isCrossRepository,url` on the
   current branch. No open PR → stop and say so. Note `isCrossRepository`: a
   fork PR may refuse the final push unless maintainer edits are allowed.
   Flag it at the publish gate, not after approval.
3. **Clean tree first.** Uncommitted changes are the user's work: stop and
   ask before anything else. Then `gh pr checkout <n>` if not already on the
   PR branch.
4. **BASE**: `git fetch origin <baseRefName>` then
   `git merge-base HEAD origin/<baseRefName>` (for a fork PR, the base repo's
   remote). If merge-base fails, ask the user for the base ref; never guess
   from a local branch, which may be stale and widen the scope. **BUILD and
   TEST** from the README or package manifest. Cannot find them? Ask. None?
   Record `none (user-confirmed)`.
5. **Cost gate**: if the user did not name this skill, confirm in one message
   the PR and the verify → fix → verdict pipeline about to run.
6. **Clerk** (`feedback-clerk-prompt.md`): fetches reviews, inline threads,
   conversation comments, failing checks; returns the **ledger**, one
   numbered item (F1, F2, …) with author, location, demand, reply routing.
   Raw JSON and CI logs stay in the clerk. `NO_FEEDBACK` → report and stop.
   `BLOCKED` → surface it; never harvest by hand.
7. **Checklist**: one item per F-id, plus Verify, Fix, Panel, Final verdict
   (strikes 0/3), Publish gate. No task list in the harness? Scratch note.

### Phase 1 — Verify every claim

1. **One verifier** (`verifier-prompt.md`), read-only, with the whole ledger,
   the PR's purpose, `BASE..HEAD`, BUILD, TEST. Only for 15+ items split
   into **sequential** batches, pasting the earlier batches' verdicts into
   each later brief so related items still get one verdict. Never parallel
   verifiers: their test runs collide.
2. Per F-id it returns one of:
   - **VALID** — confirmed, with a fix directive and acceptance check.
   - **PARTIAL** — the concern is real, the remedy wrong; corrected directive.
   - **INVALID** — refuted, with evidence and a draft pushback reply.
   - **UNCLEAR** — cannot be judged as written; a draft clarifying question.
3. Record each verdict on its checklist item. VALID and PARTIAL join the
   **fix queue**; INVALID and UNCLEAR drafts join the **reply pile**.
   `NEEDS_CONTEXT` → supply what is missing and re-dispatch, never unchanged.

A CI failure is just another claim. The verifier diagnoses the root cause;
"flaky" needs evidence, not hope.

### Phase 2 — Fix what survived

Empty fix queue → mark Fix, Panel, and Final verdict skipped (nothing on the
branch changed) and go to Phase 4 for replies only.

1. **Order**: blocking (broken behavior, security, failing CI), then simple
   (typos, naming), then complex (logic, refactors).
2. **Strategy**: one fixer for the queue when items are few or interrelated,
   otherwise one per item. Always sequential.
3. **FIXBASE** = current HEAD before the first fixer. Dispatch fixers
   (`fixer-prompt.md`) with the verifier's directive (authoritative over the
   reviewer's wording), the ledger entry, and TEST. Record each SHA.
4. PARTIALLY_FIXED / COULD_NOT_FIX → carry into the QC brief and draft an
   honest status reply. Never a silent drop.

### Phase 3 — Final verdict

1. **Panel, only when fixes are big or risky** (see below); say in the report
   whether it ran and why. Three reviewers (`reviewer-prompt.md`), distinct
   lenses, in parallel, scope `FIXBASE..HEAD` with `BASE..HEAD` as regression
   context, every brief listing the items the fixes claim to resolve.
   Consolidate findings from the text only: match by `file:line` plus
   description, drop duplicates and non-defects, route real defects to
   sequential fixers.
2. **QC gate** (`qc-prompt.md`) with BASE, FIXBASE, BUILD, TEST. It runs the
   build and tests and reads the fix range in full against every item.
   - **NOT_MERGEABLE** → route each blocker: `[defect]` → fixer,
     `[implementation]` → re-verify the item or escalate. Re-run QC in retry
     mode (prior blockers plus fix SHAs since). Each failed round is a
     strike; update the checklist. On the third, stop and ask the user.
   - **MERGEABLE** → Phase 4.

### Phase 4 — Publish gate

1. **Package**: one line per F-id, verdict → action (SHA, reply draft, or
   escalation), with the drafts attached:
   - Fixed → `Fixed in <sha> — <one line on what changed>.`
   - Pushback (INVALID) → the verifier's evidence-backed draft.
   - Clarification (UNCLEAR) → the verifier's question.
   - Unfixed → honest status and what blocks it.
   CI items have no thread: fold their outcomes into the conversation comment.
2. **Ask the user once**, presenting the package, with options: push and post
   all replies / push only, they reply themselves / revise the drafts first /
   stop, nothing outward. On revise, collect edits, apply, re-present. Flag a
   fork PR's push risk here.
3. **Publish, push first, replies second.** `git push` (never force). Confirm
   it succeeded before posting: a reply pointing at an unpushed commit points
   at nothing. Push fails → stop and report. Then inline replies in-thread:
   `gh api repos/{owner}/{repo}/pulls/<pr>/comments/<reply-to-id>/replies -f body=…`
   and one conversation comment for everything thread-less:
   `gh api repos/{owner}/{repo}/issues/<pr>/comments -f body=…`.
   gh expands `{owner}` and `{repo}` itself; substitute `<pr>` and
   `<reply-to-id>` (from the ledger) yourself.
4. **Report**: the per-item table, the clerk's skipped counts, whether the
   panel ran and why, QC's evidence, and what remains for the user: UNCLEAR threads awaiting
   reviewers, escalations, and the merge, which is never yours.

Re-runnable: after reviewers respond, the next run's clerk harvests only what
is new.

## When the panel runs

Most fix queues skip it: QC reads the full fix diff at the verdict. Escalate
to the three-lens panel when any of:

- 4+ items in the fix queue, or any PARTIALLY_FIXED / COULD_NOT_FIX,
- fixes touched security-sensitive code, a public interface or schema, or
  concurrency,
- the fix range spans more than ~5 files or ~150 lines
  (`git diff --stat FIXBASE..HEAD`).

Pick lenses for what the fixes did: behavior changes → correctness and edge
cases, regression risk, test coverage; security feedback → one security lens;
interface or schema fixes → contracts and compatibility. One lens each.

## Reply etiquette

- **No performative agreement, no thanks.** Never "You're absolutely right"
  or "Great catch". The fix is the acknowledgment; state what changed, where.
- **Pushback is technical reasoning.** Cite `file:line`, the test, the
  constraint; end with the question that would change the verdict, if real.
- **Proven wrong later?** The next run re-verifies; if the reviewer was
  right, fix it and state the correction factually.
- **Stay in the thread.** Inline feedback is answered in its thread.

## Red flags

- Reading diffs, source, or CI logs yourself.
- Fixing an item the verifier did not confirm.
- A verdict in either direction without `file:line` evidence.
- Parallel verifiers or parallel fixers.
- Pushing or posting before the gate clears, or posting before the push is
  confirmed.
- Praise or thanks in a reply draft.
- Calling a CI failure flaky without evidence.
- Skipping QC because the fixes were "trivial".
- Resolving threads, approving, force-pushing, or merging.

## Bundled files

- `feedback-clerk-prompt.md` — harvest the PR's feedback into the ledger.
- `verifier-prompt.md` — read-only verdict on the whole ledger.
- `fixer-prompt.md` — implement a verified directive and commit.
- `reviewer-prompt.md` — panel review, one lens per reviewer.
- `qc-prompt.md` — evidence-based mergeability verdict.
