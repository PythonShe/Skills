# Scout brief (read-only context)

Dispatch one read-only scout after reading the spec. The scout reports facts.
It does not decompose the work or propose tasks. Skip only for a greenfield
project with no existing code.

```
You are scouting an existing codebase so a planner can decompose an approved
spec into implementation tasks. Report what is there. Do NOT propose a task
breakdown, write code, or redesign anything.

## What is being planned

[One paragraph: what the spec commits to building, and the chosen approach.]

## What the spec claims about this codebase

[Bullet the spec's concrete claims: files it names, interfaces it assumes,
dependencies it relies on.]

## What to find

- The files and modules each part of this work would most likely touch.
- **Build and test commands**: exact invocations from the manifest, CI
  config, README, or Makefile, including how to run a single test.
- Testing conventions: framework, where tests live, how similar tests are
  structured and named.
- Existing patterns a new implementation should follow: naming, error
  handling, module layout, how similar features are wired in.
- Constraints the plan must respect: public interfaces, schemas,
  dependencies, config.
- **Spec drift**: check each claim above against the actual code. A file
  that does not exist, an interface with a different shape: flag it plainly.

Search broadly; read excerpts, not whole files. If something is absent, say
so rather than guessing.

## Report format (compact, no code dumps, no task proposals)

- **Likely touch points:** `path` — one line on its role.
- **Build/test commands:** exact invocations, incl. single-test form, or
  "none found, ask the user".
- **Testing conventions:** framework, location, one `path` example.
- **Patterns to follow:** bullets, each with a `path` example.
- **Constraints:** interfaces, schemas, deps the plan must respect.
- **Spec drift:** each mismatch as `spec says X · code has Y`, or "none".
- **Notes:** anything surprising.
```
