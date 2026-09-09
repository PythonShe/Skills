# skills

Portable agent skills for fast, low-overhead workflows on any model. Each skill
is a lean adaptation of a conductor-style skill from
[Repertoire](https://github.com/PythonShe/Repertoire), stripped of Claude Code
specifics so it runs on any harness that reads `SKILL.md`.

## Layout

- `skills/<name>/SKILL.md` — the skill. Frontmatter: `name`, `description`.
- `skills/<name>/*.md` — optional supporting prompts or references, kept short.
- `skills/<name>-workspace/` — local eval scratch, gitignored.

## Authoring rules

- Harness-agnostic: no Claude Code tool names (Agent, AskUserQuestion, Skill),
  no model pins, no plugin paths. Say "ask the user", "spawn a subagent if the
  harness supports it, otherwise do it inline".
- Built for speed and clarity: short imperative steps, one concern per phase,
  explicit stop points, no filler prose.
- Keep `SKILL.md` under ~200 lines. Move detail into a sibling file only when
  the main flow cannot read without it.
- Description under 1024 characters; lead with what it does, then when to use it.
- Names are plain verbs a user can guess without a lookup: `build`, `fix`,
  `improve`. No project-specific names.
- Subagents are assumed cheap: fan out freely, but every dispatch needs an
  inline fallback for harnesses without them.
- Skills that persist state in the target project write it under
  `docs/skills/` (for example `docs/skills/improve-backlog.md`), never at the
  top of `docs/` and never in a folder named after this repo's source.
- Every skill must degrade gracefully: if a step needs a capability the harness
  lacks, the skill says what to do instead.

## Commits

Conventional Commits: `feat(<skill>):` for a new or changed skill, `docs:`
for README and these files, `chore:` for tooling.

## Install

```
npx skills add PythonShe/Skills
```

## License

Apache-2.0. Do not add other license headers.
