# AI-Implement docs audit — testing branch — 2026-06-08

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 0 | No HIGH findings this audit |
| MEDIUM | 4 | Report-only |
| LOW | 5 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Status |
|---|---|---|---|---|---|
| M-1 | MEDIUM | `GAP_FILL_TRIGGER_SECRET` desc incorrectly attributes comment-trigger.yml as caller | `comment-trigger.yml:177-241`, `index.ts:113` | `latest/configuration/environment-variables.mdx`, `latest/reference/admin-ui.mdx` | Report |
| M-2 | MEDIUM | `AI_IMPLEMENT_LOG_LEVEL` target-repo variable undocumented | `claude-implement.yml:230`, `comment-trigger.yml:232` | `latest/configuration/environment-variables.mdx` | Report |
| M-3 | MEDIUM | `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` target-repo variable undocumented | `claude-implement.yml:16-17`, `comment-trigger.yml:21` | `latest/configuration/environment-variables.mdx` | Report |
| M-4 | MEDIUM | `teamKey` ParamField body describes only Linear semantics, no Jira alternative | `config.ts:24`, `admin.ts:104` | `latest/configuration/team-repo-mappings.mdx` | Report |

## Edits applied (HIGH findings)

No HIGH findings — no edits applied.

## HIGH-priority finding details

No HIGH findings this audit.

## MEDIUM-priority finding details

### M-1: `GAP_FILL_TRIGGER_SECRET` docs incorrectly say `comment-trigger.yml` calls `/trigger/gap-fill`

- **Docs**: `latest/configuration/environment-variables.mdx:179` says `GAP_FILL_TRIGGER_SECRET` is the "shared bearer secret for the `/trigger/gap-fill` endpoint that synced `comment-trigger.yml` workflows POST to when a user comments `/ai-implement` on a PR." `latest/reference/admin-ui.mdx:424` similarly says the endpoint is "Called by `comment-trigger.yml` in your target repo when a user comments `/ai-implement` on a PR."
- **Source**: `ai-implement/workflows/comment-trigger.yml:177-241` — the comment-trigger workflow in the testing branch does NOT call the orchestrator at all. It dispatches the implementation workflow directly as a GitHub Actions container step (`Run pipeline (gap-fill)`). The endpoint exists in `ai-implement/src/index.ts:1932` and the startup log at `index.ts:113` references `/trigger/gap-fill`, but no current workflow calls it.

### M-2: `AI_IMPLEMENT_LOG_LEVEL` target-repo variable absent from environment-variables.mdx

- **Docs**: `latest/configuration/environment-variables.mdx` Target-repo Variables section documents Bedrock variables, gap-fill run caps, and runner image override, but has no entry for `AI_IMPLEMENT_LOG_LEVEL`.
- **Source**: `ai-implement/workflows/claude-implement.yml:230` and `ai-implement/workflows/comment-trigger.yml:232` both read `vars.AI_IMPLEMENT_LOG_LEVEL` at runtime. The variable accepts `summary` (default — one result line per invocation) or `stream` (additionally tees per-turn tool activity to the workflow log). Operators who want to debug a stalled run have no docs path to discover this variable.

### M-3: `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` target-repo variable absent from environment-variables.mdx

- **Docs**: `latest/configuration/environment-variables.mdx` documents `AI_IMPLEMENT_RUNNER_IMAGE` but has no entry for `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`.
- **Source**: `ai-implement/workflows/claude-implement.yml:16-17` and `ai-implement/workflows/comment-trigger.yml:21` both accept this variable. When set, its comma-separated prefixes are added to the allowlist of registry prefixes permitted for the runner image (default only allows `ghcr.io/builddownai/`). Operators using a custom runner image from their own registry would encounter a workflow validation error with a self-documenting hint, but the variable itself is not in the docs.

### M-4: `teamKey` ParamField body describes only Linear semantics for a field shared by both providers

- **Docs**: `latest/configuration/team-repo-mappings.mdx:18-23` — the `teamKey` ParamField body says "Your Linear team identifier (e.g. `ENG`). This is the short prefix shown in Linear issue identifiers. The orchestrator uses this key to match issues polled from Linear to the correct GitHub repo." No mention of Jira.
- **Source**: `ai-implement/src/config.ts:24` — `teamKey` is the primary key for both Linear and Jira mappings. For Jira, it is an operator-chosen arbitrary identifier (there is no equivalent concept in Jira), as confirmed in `ai-implement/src/admin.ts:104` where the field is used generically. A page-level Tip (lines 12-15) does clarify the Jira case, but the ParamField body itself is Linear-only.

## LOW-priority findings (aggregate, numbered as L-1, L-2, ...)

- **L-1** — `comment-trigger.yml:71` accepts `write`, `maintain`, and `admin` permission levels for the `/ai-implement` trigger; `latest/reference/labels.mdx:114` says only "write access" — docs are more restrictive than the code.
- **L-2** — `REAPER_ALERT_THRESHOLD` default of `10` (set at `index.ts:149`) is not documented in the ParamField body in `latest/configuration/environment-variables.mdx`.
- **L-3** — Admin-ui.mdx sidebar Note (lines 12–21) lists Configure as having 3 items and Developer as 2 items; `sidebar.ts:14-34` shows 5 items in Configure (adds Triggers & channels, Policies & risk) and 5 in Developer (adds MCP server, Webhooks, Updates) — extra items are stubs showing "Coming soon" content.
- **L-4** — `latest/reference/admin-ui.mdx` runner callback endpoint table omits `GET /runner/planning-context` (`index.ts:1862`) — internal runner endpoint, no operator action required.
- **L-5** — `latest/providers/aws-bedrock.mdx:18` says `aws-actions/configure-aws-credentials` "happens twice per run: once before the main implementation run and once before the gap analysis step"; `claude-implement.yml:199-205` shows only one credentials step, with a 4-hour session covering the entire runner container invocation (confirmed in CLAUDE.md).
