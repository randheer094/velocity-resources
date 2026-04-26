# velocity-resources

Starter Claude Code configurations for new projects. Drop the
`.claude/` directory from a template into your repo root and Claude
Code will pick up the conventions, skills, and pre-PR gates for that
stack.

## Templates

- [`android/`](./android/.claude) — Jetpack Compose app template.
  MVI + Hilt architecture, Gradle Kotlin DSL with version catalog
  and convention plugins, unit + instrumented test gates.
- [`go/`](./go/.claude) — Go module template. Standard `cmd/` +
  `internal/` layout, `log/slog` logging, `-race` and ≥ 90%
  coverage gates, security rules for input validation and SQL.

Each template ships with:

- `CLAUDE.md` — entry point Claude reads first; lists build/test
  commands and links to the rest.
- `rules/conventions.md` — architecture, testing, style, and
  layout rules to follow on every change.
- `skills/prepare-for-pr/SKILL.md` — the gate sequence to run
  before opening a PR.

## Usage

Copy the `.claude/` directory from the template you want into the
root of your project:

```bash
cp -r android/.claude /path/to/your/android-project/
# or
cp -r go/.claude /path/to/your/go-project/
```

Then edit `CLAUDE.md` to replace the placeholder project
description, and tune the conventions and pre-PR skill to match
your codebase.
