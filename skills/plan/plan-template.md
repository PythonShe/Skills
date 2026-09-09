# Plan template

Decision-complete, not code-complete: every section makes a decision an
implementer would otherwise have to guess, and stops short of function bodies.
Scale each part to the work and drop what does not apply. A small plan is
short, not padded.

```markdown
# <Title> — Implementation Plan

> **For the executor:** this plan is structured into groups, each sized for
> one fresh implementer, with clean seams between them. Execute with `execute`
> or any plan executor. Tasks use checkbox syntax for tracking.

**Spec:** `docs/skills/specs/<file>.md` (or: derived from conversation, see Goal)
**Goal:** one sentence, what this builds.
**Architecture:** 2-3 sentences on the approach, lifted from the spec.
**Build:** `<exact command>` · **Test:** `<exact command>`

## File structure

Every file the plan touches, one line each on its single responsibility:

- Create `exact/path/one.py` — role.
- Modify `exact/path/two.py` — what changes and why.

## Group 1: <name>

What this group delivers and why these tasks travel together.
**Depends on:** nothing | Group N.

- [ ] **Task 1.1 — <component>**
  - **Files:** Create `exact/path.py` · Test `tests/exact/test_path.py`
  - **Behavior:** what it does, in a few sentences. Every public interface,
    signature, and data shape spelled out exactly as later tasks will
    reference it, e.g. `parse(raw: str) -> Receipt`, where `Receipt` has
    `total: Decimal`, `vendor: str | None`.
  - **Error handling:** each failure mode that matters, and what handles it.
  - **Tests:** each test by name with the behavior it pins down and its key
    assertion, e.g. `test_expired_token_rejected`: POST with expired JWT →
    401, body `{"error": "token_expired"}`.
  - **Verify:** `exact command` → expected outcome.

- [ ] **Task 1.2 — <component>**
  - Same shape. Repeat any interface from 1.1 that this task consumes;
    implementers read tasks in isolation.

## Group 2: <name>

...

## Notes for the executor

Only if needed: risks, sequencing warnings, assumptions carried from the
spec's open questions. Each line should change how someone executes.
```

Conventions:

- **Number tasks `<group>.<task>`** so reviews and fixes can point at them.
- **Repeat interfaces at the point of use.** Cross-references die when tasks
  are pasted into separate implementer contexts.
- **The header's Build and Test commands are load-bearing.** `execute` reads them
  from the plan before falling back to the README or manifest.
- **An empty "Notes for the executor" is deleted**, not left as a heading.
