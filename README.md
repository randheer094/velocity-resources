# velocity-resources

Two things live here:

1. **Claude Code starter templates** for new projects (`android/`,
   `go/`) — drop the `.claude/` directory into your repo root.
2. **Runtime prompts** consumed by the
   [velocity](https://github.com/randheer094/velocity) daemon
   (`prompts/`) — loaded from a release tarball at setup time so
   prompt edits ship without a velocity rebuild.

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

### Using a template

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

## Prompts

Externalized LLM prompts and Jira / GitHub failure-comment
templates for velocity. velocity loads `manifest.yaml` on startup,
loads each referenced template, and renders them with Go's
`text/template` at call sites.

### Why

- Prompts can be tuned without rebuilding velocity.
- Prompt history lives in git — diffs, blame, and PR review apply.
- Each velocity deployment pins to a release tag and updates on
  its own schedule via an explicit user-driven command.

### Layout

```
prompts/
  manifest.yaml                    # index — id → path + placeholder schema
  arch/plan.md                     # architect planning prompt
  code/run.md                      # code runner (fresh sub-task) prompt
  code/iterate.md                  # code iterate (existing PR) prompt
  failure/jira.md                  # Jira comment for arch/code Run failures
  failure/iterate_jira.md          # Jira comment for iterate failures
  failure/iterate_pr.md            # GitHub PR comment for iterate failures
```

### Template format

Templates use Go's [`text/template`](https://pkg.go.dev/text/template)
syntax. Placeholders are referenced as `{{.FieldName}}`. The exact
field set for each prompt is declared in `manifest.yaml` under
`placeholders`.

The renderer in velocity must:

1. Load `manifest.yaml` from the extracted release tarball.
2. For each entry, load the file at `path` (relative to
   `manifest.yaml`).
3. Parse with `text/template.New(id).Option("missingkey=error").Parse`.
4. Cache the parsed template by `id`.
5. On every render, pass a struct (or `map[string]any`) whose keys
   match the placeholders for that prompt.

`missingkey=error` is intentional — a typo in a placeholder name
should fail loudly during render, not silently produce `<no value>`
in a Jira comment.

## Releases

The `release` workflow (`.github/workflows/release.yml`) fires on
any `v*` tag push. It validates that the tag's major version
matches `prompts/manifest.yaml`'s `version:` field, then attaches
three artifacts to a GitHub Release:

- `velocity-resources-<tag>.tar.gz` — `go/`, `android/`, `prompts/`
- `velocity-resources-<tag>.zip` — same trees
- `SHA256SUMS` — checksums for both archives

Cutting a release:

```bash
git tag v1.0.0
git push origin v1.0.0
```

Tag major versus manifest `version:` is enforced — `v1.x.x`
requires `version: 1`, `v2.x.x` requires `version: 2`. Bumping
the manifest version (a breaking placeholder-schema change) means
the next tag must move to a new major.

### Velocity loading model

velocity is pinned to one release tag at a time and downloads the
artifact for that specific version. There is no fallback, no
bundled defaults, and no network call on the daemon hot path.

- **Setup (one-time):** the user supplies a version (e.g.
  `v1.0.0`) during `velocity setup`. The daemon downloads
  `velocity-resources-<tag>.tar.gz` from the matching GitHub
  Release, verifies its checksum against `SHA256SUMS`, and
  extracts it under the velocity config dir. Setup fails — and
  refuses to write a partial config — if the download or
  checksum verification fails. The user fixes the network or
  picks a reachable version and re-runs setup.
- **Daemon startup:** velocity reads prompts from the local
  extracted copy. No network calls. If the cache is missing or
  unreadable the daemon refuses to start and points the user at
  setup.
- **Updating:** a separate command (e.g.
  `velocity update-prompts <tag>`) downloads, verifies, and
  swaps in the requested release. Routine minor / patch bumps
  within the current major are non-breaking. A major bump
  (placeholder-schema change) usually requires a velocity binary
  update too — the command should warn the user and require
  explicit confirmation before applying it.

## Editing prompts

1. Edit the relevant `.md` file.
2. If you change the placeholder set, update `manifest.yaml` in
   the same commit and bump the major `version` — that gates
   when downstream velocity deployments can pick the change up.
3. Open a PR. After merge, cut a tagged release; deployments
   pull it in via `velocity update-prompts <tag>`.
