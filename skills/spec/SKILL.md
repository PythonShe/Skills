---
name: spec
description: Turns a rough idea into a build-ready spec through a guided design dialogue — a read-only scout maps the codebase, the controller clarifies intent with focused questions, proposes 2-3 approaches with a recommendation, presents the design section by section for approval, and writes the spec itself — then three parallel skeptical reviewers harden it before a final user-approval gate. The spec is the terminal artifact, never a plan and never code. Use when the user holds an idea or feature and wants it designed — "spec this out", "design this feature", "turn this idea into a spec". Turning an approved spec into an implementation plan goes to plan; a change too small to need design goes to build.
---

# spec

Turn a rough idea into a spec solid enough to build from, reached through
conversation, not guesswork, and proven sound by skeptics before anyone
writes code. You are hands-on with the design: you ask the questions, weigh
the approaches, and write the spec yourself. You delegate exactly two things:
gathering context, and the adversarial review of the finished spec.

## Rules

- **You design; you delegate scrutiny.** Dispatch only the context scout and
  the reviewers. Editing the spec in response to review is your job.
- **Conversation before commitment.** Pin down purpose, constraints, and
  success criteria through questions before proposing a design.
- **The author cannot review the author.** Fresh skeptics with distinct
  lenses review the spec, told to assume it is flawed until proven sound.
- **The spec is the terminal artifact.** Never write the plan, never build.
  Point to `plan` and then `execute` as next steps; never invoke them.
- **Too big for one spec?** Help the user split it into independent pieces
  and spec the first only.

## Subagents

Subagents are cheap. The scout and the three reviewers are fresh dispatches
from the bundled prompts; reviewers run in parallel. Without subagents, run
each brief yourself in a fresh pass from the brief and the spec file only.

## Pipeline

0. **Understand and scope** → context scout → brief.
1. **Clarify intent** through focused questions.
2. **Approaches and design**: 2-3 approaches, then the design section by
   section, each approved.
3. **Write the spec**, commit.
4. **Review panel** (3 lenses, parallel) → revise, commit.
5. **User approval** (terminal) → point to `plan`, never invoke it.

Keep a checklist with one item per phase; if the harness has no task list,
keep it in a scratch note. The committed spec is the durable trail.

### Phase 0 — Understand and scope

0. **Resuming?** Read the checklist and `git log --oneline`. If a draft spec
   is committed, re-read it and resume at the first phase not done.
1. Restate the idea in one sentence to confirm you have it right.
2. **Decomposition gate.** If the request describes several independent
   subsystems, stop and flag it before spending questions on details. Split
   with the user: the pieces, how they relate, the build order. Spec the
   first only.
3. **Too small?** If there is genuinely nothing to decide, say so and point
   to `build`. If an approved spec already exists, this skill is done.
4. **Context scout** (`scout-prompt.md`, read-only): dispatch once to map
   the relevant code, docs, and conventions into a compact brief. Read the
   brief, not the codebase. Skip only for a greenfield idea.
5. **Clean tree, then branch.** Specs are committed. Uncommitted changes
   are the user's work: ask before touching anything. If you are on main or
   master, create `spec/<topic>` or get the user's consent to commit there.
   Every commit in this skill stages only the spec file.

### Phase 1 — Clarify intent

Ask questions to pin down purpose, constraints, and success criteria. Offer
structured options when the choice is discrete (which boundary, which
trade-off); ask in prose when it is open-ended ("what does fast mean here?").
One decision per question. Stop asking when you can state the design without
guessing.

### Phase 2 — Approaches and design

1. **Propose 2-3 approaches** with trade-offs, leading with your
   recommendation and the reasoning.
2. Once one is chosen, **present the design in sections** scaled to their
   complexity: a sentence for the obvious, a paragraph for the nuanced.
   Cover architecture, components and interfaces, data flow, error handling,
   and testing. Get a quick approval after each section so a wrong turn is
   caught early. Design in small, well-bounded units: for each, say what it
   does, how it is used, and what it depends on.

### Phase 3 — Write the spec

Write to `docs/skills/specs/YYYY-MM-DD-<topic>.md`, following
`spec-template.md`. Create the directory if missing; user preference for
location overrides this. Scale or drop sections that do not apply; a small
spec is short, not padded. Record the assumptions you made during the
dialogue under Open questions and assumptions, so reviewers have something
concrete to challenge. Commit with `docs(spec): ...`.

### Phase 4 — Review panel

Dispatch **three reviewers in parallel** (`reviewer-prompt.md`), one lens
each, with the spec path and a one-paragraph brief of the idea, the agreed
approach, and any concerns carried from the dialogue:

- **Completeness and implementability**: TBDs, placeholders, missing
  sections, requirements too vague to build from, undefined interfaces.
- **Consistency and ambiguity**: internal contradictions, requirements that
  read two ways, architecture that does not match the feature descriptions.
- **Scope and YAGNI**: unrequested features, over-engineering, an empty or
  missing non-goals list, a spec that should have been decomposed.

Swap a lens when the work demands it (a data-heavy spec might want contracts
and migration). Consolidate findings from the text, drop duplicates and
anything not tied to a section. Three PASS verdicts → Phase 5 directly.
Otherwise revise the spec yourself and commit. Re-run the panel only if
revisions were large enough to introduce new risk, never as a counted loop.

### Phase 5 — User approval

Ask the user to read the committed spec and say whether they want changes.

- **Changes**: structural → back to Phase 2; small → edit, commit,
  re-review only if warranted.
- **Approved**: done. Say the natural next steps are `plan` and then
  `execute`. Do not write the plan, invoke either skill, or touch any
  source file.

## Status handling

- **Scout**: fold the relevant facts into your questions and design. Found
  nothing useful? Proceed; do not re-dispatch to fish.
- **Reviewers**: PASS / FAIL with findings tied to a section. A finding you
  cannot tie to one is not actionable; drop it.

## Red flags

- Writing the spec before the dialogue has pinned down purpose and
  constraints.
- Reviewing your own spec instead of dispatching the panel.
- Reading the whole codebase yourself instead of dispatching the scout.
- Writing an implementation plan, scaffolding, or any code.
- Invoking `plan` or `execute` after approval.

## Bundled files

- `spec-template.md` — the section structure for the spec.
- `scout-prompt.md` — read-only context scout: touch points, conventions,
  prior art, constraints, existing docs.
- `reviewer-prompt.md` — skeptical spec reviewer, one lens per dispatch.
