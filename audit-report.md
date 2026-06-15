# AI-Implement docs audit — testing branch — 2026-06-15

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 2 | Edits applied |
| MEDIUM | 4 | Report-only |
| LOW | 4 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Status |
|---|---|---|---|---|---|
| H-1 | HIGH | `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented — GHA custom-image allowlist unmentioned | `workflows/claude-implement.yml:121–169`, `workflows/comment-trigger.yml:127–173` | `latest/customize/runner-image.mdx`, `latest/configuration/environment-variables.mdx` | Edited |
| H-2 | HIGH | `GAP_FILL_TRIGGER_SECRET` docs claim comment-trigger.yml calls `/trigger/gap-fill`; it does not | `workflows/comment-trigger.yml` (no call to orchestrator endpoint) | `latest/configuration/environment-variables.mdx`, `latest/reference/admin-ui.mdx` | Edited |
| M-1 | MEDIUM | Admin UI sidebar Note lists incomplete group membership (Configure + Developer groups) | `src/admin-ui/sidebar.ts:14–34` | `latest/reference/admin-ui.mdx` | Report |
| M-2 | MEDIUM | `branchPrefix` mapping field not documented | `src/config.ts:59`, `src/admin.ts:1093–1098` | `latest/configuration/team-repo-mappings.mdx` | Report |
| M-3 | MEDIUM | `AI_IMPLEMENT_RUNNER_LABEL` target-repo variable undocumented | `workflows/claude-implement.yml:179` | `latest/configuration/environment-variables.mdx` | Report |
| M-4 | MEDIUM | `AI_IMPLEMENT_LOG_LEVEL` target-repo variable undocumented | `workflows/claude-implement.yml:241`, `workflows/comment-trigger.yml:232` | `latest/configuration/environment-variables.mdx` | Report |

## Edits applied (HIGH findings)

| Docs file | Findings | Markers |
|---|---|---|
| `latest/customize/runner-image.mdx` | H-1 | `{/* AUDIT H-1: AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES missing … */}` |
| `latest/configuration/environment-variables.mdx` | H-1, H-2 | `{/* AUDIT H-1: … */}`, `{/* AUDIT H-2: … */}` |
| `latest/reference/admin-ui.mdx` | H-2 | `{/* AUDIT H-2: /trigger/gap-fill description corrected … */}` |

---

## HIGH-priority finding details

### H-1: `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented — GHA custom-image allowlist unmentioned

- **Docs**: `latest/customize/runner-image.mdx` states that `.ai-implement/image.yml` works for both execution modes and that "In GitHub Actions mode, the orchestrator forwards your image to `claude-implement.yml` as the `container.image`" — with no mention of any allowlist restriction.
- **Source**: `workflows/claude-implement.yml:121–169` — the `validate-runner-image` job allows only images matching `ghcr.io/builddownai/` by default; `workflows/comment-trigger.yml:127–173` has the identical check. Both reject non-matching images with an error suggesting the reader set `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`. The variable is also referenced in the workflow header comments (`claude-implement.yml:18`, `comment-trigger.yml:21–22`).
- **Edit**: Added a `<Warning>` after the `<Note>` in `runner-image.mdx` describing the GHA allowlist and how to expand it with `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`. Added a `<ParamField>` for the variable in the "Runner image" section of the Target-repo workflow secrets tab in `environment-variables.mdx`.

### H-2: `GAP_FILL_TRIGGER_SECRET` docs claim comment-trigger.yml calls `/trigger/gap-fill`; it does not

- **Docs**: `latest/configuration/environment-variables.mdx` (Gap-fill trigger accordion) says the secret is "for the `/trigger/gap-fill` endpoint that synced `comment-trigger.yml` workflows POST to when a user comments `/ai-implement`"; `latest/reference/admin-ui.mdx` endpoint table says `/trigger/gap-fill` is "Called by `comment-trigger.yml`."
- **Source**: `workflows/comment-trigger.yml` (entire file) — the workflow runs the implementation pipeline directly inside the GitHub Actions job via the runner container; it does not make any HTTP call to the orchestrator's `/trigger/gap-fill` endpoint. `GAP_FILL_TRIGGER_SECRET` is not referenced anywhere in the workflow file.
- **Edit**: Updated the `GAP_FILL_TRIGGER_SECRET` `<ParamField>` description in `environment-variables.mdx` with a `<Warning>` clarifying that the current `comment-trigger.yml` does not use this endpoint. Updated the `/trigger/gap-fill` row in the `admin-ui.mdx` endpoint table to remove the false "Called by `comment-trigger.yml`" claim.

---

## MEDIUM-priority finding details

### M-1: Admin UI sidebar Note lists incomplete group membership

- **Docs**: `latest/reference/admin-ui.mdx` Note callout lists Configure as having three items (Projects, Pipelines & steps, Models & providers) and Developer as having two (Audit log, Customizations).
- **Source**: `src/admin-ui/sidebar.ts:14–34` — Configure has five items (adding Triggers & channels and Policies & risk); Developer has five items (adding MCP server, Webhooks, and Updates). These five extra items are confirmed as stub/partially-implemented pages in `src/admin-ui/pages/stubs.ts:55–113`.

### M-2: `branchPrefix` mapping field not documented

- **Docs**: `latest/configuration/team-repo-mappings.mdx` ends at `maxJobMinutes`; `branchPrefix` appears nowhere in the docs.
- **Source**: `src/config.ts:59` (`branchPrefix: string | null`); `src/admin.ts:1093–1098` (validation and storage). The field is configurable in the Projects stepper and modifies implementation branch names by prepending a path segment.

### M-3: `AI_IMPLEMENT_RUNNER_LABEL` target-repo variable undocumented

- **Docs**: `latest/configuration/environment-variables.mdx` Target-repo workflow secrets tab lists `AI_IMPLEMENT_PROVIDER`, `AI_IMPLEMENT_AWS_REGION`, three run-cap variables, and `AI_IMPLEMENT_RUNNER_IMAGE` — but not `AI_IMPLEMENT_RUNNER_LABEL`.
- **Source**: `workflows/claude-implement.yml:179` — `runs-on: ${{ vars.AI_IMPLEMENT_RUNNER_LABEL || 'ubuntu-latest' }}`. Controls the GitHub Actions runner label for the implement job; setting it to a larger-runner label (e.g. `ubuntu-latest-4-cores`) reduces wall-clock time for CPU-bound test suites.

### M-4: `AI_IMPLEMENT_LOG_LEVEL` target-repo variable undocumented

- **Docs**: `latest/configuration/environment-variables.mdx` Target-repo workflow secrets tab does not document `AI_IMPLEMENT_LOG_LEVEL`.
- **Source**: `workflows/claude-implement.yml:241` and `workflows/comment-trigger.yml:232` — both pass `${{ vars.AI_IMPLEMENT_LOG_LEVEL }}` to the runner. Controls runner log verbosity (`summary` default, `stream` for per-turn tool activity).

---

## LOW-priority findings (aggregate, numbered as L-1, L-2, …)

- **L-1** — `latest/setup/target-repo.mdx` Step 2 instructs readers to use `gh workflow run sync-workflow.yml` for initial sync; the admin UI's **Sync workflows** action on the Projects row is the primary operator path; `sync-workflow.yml` is now a manual fallback (`src/admin-ui/sidebar.ts` + CLAUDE.md context).
- **L-2** — `latest/reference/labels.mdx:114` says the `/ai-implement` comment trigger requires "write access" but `workflows/comment-trigger.yml:71` accepts `write`, `maintain`, or `admin` permission — a collaborator with `maintain` would be incorrectly told they lack access.
- **L-3** — `latest/configuration/planning.mdx:10` says "Claude posts structured analysis directly to the originating issue"; `workflows/claude-plan.yml:325–343` and `workflows/PLANNING.md:92–93` show Claude writes to `ai-output/comments/*.md` files and the orchestrator posts them via callback — an internal detail but the docs description is inaccurate.
- **L-4** — `latest/configuration/planning.mdx:28` says Work Units is "Skipped when the issue is too small or too tightly coupled to decompose"; `workflows/PLANNING.md:128–145` instructs Claude to always write Comment 3 (Work Units) with no conditional skip logic — the conditionality is not in the prompt template.
