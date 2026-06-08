# AI-Implement docs audit — main branch — 2026-06-08

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 0 | No HIGH findings this audit |
| MEDIUM | 4 | Report-only |
| LOW | 4 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Status |
|---|---|---|---|---|---|
| M-1 | MEDIUM | `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented in target-repo variables tab | `workflows/claude-implement.yml:19-21`, `workflows/comment-trigger.yml:13-15` | `configuration/environment-variables.mdx` | Report |
| M-2 | MEDIUM | Admin UI sidebar Note understates Configure (3 items) and Developer (2 items) vs actual sidebar (5+5) | `src/admin-ui/sidebar.ts:14-19,28-34` | `reference/admin-ui.mdx` | Report |
| M-3 | MEDIUM | `teamKey` ParamField body is Linear-specific; Jira users depend on page-level Tip for correct guidance | `src/config.ts:24-52` | `configuration/team-repo-mappings.mdx` | Report |
| M-4 | MEDIUM | Destroying a Jira session transitions status to `Plan Approved`, not just removes `Implementing`; docs omit the specific target state | `src/providers/jira.ts:251-254` | `reference/admin-ui.mdx`, `reference/labels.mdx` | Report |

## Edits applied (HIGH findings)

No HIGH findings — no edits applied.

## HIGH-priority finding details

No HIGH findings this audit.

## MEDIUM-priority finding details

### M-1: `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented

- **Docs**: `configuration/environment-variables.mdx` Target-repo workflow secrets tab documents `AI_IMPLEMENT_PROVIDER` and `AI_IMPLEMENT_AWS_REGION` as the only optional repo variables, with no mention of runner-image overrides.
- **Source**: `workflows/claude-implement.yml:19-21` and `workflows/comment-trigger.yml:13-15` — both workflows accept a `AI_IMPLEMENT_RUNNER_IMAGE` repo variable (overrides the default runner image per-repo or per-org) and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` (comma-separated additional allowed image prefixes beyond `ghcr.io/builddownai/`). An operator trying to use a custom runner image in GitHub Actions mode would have no documented path; if they use an unlisted prefix, the workflow fails with an error message that mentions `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` — self-discoverable, but only after a failed run.

### M-2: Admin UI sidebar Note understates sidebar group membership

- **Docs**: `reference/admin-ui.mdx:9-15` — the opening `<Note>` lists Configure as having three panels (Projects, Pipelines & steps, Models & providers) and Developer as having two (Audit log, Customizations).
- **Source**: `src/admin-ui/sidebar.ts:14-19` — Configure has five navigation items: Projects, Pipelines & steps, Models & providers, Triggers & channels, Policies & risk. `src/admin-ui/sidebar.ts:28-34` — Developer has five items: MCP server, Webhooks, Audit log, Customizations, Updates. The additional items (Triggers & channels, Policies & risk, MCP server, Webhooks, Updates) are stub/roadmap pages, not yet functional — operators who click them see a roadmap placeholder. The Note is not wrong in its descriptions, but it creates an inaccurate mental model of the sidebar; an operator using the admin UI will see more items than the docs lead them to expect.

### M-3: `teamKey` ParamField description is Linear-specific

- **Docs**: `configuration/team-repo-mappings.mdx:15-17` — the `teamKey` ParamField description reads "Your Linear team identifier (e.g. `ENG`). This is the short prefix shown in Linear issue identifiers. The orchestrator uses this key to match issues polled from Linear to the correct GitHub repo."
- **Source**: `src/config.ts:24-52` — `teamKey` (the map key in `SEED_MAPPINGS` / DB `team_key` column) is entirely provider-agnostic; it is used as an arbitrary label for any mapping regardless of ticketing system. The page-level Tip (lines 8–11) does correctly say "For Jira mappings, `teamKey` is an arbitrary identifier you choose," but a Jira operator reading only the `teamKey` ParamField body would receive inaccurate guidance implying a Linear key is required.

### M-4: Destroying a Jira session transitions to `Plan Approved`, not just clears `Implementing`

- **Docs**: `reference/admin-ui.mdx:195-198` and `reference/labels.mdx:88-90` — describe session destruction as "Removing the in-progress marker from the associated issue (`Implementing` status on Jira)," implying the status is simply removed/cleared.
- **Source**: `src/providers/jira.ts:251-254` — `clearWorkingState()` calls `setStatus(issueId, scopeKey, STATUS_VALUES.APPROVED)`, which writes the string `"Plan Approved"` to the Jira issue's status field. The issue is not cleared to neutral; it transitions specifically to `Plan Approved`, making it immediately eligible for a new implementation dispatch on the next poll cycle. Operators who destroy a stuck Jira session without wanting a re-dispatch would need to manually move the status away from `Plan Approved`, but the docs do not mention this requirement or the specific state the issue lands in.

## LOW-priority findings (aggregate, numbered as L-1, L-2, ...)

- **L-1** — Default runner image is inconsistent across synced workflow templates: `claude-implement.yml:100` falls back to `ghcr.io/builddownai/ai-implement-runner:next` while `comment-trigger.yml:115` and `claude-plan.yml` fall back to `:latest`. Not documented in user docs but creates a silent version mismatch between planning, implementation, and gap-fill runs on the same repo.
- **L-2** — Comment-trigger permission check accepts `write`, `maintain`, or `admin` roles (`comment-trigger.yml:61`), but `reference/labels.mdx:110` says the comment "must be posted by a collaborator with write access."
- **L-3** — `defaultBranch` in `configuration/team-repo-mappings.mdx:35-37` carries the default value (`main`) in the body text but the `<ParamField>` tag has no `default="main"` attribute, so the default does not appear in the rendered parameter metadata.
- **L-4** — `SESSION_IMAGE` documented as "Per-repo overrides via `.ai-implement/image.yml` take precedence" (`configuration/environment-variables.mdx:148-149`), but the CLAUDE.md developer guide notes this file is for Fly Machine mode only and the Sync workflows action from the admin UI is now the primary sync path; the relationship between `.ai-implement/image.yml`, `SESSION_IMAGE`, and the GHA-mode `AI_IMPLEMENT_RUNNER_IMAGE` repo variable is not explained together anywhere in the user docs.
