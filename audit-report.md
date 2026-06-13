# AI-Implement docs audit (multi-run synthesis, N=3) — testing branch — 2026-06-13

Synthesized from 3 independent audit runs against the same source+docs state.
Recurrence column shows how many runs flagged each finding.

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 6 | Edits applied |
| MEDIUM | 3 | Report-only |
| LOW | 7 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Runs | Status |
|---|---|---|---|---|---|---|
| H-1 | HIGH | `branchPrefix` mapping field completely undocumented | `src/config.ts:59` | `latest/configuration/team-repo-mappings.mdx`, `latest/changelog.mdx` | 1/3 | Edited |
| H-2 | HIGH | `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented; custom-image guide leads to workflow failure | `workflows/claude-implement.yml:140-169` | `latest/customize/runner-image.mdx`, `latest/configuration/environment-variables.mdx` | 3/3 | Edited |
| H-3 | HIGH | `AI_IMPLEMENT_RUNNER_LABEL` target-repo variable undocumented | `workflows/claude-implement.yml:179` | `latest/configuration/environment-variables.mdx` | 3/3 | Edited |
| H-4 | HIGH | `AI_IMPLEMENT_LOG_LEVEL` target-repo variable undocumented | `workflows/claude-implement.yml:240-241` | `latest/configuration/environment-variables.mdx` | 3/3 | Edited |
| H-5 | HIGH | Settings panel description omits Global Machine Secrets feature | `src/admin-ui/pages/settings.ts:35` | `latest/reference/admin-ui.mdx` | 1/3 | Edited |
| H-6 | HIGH | Platform → Secrets panel described as functional but is a stub | `src/admin-ui/pages/stubs.ts:76` | `latest/reference/admin-ui.mdx` | 1/3 | Edited |
| M-1 | MEDIUM | Setup guide describes manual CLI sync as primary path; admin UI sync is now preferred | `ai-implement/CLAUDE.md` (workflow templates section) | `latest/setup/target-repo.mdx` | 2/3 | Report |
| M-2 | MEDIUM | Sidebar `<Note>` omits five stub routes visible in the real sidebar | `src/admin-ui/sidebar.ts:16-34` | `latest/reference/admin-ui.mdx` | 3/3 | Report |
| M-3 | MEDIUM | `teamKey` `<ParamField>` body is Linear-only despite multi-provider support | `src/config.ts:24` | `latest/configuration/team-repo-mappings.mdx` | 2/3 | Report |

## Edits applied (HIGH findings)

| Docs file | Findings | Edit source |
|---|---|---|
| `latest/configuration/team-repo-mappings.mdx` | H-1 | run-1 (only option) |
| `latest/changelog.mdx` | H-1 | run-1 (only option) |
| `latest/customize/runner-image.mdx` | H-2 | run-3 (strongest lead line, mentions exact error message, notes both override paths) |
| `latest/configuration/environment-variables.mdx` | H-2, H-3, H-4 | run-1 (H-2 allowlist ParamField, H-3 runner label with nested `<Warning>`), run-2 (H-4 log level — only option) |
| `latest/reference/admin-ui.mdx` | H-5, H-6 | run-2 (only option for both) |

## HIGH-priority finding details

### H-1: `branchPrefix` mapping field completely undocumented  (1/3 runs)

- **Docs**: `latest/configuration/team-repo-mappings.mdx` lists all `RepoMapping` fields through `maxJobMinutes` but has no `branchPrefix` entry; `latest/changelog.mdx` has no entry for this feature.
- **Source**: `src/config.ts:59` — `branchPrefix: string | null` is a fully wired field on `RepoMapping`, persisted in SQLite and forwarded to dispatches as the `branch_prefix` workflow input. Documented in CLAUDE.md under "Per-project branch prefix (admin UI)" as an operator-configurable per-project setting exposed in the admin UI Projects stepper.
- **Edit**: Added `branchPrefix` `<ParamField>` to `latest/configuration/team-repo-mappings.mdx` after the run-caps section, with an inline `<Warning>` about the re-sync requirement and gap-fill exclusion. Added `<Update label="Per-project branch prefix">` entry to `latest/changelog.mdx` as the first entry, with a cross-link to the ParamField anchor. — selected from run-1 (only candidate).
- **Priority agreement**: HIGH in 1 run. Other runs did not surface this finding.

### H-2: `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented; custom-image guide leads to workflow failure  (3/3 runs)

- **Docs**: `latest/customize/runner-image.mdx` shows `ghcr.io/your-org/your-runner:v1` as an example custom image and describes both override mechanisms (`.ai-implement/image.yml` and `AI_IMPLEMENT_RUNNER_IMAGE`) without noting any registry restriction. A reader following the guide for GitHub Actions mode would build and commit a custom image and then see dispatches fail at image validation. The variable `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` is absent from all docs pages.
- **Source**: `workflows/claude-implement.yml:140-169` — the `validate-runner-image` job enforces an allowlist starting with `ghcr.io/builddownai/` and rejects images outside it unless `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` is also set. The same check applies in `comment-trigger.yml`.
- **Edit**: Added `<Warning>` to `latest/customize/runner-image.mdx` (between the Tip and the "Alternative: GitHub Actions variable" section) explaining the default restriction, the "image not allowed" error, and the `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` escape hatch with comma-separated-prefix syntax. Added `<ParamField>` for `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` in the Runner image section of `latest/configuration/environment-variables.mdx`. — runner-image.mdx edit selected from run-3; env-vars ParamField body selected from run-1.
- **Priority agreement**: HIGH in all 3 runs.

### H-3: `AI_IMPLEMENT_RUNNER_LABEL` target-repo variable undocumented  (3/3 runs)

- **Docs**: `latest/configuration/environment-variables.mdx` Target-repo Variables tab lists `AI_IMPLEMENT_RUNNER_IMAGE` but does not mention `AI_IMPLEMENT_RUNNER_LABEL`. CLAUDE.md documents this as a first-class feature under "Runner label (GHA mode)" with a security note about self-hosted runner access.
- **Source**: `workflows/claude-implement.yml:179` — `runs-on: ${{ vars.AI_IMPLEMENT_RUNNER_LABEL || 'ubuntu-latest' }}`. The workflow comments also list it alongside `AI_IMPLEMENT_RUNNER_IMAGE` as an optional variable.
- **Edit**: Added a `### Runner host` subsection with a `<ParamField>` for `AI_IMPLEMENT_RUNNER_LABEL` (default `ubuntu-latest`) in `latest/configuration/environment-variables.mdx`, including a nested `<Warning>` about the security implication of org-variable-controlled runner targeting. — selected from run-1.
- **Priority agreement**: HIGH in 2/3 runs, MEDIUM in 1/3 (run-3). Kept HIGH per no-downgrade rule.

### H-4: `AI_IMPLEMENT_LOG_LEVEL` target-repo variable undocumented  (3/3 runs)

- **Docs**: `latest/configuration/environment-variables.mdx` Target-repo Variables tab does not include `AI_IMPLEMENT_LOG_LEVEL`. CLAUDE.md documents both accepted values (`summary` default, `stream` for per-turn tool-call logging) and the re-sync requirement.
- **Source**: `workflows/claude-implement.yml:240-241` — `AI_IMPLEMENT_LOG_LEVEL: ${{ vars.AI_IMPLEMENT_LOG_LEVEL }}` is passed to the runner entrypoint. Also present in `comment-trigger.yml`.
- **Edit**: Added `<ParamField>` for `AI_IMPLEMENT_LOG_LEVEL` (default `summary`) in `latest/configuration/environment-variables.mdx` under the Runner host section, with a bullet list describing both accepted values and the re-sync requirement. — selected from run-2 (only run that produced an MDX edit for this variable).
- **Priority agreement**: HIGH in 1/3 runs (run-2), MEDIUM in 2/3 runs (runs 1 and 3). Kept HIGH per no-downgrade rule.

### H-5: Settings panel description omits Global Machine Secrets feature  (1/3 runs)

- **Docs**: `latest/reference/admin-ui.mdx` Settings section lists only `flySessionsApp` and `flySessionsRegion` as managed settings.
- **Source**: `src/admin-ui/pages/settings.ts:35` — the Settings page includes a full "Global Machine Secrets" card with list/add/delete UI for Fly secrets injected into every session machine.
- **Edit**: Expanded the Settings section in `latest/reference/admin-ui.mdx` to document the Global Machine Secrets card — what it manages, that values are write-only, typical use case, and a `<Warning>` about the naming restriction on globally-scoped secrets. — selected from run-2 (only option).
- **Priority agreement**: HIGH in 1 run. Other runs did not surface this finding.

### H-6: Platform → Secrets panel described as functional but is a stub  (1/3 runs)

- **Docs**: `latest/reference/admin-ui.mdx` "### Secrets" section under Platform presents per-team and global secrets management (API endpoints, storage details) as if they live in a navigable Secrets panel, with no indication the panel itself is not implemented.
- **Source**: `src/admin-ui/pages/stubs.ts:76` — the `secrets` route is a "Coming soon" stub; actual secrets management is in Settings (global secrets) and the Projects panel per-row Secrets button (per-team secrets).
- **Edit**: Added a `<Note>` at the top of the Secrets section stating the dedicated panel is not yet available, with pointers to the Settings panel (global secrets) and Configure → Projects rows (per-team secrets). Updated the Global secrets paragraph to cross-reference the Settings panel. API endpoint and warning documentation preserved. Also updated the Secrets → Global secrets paragraph to reference the Settings panel → Global Machine Secrets card. — selected from run-2 (only option).
- **Priority agreement**: HIGH in 1 run. Other runs did not surface this finding.

## MEDIUM-priority finding details

### M-1: Setup guide describes manual CLI sync as primary path; admin UI sync is now preferred  (2/3 runs)

- **Docs**: `latest/setup/target-repo.mdx` Step 2 instructs operators to run `gh workflow run sync-workflow.yml -f target_repo=...` from the AI-Implement CLI as the primary method to sync workflow files into a target repo, with the mapping created only later (Step 6).
- **Source**: `ai-implement/CLAUDE.md` (workflow templates section) explicitly states "`.github/workflows/sync-workflow.yml` remains as a **manual fallback**, but normal distribution should happen from the orchestrator." CLAUDE.md's onboarding steps put mapping creation first (Step 1), then Click **Sync workflows** (Step 3) — the opposite order from the setup guide.

### M-2: Sidebar `<Note>` omits five stub routes visible in the real sidebar  (3/3 runs)

- **Docs**: `latest/reference/admin-ui.mdx:12-21` `<Note>` lists Configure as "Projects, Pipelines & steps, Models & providers" (3 items) and Developer as "Audit log, Customizations" (2 items).
- **Source**: `src/admin-ui/sidebar.ts:16-34` — Configure has 5 items (Projects, Pipelines & steps, Models & providers, Triggers & channels, Policies & risk) and Developer has 5 items (MCP server, Webhooks, Audit log, Customizations, Updates). The five unlisted routes are rendered stubs with "Coming soon" badges (`src/admin-ui/pages/stubs.ts`). An operator seeing them in the sidebar and searching the docs will find no mention.

### M-3: `teamKey` `<ParamField>` body is Linear-only despite multi-provider support  (2/3 runs)

- **Docs**: `latest/configuration/team-repo-mappings.mdx:19-21` — the `<ParamField body="teamKey">` body says "Your Linear team identifier…the orchestrator uses this key to match issues polled from Linear." This framing is misleading for Jira operators; the surrounding `<Tip>` at line 13 does note the Jira case, but the field body itself does not.
- **Source**: `src/config.ts:24` — `teamKey` is the primary key for all mappings regardless of `ticketingProvider`; for Jira it is an arbitrary operator-chosen label with no ticketing-system counterpart.

## LOW-priority findings (aggregate, numbered as L-1 … L-7)

- **L-1** — `latest/customize/workflow-md.mdx` describes the `setup:` front-matter key as running "before Claude starts" but does not note that a non-zero exit from the setup script aborts the run early — a useful fact for operators troubleshooting stalled dispatches (`ai-implement/CLAUDE.md` workflow templates section). (Run 1)
- **L-2** — `latest/customize/runner-image.mdx` documents `:latest` and `:next` channel tags but does not mention dated channel tags (`base-<channel>-vYYYYMMDD-<sha>`), which are the recommended mechanism for pinning a specific build without relying on mutable tags (`ai-implement/CLAUDE.md` runner image channels section). (Run 1)
- **L-3** — `latest/configuration/team-repo-mappings.mdx` states `maxTurns` default as `50`, but `src/config.ts` has `maxTurns: number | null` where `null` means "use Claude's built-in default" — the `50` is the admin UI blank-field sentinel, not a server-enforced hard cap. (Run 2)
- **L-4** — `workflows/claude-implement.yml:4-10` header comment still lists `ANTHROPIC_API_KEY` as a "Required secrets" entry, though Bedrock and OAuth paths make it optional in practice. No operator-visible impact since step-level validation is correct. (Run 2)
- **L-5** — `reference/labels.mdx:114` (and comment-trigger docs) state the `/ai-implement` trigger requires "write access" but `comment-trigger.yml:71-73` accepts `write`, `maintain`, or `admin` roles. (Run 3)
- **L-6** — `REAPER_ALERT_THRESHOLD` has no documented default in `latest/configuration/environment-variables.mdx`; `.env.example:112` shows `10` as the default value. (Run 3)
- **L-7** — `latest/providers/aws-bedrock.mdx` states credentials are configured "twice per run (implementation + gap analysis step)" but `claude-implement.yml:208-213` shows a single `configure-aws-credentials` step with a 4-hour session duration covering both steps in the same job. (Run 3)

## Synthesis decisions log

### H-1

- **Present in:** Run 1 HIGH
- **Dedup reasoning:** Only Run 1 surfaced the branchPrefix drift. Runs 2 and 3 did not flag the missing `branchPrefix` field.
- **Selected edit from:** Run 1 (only option — no selection needed).

### H-2

- **Present in:** Run 1 HIGH, Run 2 HIGH, Run 3 HIGH
- **Dedup reasoning:** All three runs identify the same source behavior — the `validate-runner-image` job at `workflows/claude-implement.yml:140-169` rejects images outside `ghcr.io/builddownai/` unless `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` is set. Titles differ ("GHA allowlist blocks custom runner images," "custom-image guide leads to workflow failure," "Custom runner image prefix allowlist undocumented") but all cite the same source behavior and target the same docs pages.
- **Priority resolution:** (omitted — all runs agreed HIGH)
- **Selected edit from:** Run 3 for `runner-image.mdx`; Run 1 for the `environment-variables.mdx` `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` ParamField body.
- **Selection reasoning:** Run 3's `<Warning>` leads with "Images from registries other than `ghcr.io/builddownai/` are blocked by default" — the most scannable first line — and uniquely mentions the exact "image not allowed" error text and that the variable applies to both override paths. Run 1's env-vars ParamField body explicitly notes "Fly Machines mode has no equivalent allowlist," useful context for operators working across execution modes. Run 2's `## Allowed image registries` section is positioned after the Alternative section (too late for readers following the primary Setup steps path) and references the workflow filename in prose.

**Edit candidates compared (runner-image.mdx):**

Run 3's approach (selected):
```mdx
{/* AUDIT H-2: ... */}
<Warning>
  Images from registries other than `ghcr.io/builddownai/` are blocked by default.

  The workflow validates the resolved image against an allowlist before booting the container.
  The default allowlist contains only `ghcr.io/builddownai/` — images from any other registry,
  including your own organization's GHCR namespace, fail this check with an "image not allowed" error.

  To allow additional registries, set `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` as a
  repository or organization variable (Settings → Secrets and variables → Actions → Variables).
  Use a comma-separated list of registry prefixes:

  ```
  AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES=ghcr.io/your-org/
  ```

  Multiple prefixes: `ghcr.io/org-a/,ghcr.io/org-b/`. This variable applies to both the
  `.ai-implement/image.yml` path and the `AI_IMPLEMENT_RUNNER_IMAGE` variable path.
</Warning>
```

Run 1's approach (not selected for runner-image.mdx):
```mdx
{/* AUDIT H-2: ... */}
<Warning>
  **GitHub Actions mode only:** The synced workflow validates that the resolved runner image
  begins with an allowed registry prefix. By default only `ghcr.io/builddownai/` images pass
  this check — including images forwarded from `.ai-implement/image.yml`.

  To use a custom image from your own registry, set the `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`
  repository or organization variable to a comma-separated list of additional allowed prefixes:

  ```
  AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES=ghcr.io/your-org/
  ```

  Without this, any image outside `ghcr.io/builddownai/` causes the workflow to fail at image
  validation. Fly Machines mode has no equivalent check.
</Warning>
```

Run 2's approach (not selected — adds `## Allowed image registries` section after Alternative section, placed too late in the page for readers following the primary Setup steps path):
```mdx
{/* AUDIT H-3: ... */}
## Allowed image registries

The synced `claude-implement.yml` validates the resolved runner image before the job starts.
By default only images hosted at `ghcr.io/builddownai/` are allowed...
[... 20 more lines truncated for brevity ...]
```

### H-3

- **Present in:** Run 1 HIGH, Run 2 HIGH, Run 3 MEDIUM
- **Dedup reasoning:** All three runs cite `workflows/claude-implement.yml:179` and target `latest/configuration/environment-variables.mdx`. Same drift, same docs surface, different priority ratings.
- **Priority resolution:** Kept HIGH per no-downgrade rule; Run 3 rated this MEDIUM.
- **Selected edit from:** Run 1
- **Selection reasoning:** Run 1 uses a nested `<Warning>` inside the `<ParamField>` for the security implication (self-hosted runner access to secrets) — the correct semantic component for a blocking security concern. Run 1 also introduces a `### Runner host` subsection that distinguishes runner-image variables from runner-host variables, improving navigation. Run 2 handles the security note in prose without a semantic Warning component.

**Edit candidates compared:**

Run 1's approach (selected):
```mdx
### Runner host

{/* AUDIT H-3: AI_IMPLEMENT_RUNNER_LABEL undocumented */}
<ParamField body="AI_IMPLEMENT_RUNNER_LABEL" type="string" default="ubuntu-latest">
  GitHub Actions runner label for the implementation job. Default GitHub-hosted runners are
  2 vCPU; switching to a larger runner (for example `ubuntu-latest-4-cores`) roughly halves
  wall-clock time for repos whose test suites are CPU-bound.

  Set as a repository or organization variable (Settings → Secrets and variables → Actions →
  Variables), the same way as `AI_IMPLEMENT_PROVIDER`. Takes effect after the target repo has
  re-synced `claude-implement.yml`.

  <Warning>
    Write access to this variable lets the holder redirect the implementation job to an
    arbitrary runner, including self-hosted runners that can access the job's secrets. Treat
    it with the same trust model as any org variable that controls the runner host.
  </Warning>
</ParamField>
```

Run 2's approach (not selected — security note in prose, no semantic `<Warning>`, no `### Runner host` subsection):
```mdx
<ParamField body="AI_IMPLEMENT_RUNNER_LABEL" type="string" default="ubuntu-latest">
  GitHub Actions `runs-on` label for the implement job. Defaults to `ubuntu-latest`.

  Claude's test suites during an implement pass are CPU-bound and scale roughly linearly with
  available cores. Pointing this at a larger-runner label (e.g. `ubuntu-latest-4-cores`)
  roughly halves the implement job's wall-clock time on most repos.

  Only affects GitHub Actions execution mode. Granting write access to this variable lets the
  holder redirect the job to an arbitrary runner (including self-hosted), which would see the
  job's secrets — treat it with the same care as any org-level `runs-on` override.
</ParamField>
```

### H-4

- **Present in:** Run 2 HIGH, Run 1 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** All three runs cite the same `workflows/claude-implement.yml:240-241` line and target `latest/configuration/environment-variables.mdx`. Same drift. Run 2 rated HIGH; Runs 1 and 3 rated MEDIUM.
- **Priority resolution:** Kept HIGH per no-downgrade rule; Runs 1 and 3 rated this MEDIUM.
- **Selected edit from:** Run 2 (only run that produced an MDX edit for this variable — Runs 1 and 3 treated it as MEDIUM/report-only).

### H-5

- **Present in:** Run 2 HIGH
- **Dedup reasoning:** Only Run 2 surfaced the Global Machine Secrets omission from the Settings section description. No other run mentioned it.
- **Selected edit from:** Run 2 (only option — no selection needed).

### H-6

- **Present in:** Run 2 HIGH
- **Dedup reasoning:** Only Run 2 identified the Secrets panel as a stub being described as functional. No other run surfaced this finding (Run 1's L-1 mentions sidebar items count, which is related but distinct — it targets the sidebar Note, not the Secrets panel prose).
- **Selected edit from:** Run 2 (only option — no selection needed).

### M-1

- **Present in:** Run 1 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** Both runs cite `ai-implement/CLAUDE.md` (workflow templates / sync guidance section) and target `latest/setup/target-repo.mdx`. Both describe the same drift: setup guide presents `sync-workflow.yml` CLI as the primary sync method when CLAUDE.md explicitly calls it a fallback. Run 2 did not surface this finding.

### M-2

- **Present in:** Run 1 LOW, Run 2 MEDIUM (×2 — M-2 Configure + M-3 Developer), Run 3 MEDIUM
- **Dedup reasoning:** All three runs describe the same drift — the `<Note>` in `latest/reference/admin-ui.mdx` lists 3 Configure items and 2 Developer items while the actual sidebar has 5 each (with five additional stub routes). Run 2 split this into two findings (Configure sidebar vs. Developer sidebar) but both target the same `<Note>` block. Consolidated into one finding.
- **Priority resolution:** Kept MEDIUM per no-downgrade rule; Run 1 rated this LOW.

### M-3

- **Present in:** Run 2 MEDIUM, Run 3 LOW
- **Dedup reasoning:** Both cite `src/config.ts:24` and target the `teamKey` `<ParamField>` in `latest/configuration/team-repo-mappings.mdx`. Both describe the same issue: the field body is written as "Your Linear team identifier" without noting that for Jira mappings `teamKey` is an arbitrary operator-chosen label. Run 2 rated MEDIUM; Run 3 rated LOW (as L-4).
- **Priority resolution:** Kept MEDIUM per no-downgrade rule; Run 3 rated this LOW. Note: the surrounding `<Tip>` at the top of the page already explains the Jira case, which is why Run 3 rated it LOW — but the field body itself still misleads Jira operators who read field descriptions without the Tip.
