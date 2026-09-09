# skills

Portable, harness-agnostic agent skills for fast, low-overhead workflows that
run on any model.

These are lean adaptations of the conductor-style skills in
[Repertoire](https://github.com/PythonShe/Repertoire), with the Claude Code
specific machinery removed so they run wherever a `SKILL.md` is read: Claude
Code, Codex, Cursor, Gemini CLI, OpenCode, and others.

## Install

```
npx skills add PythonShe/Skills
```

Install a single skill:

```
npx skills add PythonShe/Skills --skill <name>
```

See [vercel-labs/skills](https://github.com/vercel-labs/skills) for supported
agents and options.

## Skills

| Skill | Use it when | Adapted from |
|---|---|---|
| `build` | One concrete change to build now, no spec or plan first | Repertoire `presto` |
| `fix` | A reproducible bug, regression, or failing test to diagnose and fix | Repertoire `tuner` |
| `improve` | "Make this repo better, you pick": an unattended session that finds and ships 3-5 small wins | Repertoire `jam` |
| `spec` | An idea or feature that needs designing into a build-ready spec | Repertoire `libretto` |
| `plan` | A settled spec or design that needs a decision-complete implementation plan | Repertoire `score` |
| `execute` | A written plan (or a spec) to build group by group with review and a QC gate | Repertoire `maestro` |

Typical flows: `build` for one change, `fix` for a bug, `spec` then `plan`
then `execute` for anything bigger (execute also accepts a spec directly for
work of a few groups), `improve` when you have no target in mind.

Each skill assumes subagents are cheap and fans out freely, and each dispatch
has an inline fallback for agents without subagents. Skills that keep state in
your project write it under `docs/skills/`.

## Layout

```
skills/
  <name>/
    SKILL.md        # the skill
    *.md            # optional short supporting prompts
```

## License

Apache-2.0
