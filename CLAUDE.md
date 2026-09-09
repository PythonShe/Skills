# skills

Portable agent skills for fast workflows on lighter or weaker models. Each skill
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
- Built for lighter models: short imperative steps, one concern per phase,
  explicit stop points, no prose that a strong model would skim past anyway.
- Keep `SKILL.md` under ~150 lines. Move detail into a sibling file only when
  the main flow cannot read without it.
- Description under 1024 characters; lead with what it does, then when to use it.
- Every skill must degrade gracefully: if a step needs a capability the harness
  lacks, the skill says what to do instead.

## Install

```
npx skills add PythonShe/skills
```

## License

Apache-2.0. Do not add other license headers.
