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
  commands and indexes the rule files. Pure technical guide; no
  project-specific content.
- `rules/*.md` — per-topic rule files (architecture, data,
  testing, security, build, …). Claude reads only the ones
  relevant to the current change instead of a single monolith.
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

Project-specific context (what this app does, owners, key URLs,
runbooks) lives in a `CLAUDE.md` at your repo root — the
template's `.claude/CLAUDE.md` is purely a technical index and
should stay that way. Tune `rules/*.md` and the pre-PR skill to
match your stack.
