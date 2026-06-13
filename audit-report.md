# AI-Implement docs audit (multi-run synthesis, N=3) — main branch — 2026-06-13

Synthesized from 3 independent audit runs against the same source+docs state.
Recurrence column shows how many runs flagged each finding.

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 4 | Edits applied |
| MEDIUM | 5 | Report-only |
| LOW | 6 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Runs | Status |
|---|---|---|---|---|---|---|
| H-1 | HIGH | Custom pipeline YAML uses invalid `type: feedback-loop` | `pipelines/autonomous.yml:7-9`, `src/pipeline/pipeline-loader.ts:7-22` | `customize/custom-steps.mdx` | 1/3 | Edited |
| H-2 | HIGH | Eight step IDs listed as overridable cannot be overridden via `custom/steps/` | `src/pipeline/default-pipeline.ts:20-27`, `src/pipeline/planning-pipeline.ts:104-114` | `customize/custom-steps.mdx` | 1/3 | Edited |
| H-3 | HIGH | GHA runner image customization missing — docs imply it's impossible | `workflows/claude-implement.yml:88-92`, `workflows/comment-trigger.yml:108-109` | `customize/runner-image.mdx` | 1/3 | Edited |
| H-4 | HIGH | Jira target-repo secrets documented as required but not read by any workflow | `workflows/claude-implement.yml`, `workflows/claude-plan.yml`, `workflows/comment-trigger.yml` | `configuration/environment-variables.mdx` | 1/3 | Edited |
| M-1 | MEDIUM | Admin UI sidebar overview omits five stub panels | `src/admin-ui/sidebar.ts:14-34` | `reference/admin-ui.mdx` | 3/3 | Report |
| M-2 | MEDIUM | Secrets sidebar panel described as functional; actual panel is a stub | `src/admin-ui/pages/stubs.ts:79-87` | `reference/admin-ui.mdx` | 2/3 | Report |
| M-3 | MEDIUM | `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented in stable docs | `workflows/claude-implement.yml:18-21`, `workflows/comment-trigger.yml:13-15` | `configuration/environment-variables.mdx` | 3/3 | Report |
| M-4 | MEDIUM | Setup guide prescribes manual `sync-workflow.yml` as primary sync path | `ai-implement/CLAUDE.md:150-151` | `setup/target-repo.mdx` | 2/3 | Report |
| M-5 | MEDIUM | `teamKey` field description is Linear-only but field is used in Jira mappings too | `src/config.ts:24-52`, `src/providers/jira.ts:134` | `configuration/team-repo-mappings.mdx` | 2/3 | Report |

## Edits applied (HIGH findings)

| Docs file | Findings | Edit source |
|---|---|---|
| `customize/custom-steps.mdx` | H-1, H-2 | run-1 (only candidate) |
| `customize/runner-image.mdx` | H-3 | run-2 (only candidate) |
| `configuration/environment-variables.mdx` | H-4 | run-3 (only candidate) |

## HIGH-priority finding details

### H-1: Custom pipeline YAML uses invalid `type: feedback-loop`  (1/3 runs)

- **Docs**: `customize/custom-steps.mdx` showed a pipeline YAML example with `- id: feedback-loop, type: feedback-loop`.
- **Source**: `src/pipeline/pipeline-loader.ts:7-22` — `VALID_STEP_TYPES` does not include `feedback-loop`; the validator rejects any step whose type is not in that set. The actual `pipelines/autonomous.yml:7-9` uses `type: custom, moduleId: feedback-loop`.
- **Edit**: Changed `type: feedback-loop` to `type: custom` with `moduleId: feedback-loop` in the example YAML. Updated the `<Tip>` to clarify that `feedback-loop` requires `type: custom, moduleId: feedback-loop` and is not a recognized type value — selected from run-1 (only candidate).
- **Priority agreement**: HIGH in 1/3 runs (only Run 1 surfaced this finding).

### H-2: Eight step IDs listed as overridable cannot be overridden via `custom/steps/`  (1/3 runs)

- **Docs**: `customize/custom-steps.mdx` stated "Any of them can be overridden by placing a matching file in `custom/steps/`" across all 14 accordion items, including planning-phase steps and `implement`/`review`.
- **Source**: `src/pipeline/default-pipeline.ts:20-27` — only six steps are checked for custom overrides: `clone`, `install`, `feedback-loop`, `preflight`, `push`, `post-push-review`. `src/pipeline/planning-pipeline.ts:104-114` — planning steps are registered as direct built-in imports with no custom-override resolution. `src/pipeline/steps/feedback-loop.ts:3-4` — `implementStep` and `reviewStep` are imported directly, bypassing any `custom/steps/` file.
- **Edit**: Replaced the intro text with a precise split — overridable autonomous pipeline steps listed explicitly; planning-phase steps and `implement`/`review` marked as reference-only with a `<Warning>` explaining how to actually customize them — selected from run-1 (only candidate).
- **Priority agreement**: HIGH in 1/3 runs (only Run 1 surfaced this finding).

### H-3: GHA runner image customization missing — docs imply it's impossible  (1/3 runs)

- **Docs**: `customize/runner-image.mdx` Note stated "This mechanism only applies to the **fly-machines** execution mode. GitHub Actions runs use the `ubuntu-latest` runner provided by GitHub and are not affected by `.ai-implement/image.yml`." Readers following this would believe GHA-mode runner environments cannot be customized.
- **Source**: `workflows/claude-implement.yml:88-92` reads `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` repo variables and boots the implement job inside that container. `workflows/comment-trigger.yml:108-109` reads the same variables for gap-fill runs.
- **Edit**: Replaced the misleading Note with one accurately saying `.ai-implement/image.yml` is Fly-only. Added a `<Tip>` explaining that GHA-mode repos can set `AI_IMPLEMENT_RUNNER_IMAGE` to a custom image and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` to allow non-builddownai registries — selected from run-2 (only candidate).
- **Priority agreement**: HIGH in 1/3 runs (only Run 2 surfaced this finding).

### H-4: Jira target-repo secrets documented as required but not read by any workflow  (1/3 runs)

- **Docs**: `configuration/environment-variables.mdx` Target-repo workflow secrets tab contained a Jira accordion declaring `JIRA_TOKEN`, `JIRA_CLOUD_ID`, and `JIRA_SITE_URL` as required secrets, described as being used by the planning and implementation workflows.
- **Source**: `workflows/claude-implement.yml` passes `LINEAR_API_KEY` to the runner but no Jira credentials; `workflows/claude-plan.yml` passes `LINEAR_API_KEY` for label updates but no Jira credentials; `workflows/comment-trigger.yml` similarly omits all Jira secrets. The Jira client lives entirely inside the orchestrator service (`src/providers/jira.ts`).
- **Edit**: Replaced the `### Ticketing (one provider required)` section (including Jira accordion and three `<ParamField>` blocks) with a single `LINEAR_API_KEY` `<ParamField>` and a `<Note>` directing Jira operators to configure credentials on the orchestrator (Orchestrator runtime tab) rather than on target repos — selected from run-3 (only candidate).
- **Priority agreement**: HIGH in 1/3 runs (only Run 3 surfaced this finding).

## MEDIUM-priority finding details

### M-1: Admin UI sidebar overview omits five stub panels  (3/3 runs)

- **Docs**: `reference/admin-ui.mdx` Note (lines 8–17) lists Configure with three items (Projects, Pipelines & steps, Models & providers) and Developer with two items (Audit log, Customizations) — five sidebar items a reader would see in the actual UI are absent.
- **Source**: `src/admin-ui/sidebar.ts:14-34` adds Triggers & channels and Policies & risk to Configure; MCP server, Webhooks, and Updates to Developer. All five render as "Coming soon" stubs (`src/admin-ui/pages/stubs.ts:54-114`) — visible in the sidebar but not acknowledged by the docs.

### M-2: Secrets sidebar panel described as functional; actual panel is a stub  (2/3 runs)

- **Docs**: `reference/admin-ui.mdx` `### Secrets` section (lines 234–252) describes per-team secrets at `/api/mappings/:teamKey/secrets` and global secrets at `/api/global-secrets` as though a dedicated management UI exists.
- **Source**: `src/admin-ui/pages/stubs.ts:79-87` — the Secrets sidebar item renders a "Coming soon" stub directing users to the Settings page (global secrets) and the Projects page (per-team secrets). The API endpoints are implemented but there is no dedicated Secrets UI panel.

### M-3: `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented in stable docs  (3/3 runs)

- **Docs**: `configuration/environment-variables.mdx` Target-repo workflow secrets tab has no section for GHA runner image variables.
- **Source**: `workflows/claude-implement.yml:18-21` and `workflows/comment-trigger.yml:13-15` document both variables in workflow header comments as optional repo/org variables that control the runner image for GitHub Actions execution mode.

### M-4: Setup guide prescribes manual `sync-workflow.yml` as primary sync path  (2/3 runs)

- **Docs**: `setup/target-repo.mdx` Step 2 instructs operators to run `gh workflow run sync-workflow.yml` from the AI-Implement repo as the primary way to push workflow templates.
- **Source**: `ai-implement/CLAUDE.md:150-151` states `.github/workflows/sync-workflow.yml` remains as a manual fallback; normal distribution should happen from the orchestrator via the **Sync workflows** admin UI button (`src/admin-ui/pages/projects.ts:239`).

### M-5: `teamKey` field description is Linear-only but field is used in Jira mappings too  (2/3 runs)

- **Docs**: `configuration/team-repo-mappings.mdx` `<ParamField>` for `teamKey` reads "Your Linear team identifier (e.g. `ENG`)…The orchestrator uses this key to match issues polled from Linear to the correct GitHub repo." Jira users get no guidance.
- **Source**: `src/config.ts:24-52` defines `RepoMapping` with a `ticketingProvider` field covering both providers. `src/providers/jira.ts:134` uses `teamKey` only as an arbitrary scope key — not a Linear team identifier.
- **Priority note**: HIGH in 0/3 runs; MEDIUM in 1/3 (Run 2), LOW in 1/3 (Run 1). Kept MEDIUM per no-downgrade rule.

## LOW-priority findings (aggregate, numbered as L-1 … L-6)

- **L-1** — Default runner image tag differs between workflow templates: `claude-implement.yml` falls back to `:next` (`workflows/claude-implement.yml:100`) while `comment-trigger.yml` falls back to `:latest` (`workflows/comment-trigger.yml:115`); neither inconsistency is documented (3/3 runs)
- **L-2** — `/ai-implement` comment trigger accepts `write`, `maintain`, and `admin` permission levels but `reference/labels.mdx:110` says only "write access" (`workflows/comment-trigger.yml:53-63`) (2/3 runs)
- **L-3** — `await_ci` is present in `src/pipeline/types.ts:8` and `src/pipeline/pipeline-loader.ts:14` as a valid step type but has no built-in step file and is undocumented in `customize/custom-steps.mdx`; appears reserved (2/3 runs)
- **L-4** — `reference/admin-ui.mdx:41` describes the Pipelines sidebar panel using its internal route key `jobs` in a parenthetical; the user-visible label is "Pipelines" (2/3 runs)
- **L-5** — `.env.example` comment says the gap-fill trigger endpoint is `/api/gap-fill-trigger` but the actual path in `src/index.ts:1609` is `POST /trigger/gap-fill` (1/3 runs)
- **L-6** — `providers/anthropic.mdx:116` lists `claude-opus-4-7` as an example model ID, which may be outdated; the example is illustrative but could lead operators to a defunct ID (1/3 runs)

## Synthesis decisions log

### H-1

- **Present in:** Run 1 HIGH
- **Dedup reasoning:** Only Run 1 surfaced this finding. Distinguished from H-2 (which targets the incorrect overridability claim on the same page): H-1 is specifically about the YAML example using an invalid `type` value.
- **Selected edit from:** Run 1 (only candidate — no selection needed)

### H-2

- **Present in:** Run 1 HIGH
- **Dedup reasoning:** Only Run 1 surfaced this finding. Distinguished from H-1 (the YAML type error): H-2 is about the broader prose claim that all 14 steps are overridable via `custom/steps/`, which is contradicted by source code showing only 6 autonomous-pipeline steps are wired for custom override resolution.
- **Selected edit from:** Run 1 (only candidate — no selection needed)

### H-3

- **Present in:** Run 2 HIGH
- **Dedup reasoning:** Only Run 2 surfaced this finding. Kept distinct from M-3 (runner image variables missing from `environment-variables.mdx`) because H-3 targets a different docs file (`customize/runner-image.mdx`) and a different form of drift: an actively wrong claim ("impossible") vs a missing reference in a variables list.
- **Selected edit from:** Run 2 (only candidate — no selection needed)

### H-4

- **Present in:** Run 3 HIGH
- **Dedup reasoning:** Only Run 3 surfaced this finding. The Credentials at a glance table at the bottom of `environment-variables.mdx` still references "Orchestrator + Jira target repos" for the Jira credentials — this table was not part of Run 3's submitted edit scope and so was not modified.
- **Selected edit from:** Run 3 (only candidate — no selection needed)

### M-1

- **Present in:** Run 1 MEDIUM, Run 2 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** All three runs cite the same source (`src/admin-ui/sidebar.ts` adding the same five items) and the same docs surface (`reference/admin-ui.mdx` Note listing sidebar groups). Title wording differed across runs ("omits five sidebar items" / "omits five stub panels" / "missing five stub panels") but the underlying drift is identical.

### M-2

- **Present in:** Run 2 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** Both cite `src/admin-ui/pages/stubs.ts` (same line range) and describe the same drift: `reference/admin-ui.mdx` presents the Secrets panel as a functional management interface while the source renders a "Coming soon" stub.

### M-3

- **Present in:** Run 1 MEDIUM, Run 2 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** All three runs cite the same two source locations (`workflows/claude-implement.yml:18-21` and `workflows/comment-trigger.yml:13-15`) and describe the same gap — `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` absent from stable docs. Run 2 additionally listed `setup/target-repo.mdx` as a docs target; not carried into the union since it was not the primary surface in the other two runs.

### M-4

- **Present in:** Run 2 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** Both cite `ai-implement/CLAUDE.md:150-151` and identify the same divergence: `setup/target-repo.mdx` Step 2 uses `sync-workflow.yml` as the primary path while CLAUDE.md says the admin UI **Sync workflows** button is normal and the workflow is a fallback.

### M-5

- **Present in:** Run 2 MEDIUM, Run 1 LOW
- **Dedup reasoning:** Run 1 L-4 and Run 2 M-3 describe the same drift: the `teamKey` ParamField body in `configuration/team-repo-mappings.mdx` references only Linear terminology while the field applies to both Linear and Jira mappings.
- **Priority resolution:** Kept MEDIUM per no-downgrade rule; Run 1 rated this LOW, Run 2 rated it MEDIUM.
