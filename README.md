# skills

Portable, harness-agnostic agent skills for fast, low-overhead workflows that
run on any model.

These are lean adaptations of the conductor-style skills in
[Repertoire](https://github.com/PythonShe/Repertoire), with the Claude Code
specific machinery removed so they run wherever a `SKILL.md` is read: Claude
Code, Codex, Cursor, Gemini CLI, OpenCode, and others.

## Install

```
npx skills add PythonShe/skills
```

Install a single skill:

```
npx skills add PythonShe/skills --skill <name>
```

See [vercel-labs/skills](https://github.com/vercel-labs/skills) for supported
agents and options.

## Skills

| Skill | Use it when | Adapted from |
|---|---|---|
| `build` | One concrete change to build now, no spec or plan first | Repertoire `presto` |

Planned: `fix` (from `tuner`), `improve` (from `jam`).

## Layout

```
skills/
  <name>/
    SKILL.md        # the skill
    *.md            # optional short supporting prompts
```

## License

Apache-2.0
