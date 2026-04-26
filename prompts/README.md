# prompts/

Externalized LLM prompts and Jira / GitHub failure-comment templates
for the [velocity](https://github.com/randheer094/velocity) daemon.
velocity fetches `manifest.yaml` on startup, then fetches each
referenced template, and renders them with Go's `text/template` at
call sites.

## Why

Keeping prompts in this repo means:

- Prompts can be tuned without rebuilding velocity.
- Prompt history lives in git — diffs, blame, and PR review apply.
- Multiple velocity deployments can pin to a tag/SHA of this repo
  for reproducibility, or float on `main` for live updates.

## Layout

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

## Template format

Templates use Go's [`text/template`](https://pkg.go.dev/text/template)
syntax. Placeholders are referenced as `{{.FieldName}}`. The exact
field set for each prompt is declared in `manifest.yaml` under
`placeholders`.

The renderer in velocity must:

1. Fetch `manifest.yaml` at startup.
2. For each entry, fetch the file at `path` (relative to
   `manifest.yaml`).
3. Parse with `text/template.New(id).Option("missingkey=error").Parse`.
4. Cache the parsed template by `id`.
5. On every render, pass a struct (or `map[string]any`) whose keys
   match the placeholders for that prompt.

`missingkey=error` is intentional — a typo in a placeholder name
should fail loudly during render, not silently produce `<no value>`
in a Jira comment.

## Fetching

Default source: `https://raw.githubusercontent.com/randheer094/velocity-resources/main/prompts/`.

Pin a specific revision by replacing `main` with a tag or commit SHA
in velocity's config. Velocity should fall back to bundled defaults
(the strings currently in `internal/arch/prompt.go` etc.) if the
fetch fails on startup, so the daemon stays bootable on a network
hiccup.

## Editing prompts

1. Edit the relevant `.md` file.
2. If you change the placeholder set, update `manifest.yaml` in the
   same commit and bump `version`.
3. Render-test locally: `go run` a small script that loads the
   manifest and renders each template against fixture data.
4. Open a PR. Review focuses on the prompt itself; downstream
   velocity deployments pick up the change as soon as they restart
   (or via `velocity reload-prompts`, if implemented).
