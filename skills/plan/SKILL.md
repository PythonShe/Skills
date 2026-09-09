---
name: plan
description: Turns a spec or agreed design into a decision-complete implementation plan — a read-only scout maps the codebase, the user approves the file structure and task grouping at one checkpoint, the controller writes the plan itself with exact paths, interfaces, named tests, and verify commands but no function bodies, then three parallel skeptical reviewers harden it before a final user-approval gate. The plan is the terminal artifact, never code. Use when a design is settled and the user wants the implementation plan — "write the implementation plan", "plan this out", "break the spec into tasks". Executing a finished plan goes to execute; one small change with no plan needed goes to build.
---

# plan

Turn an approved spec into a plan complete enough that a fresh implementer
with zero conversation context can build each part without asking what you
meant. You write the plan yourself. You delegate exactly two things: gathering
context, and the adversarial review of the finished plan.

## Rules

- **Decision-complete, not code-complete.** Exact file paths, interfaces,
  signatures, data shapes, named tests with key assertions, exact verify
  commands. No function bodies. A plan full of code is the implementation
  done in the wrong phase with no review behind it.
- **You plan; you delegate scrutiny.** Dispatch only the context scout and
  the reviewers. Editing the plan in response to review is your job.
- **Groups are the unit.** Tasks are grouped the way `execute` will build
  them: coupled work travels together, one implementer-sized unit per group,
  clean seams between groups.
- **The plan is the terminal artifact.** Never implement. Never redesign:
  open design questions go back to the user, not into the plan as guesses.
- **One user checkpoint before writing, one approval after.** Never skip the
  structure checkpoint because the decomposition seems obvious.

## Subagents

Subagents are cheap. The scout and the three reviewers are fresh dispatches
from the bundled prompts. Reviewers run in parallel. Without subagents, run
each brief yourself in a fresh pass that starts from the brief and the plan
file only.

## Pipeline

0. **Locate the spec** and scope-check.
1. **Context scout** → brief.
2. **Structure checkpoint** → user approves file map and groups.
3. **Write the plan**, self-check, commit.
4. **Review panel** (3 lenses, parallel) → revise, commit.
5. **User approval** (terminal) → point to `execute`, never invoke it.

The committed plan is the durable trail.

### Phase 0 — Locate the spec

0. **Resuming?** Read the checklist and `git log --oneline`. If a draft plan
   is committed, re-read it and resume at the first phase not done.
1. **Find and read the spec in full.** Use the path the user gave, or look in
   `docs/skills/specs/` (where `spec` writes) and any obvious design doc.
   Confirm if ambiguous. Reading the spec is your job, not a scout's.
2. **No written spec?** Run a short clarify pass: what is being built, the
   constraints, the success criteria, in a handful of questions. This is not
   a design dialogue. If real design decisions are open (approach unchosen,
   interfaces unsettled), stop and point to `spec`.
3. **Scope check.** If a single implementer could just do it, say so and
   point to `build`. If the goal is really several unrelated goals, help the
   user pick the first; plan one at a time. If it is one goal too big for
   one `execute` session, split it at its cleanest seam into a sequence of
   plans: plan the first now, and list the rest with one-line deliverables
   under Notes for the executor.
4. **Clean tree, then branch.** Plans are committed. Uncommitted changes
   are the user's work: ask before touching anything. If you are on main or
   master, create `plan/<topic>` or get the user's consent to commit there.
   Every commit in this skill stages only the plan file.
5. **Checklist**: one item per phase. If the harness has no task list, keep
   it in a scratch note.

### Phase 1 — Context scout

Dispatch one read-only scout (`scout-prompt.md`) to map the ground: likely
touch points, conventions, testing patterns, exact build and test commands,
and a check of the spec's claims about the codebase, flagging drift. Read the
brief, not the codebase. Dispatch once. Skip only for a greenfield project.

Spec drift (the spec names files or interfaces that do not exist as described)
goes to the user at the structure checkpoint. It may need a spec fix, not a
plan workaround.

### Phase 2 — Structure checkpoint

1. **Map the file structure.** Every file the plan creates or modifies, one
   clear responsibility each. Follow the codebase's patterns.
2. **Group tasks.** Tasks travel together when they touch the same files or
   module, when one depends on another's output, or when splitting them would
   force a second implementer to re-learn the first's context. Keep groups
   independent where you can. A group must be small enough for one agent to
   hold in context and large enough to stand on its own.
3. **Present compactly**: the file map, the groups, one-line intent each.
   Ask the user to approve before writing. A wrong decomposition caught here
   costs a paragraph; caught after review, it costs the whole plan.

### Phase 3 — Write the plan

Write to `docs/skills/plans/YYYY-MM-DD-<topic>.md`, following
`plan-template.md`. Create the directory if missing. User preference for
location overrides this.

**Never write:**
- "TBD", "TODO", "decide later", "fill in details".
- "Add appropriate error handling": name each failure mode and its handler.
- "Write tests for the above": name each test and what it pins down.
- "Similar to Task N": repeat the interface. Groups are read in isolation.
- A reference to a type, function, or signature defined nowhere in the plan
  or the existing codebase.
- Full code blocks for routine work.

**Self-check, then commit.** Re-read the spec section by section: can you
point at the task that implements each requirement? Scan for the failure
patterns above. Check that names used in later groups match where they were
defined. Fix inline, then commit with `docs(plan): ...`.

### Phase 4 — Review panel

Dispatch **three reviewers in parallel** (`reviewer-prompt.md`), one lens
each, with the plan path and the spec path (or the clarified requirements
pasted in, and say so):

- **Spec coverage and traceability**: requirements with no task, tasks with
  no spec backing, contradictions of decisions the spec made.
- **Buildability and self-containment**: could a fresh implementer execute
  each task alone? Undefined interfaces, placeholders, vague tests, missing
  verify commands.
- **Sequencing and decomposition**: tasks referencing later work, seams that
  split coupled work or weld independent work, groups too big for one agent.

Swap a lens when the work demands it (a migration-heavy plan might want
rollback and ordering). Consolidate findings from the text, drop duplicates
and anything not tied to a specific group or task. Revise the plan yourself.
Commit. Re-run the panel only if revisions restructured groups or changed
interfaces, never as a counted loop.

### Phase 5 — User approval

Ask the user to read the committed plan and say whether they want changes.

- **Changes**: structural → back to Phase 2; small → edit, commit, re-review
  only if warranted.
- **Approved**: done. Say the natural next step is `execute`. Do not invoke it,
  start implementing, or touch any source file.

## Status handling

- **Scout**: fold touch points, commands, and drift into the structure
  proposal. Found nothing useful? Proceed; do not re-dispatch to fish.
- **Reviewers**: PASS / FAIL with findings tied to a group and task. A
  finding you cannot tie to one is not actionable; drop it.

## Red flags

- Choosing an approach or settling an interface inside the plan. That is
  design work; stop and say so.
- Full code blocks for routine work.
- Any placeholder pattern from the Phase 3 list.
- Skipping the structure checkpoint.
- Reading the whole codebase yourself instead of dispatching the scout.
- Writing code, scaffolding, or invoking `execute` after approval.

## Bundled files

- `plan-template.md` — the plan's structure: header, file map, groups, tasks.
- `scout-prompt.md` — read-only context scout: touch points, conventions,
  build and test commands, spec drift.
- `reviewer-prompt.md` — skeptical plan reviewer, one lens per dispatch.
