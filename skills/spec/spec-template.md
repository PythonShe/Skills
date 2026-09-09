# Spec template

Scale each section to its complexity: a sentence where the answer is obvious,
a few paragraphs where it is not. Drop a section that does not apply. A small
spec is short, not padded. The goal is a document someone could build from
without asking what you meant.

```markdown
# <Title>

## Problem and context
What we are solving and why it matters now. The current situation and what
is wrong or missing.

## Goals and non-goals
- **Goals**: what success looks like, as concrete bullets.
- **Non-goals**: what we are deliberately not doing or deferring, each with
  a one-line reason. An empty non-goals list is a warning sign.

## Approach
The chosen design, and one or two sentences on why it beat the alternatives.
The reader should understand not just what, but why this and not that.

## Components and interfaces
Each unit, small and well-bounded. For each: what it does, how it is used
(its interface), and what it depends on.

## Data flow
How data moves through the system, the key pieces of state, and where they
live.

## Error handling and edge cases
The failure modes that matter and how each is handled. The edge cases worth
calling out before they surprise the implementer.

## Testing strategy
How we will know it works: what to test, at what level, and what "done"
proves.

## Open questions and assumptions
Assumptions made while designing, stated plainly so reviewers can challenge
them, and anything still unresolved.

```
