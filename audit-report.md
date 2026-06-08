# AI-Implement docs audit — testing branch — 2026-06-08

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 2 | Edits applied |
| MEDIUM | 1 | Report-only |
| LOW | 3 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Status |
|---|---|---|---|---|---|
| H-1 | HIGH | `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` not documented — custom images blocked without explanation | `workflows/claude-implement.yml:115,160-161` | `latest/customize/runner-image.mdx`, `latest/configuration/environment-variables.mdx`, `latest/changelog.mdx` | Edited |
| H-2 | HIGH | `AI_IMPLEMENT_LOG_LEVEL` target-repo variable not documented | `workflows/claude-implement.yml:230`, `src/run-autonomous.ts:185` | `latest/configuration/environment-variables.mdx`, `latest/changelog.mdx` | Edited |
| M-1 | MEDIUM | Admin UI sidebar note omits four live stub panels from Configure and Developer groups | `src/admin-ui/sidebar.ts:17-35` | `latest/reference/admin-ui.mdx` | Report |

## Edits applied (HIGH findings)

| Docs file | Findings | Markers |
|---|---|---|
| `latest/customize/runner-image.mdx` | H-1 | `{/* AUDIT H-1: AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES allowlist not documented */}` |
| `latest/configuration/environment-variables.mdx` | H-1, H-2 | `{/* AUDIT H-1: ... */}`, `{/* AUDIT H-2: ... */}` |
| `latest/changelog.mdx` | H-1, H-2 | `{/* AUDIT H-1: ... */}`, `{/* AUDIT H-2: ... */}` |

## HIGH-priority finding details

### H-1: `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` not documented — custom images blocked without explanation

- **Docs**: `latest/customize/runner-image.mdx` guides operators to use custom images from any public registry (e.g. `ghcr.io/your-org/your-runner:v1`) without mentioning an allowlist or how to configure one. `latest/configuration/environment-variables.mdx` has no entry for this variable.
- **Source**: `workflows/claude-implement.yml:115,160-161` — a `validate-runner-image` job rejects any image whose name does not begin with `ghcr.io/builddownai/` unless `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` is set; the failure message itself names the variable (`AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES=<prefix>`) but the docs never introduce it.
- **Edit**: Added a `<Warning>` block to `latest/customize/runner-image.mdx` explaining the allowlist and the variable. Added a `<ParamField body="AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES">` to the Runner image section of the Target-repo variables in `latest/configuration/environment-variables.mdx`. Added a changelog `<Update>` entry.

### H-2: `AI_IMPLEMENT_LOG_LEVEL` target-repo variable not documented

- **Docs**: `latest/configuration/environment-variables.mdx` Target-repo tab Variables section has entries for Bedrock variables, gap-fill run caps, and runner image, but no entry for `AI_IMPLEMENT_LOG_LEVEL`. `latest/changelog.mdx` has no entry for this feature.
- **Source**: `workflows/claude-implement.yml:230` and `workflows/comment-trigger.yml:232` both forward `${{ vars.AI_IMPLEMENT_LOG_LEVEL }}` into the runner container; `src/run-autonomous.ts:185` reads it via `resolveLogLevel()`, selecting between `summary` (default) and `stream` verbosity.
- **Edit**: Added a `<ParamField body="AI_IMPLEMENT_LOG_LEVEL">` with accepted values, defaults, and guidance under a new "Runner log verbosity" section in the Target-repo variables tab. Added a changelog `<Update>` entry with a cross-link to the new ParamField anchor.

## MEDIUM-priority finding details

### M-1: Admin UI sidebar note omits four live stub panels from Configure and Developer groups

- **Docs**: `latest/reference/admin-ui.mdx:14-20` lists the sidebar contents as Configure = "Projects, Pipelines & steps, Models & providers" and Developer = "Audit log, Customizations".
- **Source**: `src/admin-ui/sidebar.ts:17-35` — Configure group has five items (adding **Triggers & channels** and **Policies & risk**); Developer group has five items (adding **MCP server**, **Webhooks**, and **Updates**). Per `CLAUDE.md:134`, all five additions are currently stub routes showing a Roadmap placeholder.

## LOW-priority findings (aggregate, numbered as L-1, L-2, L-3)

- **L-1** — `latest/setup/target-repo.mdx:18` uses `gh workflow run sync-workflow.yml` as the initial sync step, but `CLAUDE.md` states the admin UI **Sync workflows** button is the normal distribution path and `sync-workflow.yml` is the manual fallback; the docs' step order (sync before create mapping) is the legacy approach.
- **L-2** — `latest/customize/runner-image.mdx:3` title says "Use a custom Fly Machine runner image" but the mechanism (`image.yml` and `AI_IMPLEMENT_RUNNER_IMAGE`) applies to GitHub Actions mode as well, which the page body correctly describes (`latest/customize/runner-image.mdx:75-78`); title alone is narrower than the actual scope.
- **L-3** — `latest/reference/admin-ui.mdx:366` PATCH endpoint description says only "Update a mapping's concurrency cap (`maxInProgressAiIssues`) or paused state (`paused`)" but the handler in `src/admin.ts:205-209` (via `handlePatchMapping`) may accept additional mapping fields; the narrow description could mislead operators writing automation against the API.
