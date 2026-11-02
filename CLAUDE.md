# skills

Portable agent skills, lean harness-agnostic ports of the conductor skills in
[Repertoire](https://github.com/PythonShe/Repertoire). Install and skill list:
`README.md`.

## Layout

- `skills/<name>/SKILL.md`: the skill; frontmatter `name`, `description`.
- `skills/<name>/*.md`: optional short supporting prompts or templates.
- `skills/<name>-workspace/`: local eval scratch, gitignored.

## Authoring rules

- Harness-agnostic: no Claude Code tool names (Agent, AskUserQuestion, Skill),
  no model pins, no plugin paths. Write "ask the user", "spawn a subagent if
  the harness supports it, otherwise do it inline".
- Short imperative steps, one concern per phase, explicit stop points.
- Keep `SKILL.md` under ~200 lines; move detail to a sibling file only when the
  main flow cannot read without it.
- Description under 1024 characters: what it does first, then when to use it.
- Names are plain guessable verbs (`build`, `fix`, `plan`), never
  project-specific.
- Subagents are assumed cheap, but every dispatch and every harness-dependent
  step needs an inline fallback.
- Skills that persist state in the target project write under `docs/skills/`
  (e.g. `docs/skills/improve-backlog.md`), never at the top of `docs/`.
- Commit scopes: `feat(<skill>):` for a skill change, `docs:` for README and
  these files, `chore:` for tooling.
- License is Apache-2.0; add no other license headers.
