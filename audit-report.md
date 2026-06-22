# AI-Implement docs audit — testing branch — 2026-06-22

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 2 | Edits applied |
| MEDIUM | 3 | Report-only |
| LOW | 5 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Status |
|---|---|---|---|---|---|
| H-1 | HIGH | `branchPrefix` field missing from project mappings reference | `src/config.ts:58` | `latest/configuration/team-repo-mappings.mdx` | Edited |
| H-2 | HIGH | `AI_IMPLEMENT_RUNNER_LABEL` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` missing from target-repo variables | `workflows/claude-implement.yml:18-20` | `latest/configuration/environment-variables.mdx` | Edited |
| M-1 | MEDIUM | `latest/customize/runner-image.mdx` doesn't mention the image-prefix allowlist needed for third-party registries | `workflows/claude-implement.yml:122-194` | `latest/customize/runner-image.mdx` | Report |
| M-2 | MEDIUM | `latest/reference/admin-ui.mdx` sidebar Note lists incomplete Configure and Developer groups | `src/admin-ui/sidebar.ts:14-35` | `latest/reference/admin-ui.mdx` | Report |
| M-3 | MEDIUM | `latest/configuration/team-repo-mappings.mdx` `teamKey` ParamField body describes only the Linear case | `src/config.ts:24-60`, `src/admin.ts:109-116` | `latest/configuration/team-repo-mappings.mdx` | Report |

## Edits applied (HIGH findings)

| Docs file | Findings | Markers |
|---|---|---|
| `latest/configuration/team-repo-mappings.mdx` | H-1 | `{/* AUDIT H-1: branchPrefix field missing from project mappings */}` |
| `latest/configuration/environment-variables.mdx` | H-2 | `{/* AUDIT H-2: AI_IMPLEMENT_RUNNER_LABEL and AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES missing from target-repo variables */}` |

## HIGH-priority finding details

### H-1: `branchPrefix` field missing from project mappings reference

- **Docs**: `latest/configuration/team-repo-mappings.mdx` documents project mapping fields through `maxJobMinutes` and the run-caps tip, but `branchPrefix` is absent — readers have no way to discover the feature.
- **Source**: `src/config.ts:58` — `branchPrefix: string | null` is a first-class mapping field with format validation, admin UI support (a dedicated Branch Prefix input in the Projects edit dialog), and dispatch wiring that prefixes implementation branch names as a path segment (e.g. `pr` → `pr/ai-implement/PROJ-123-...`); also documented in `ai-implement/CLAUDE.md` under "Per-project branch prefix (admin UI)".
- **Edit**: Added a `<ParamField body="branchPrefix">` block after the run-caps Tip, including the accepted format constraints and a `<Warning>` about the comment-trigger gap-fill exclusion and the re-sync requirement.

### H-2: `AI_IMPLEMENT_RUNNER_LABEL` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` missing from target-repo variables

- **Docs**: `latest/configuration/environment-variables.mdx` target-repo variables section covers Bedrock routing, run-cap variables, and `AI_IMPLEMENT_RUNNER_IMAGE`, but omits two other operator-facing runner variables — readers who want larger runners or third-party images cannot discover these.
- **Source**: `workflows/claude-implement.yml:18-20` — the workflow header comments document both `AI_IMPLEMENT_RUNNER_LABEL` ("runs-on label for the implement job, default ubuntu-latest; set to a larger-runner label to speed up CPU-bound test suites") and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` ("Extra image prefixes allowed in addition to ghcr.io/builddownai/ and the repo owner's own ghcr.io/\<owner\>/ namespace"); the allowlist enforcement is in the `validate-runner-image` job at lines 122–194.
- **Edit**: Added `<ParamField body="AI_IMPLEMENT_RUNNER_LABEL">` and `<ParamField body="AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES">` blocks after the existing `AI_IMPLEMENT_RUNNER_IMAGE` ParamField, with a `<Warning>` on the runner-label security threat model and inline format examples for the prefix list.

## MEDIUM-priority finding details

### M-1: Runner-image page doesn't mention allowlist requirement for third-party registries

- **Docs**: `latest/customize/runner-image.mdx` documents the `.ai-implement/image.yml` mechanism and `AI_IMPLEMENT_RUNNER_IMAGE` but does not mention that images outside the auto-trusted `ghcr.io/builddownai/` and repo-owner `ghcr.io/<owner>/` namespaces require `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` to be set — readers using a vendor or internal registry get an opaque "not allowed" dispatch failure.
- **Source**: `workflows/claude-implement.yml:122-194` — the `validate-runner-image` job enforces the prefix allowlist and emits the error message that points at `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`, but only operators who inspect the workflow file would see this.

### M-2: Admin UI sidebar Note incomplete for Configure and Developer groups

- **Docs**: `latest/reference/admin-ui.mdx` Note lists **Configure** as "Projects, Pipelines & steps, Models & providers" and **Developer** as "Audit log, Customizations" — both omit entries visible in the sidebar.
- **Source**: `src/admin-ui/sidebar.ts:14-35` — Configure also contains "Triggers & channels" (`channels`) and "Policies & risk" (`policies`); Developer also contains "MCP server" (`mcp`), "Webhooks" (`webhooks`), and "Updates" (`updates`). All five omitted items are currently stub pages (confirmed in `src/admin-ui/pages/stubs.ts`), so the omission is unlikely to cause incorrect operator actions — but the Note misrepresents the sidebar structure.

### M-3: `teamKey` ParamField body describes only Linear semantics

- **Docs**: `latest/configuration/team-repo-mappings.mdx:19-21` — the `teamKey` ParamField body says "Your Linear team identifier (e.g. `ENG`). This is the short prefix shown in Linear issue identifiers. The orchestrator uses this key to match issues polled from Linear to the correct GitHub repo." This description is entirely Linear-specific.
- **Source**: `src/admin.ts:109-116` — `validateTicketingMapping` accepts `ticketingProvider: "linear" | "jira"` with no special teamKey constraint for Jira; the field is an arbitrary operator-chosen label in that context. The Tip block above the field does note the Jira difference, but the ParamField body itself is inaccurate for Jira users.

## LOW-priority findings (aggregate, numbered as L-1 … L-5)

- **L-1** — `latest/configure/custom-steps.mdx` pipeline YAML example omits the `setup` and `verify` steps that the default `pipelines/autonomous.yml` includes; the example is valid but incomplete as a reference for recreating the default pipeline. (`ai-implement/pipelines/autonomous.yml:8-19`)
- **L-2** — `latest/customize/runner-image.mdx` describes the page as covering Fly Machine runners ("Use a custom Fly Machine runner image"), but its own Note says the mechanism works for both execution modes; the page title implies Fly-only. (`latest/customize/runner-image.mdx:1-3`)
- **L-3** — `latest/reference/admin-ui.mdx` `Customizations` section says the `protect-custom.yml` CI check is "currently advisory — it won't hard-block a merge but will surface a warning in CI" with the team tracking a hard-fail restore; no corresponding source evidence this is still soft (the CI file state was not audited). Worth re-verifying before treating the advisory note as accurate. (`latest/reference/admin-ui.mdx:298-300`)
- **L-4** — `latest/configuration/environment-variables.mdx` Orchestrator runtime tab documents `SESSION_IMAGE` with default `ghcr.io/builddownai/ai-implement-runner:latest`; CLAUDE.md notes `SESSION_IMAGE` is the deprecated former name of the env var and the orchestrator logs a deprecation warning, but the docs make no mention of the deprecation or the migration path. (`ai-implement/CLAUDE.md:278`)
- **L-5** — `latest/configuration/team-repo-mappings.mdx` Tip at the top has a trailing colon in the prose "for `/ai-implement` comment-triggered gap-fill runs.:" (line 155). Minor copy error.
