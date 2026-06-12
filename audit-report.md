# AI-Implement docs audit (multi-run synthesis, N=3) — main branch — 2026-06-12

Synthesized from 3 independent audit runs against the same source+docs state.
Recurrence column shows how many runs flagged each finding.

## Summary

| Priority | Count | Action |
|---|---|---|
| HIGH | 3 | Edits applied |
| MEDIUM | 4 | Report-only |
| LOW | 5 | Aggregate count |

## Findings at a glance

| # | Priority | Title | Source | Docs file | Runs | Status |
|---|---|---|---|---|---|---|
| H-1 | HIGH | Secrets panel is a stub; per-team secrets are on Projects page, global secrets on Settings page | `src/admin-ui/pages/stubs.ts:79-86`, `src/admin-ui/pages/projects.ts:241`, `src/admin-ui/pages/settings.ts:35-57` | `reference/admin-ui.mdx` | 1/3 | Edited |
| H-2 | HIGH | `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented for GitHub Actions mode | `workflows/claude-implement.yml:87-100`, `workflows/comment-trigger.yml:107-115` | `customize/runner-image.mdx` | 2/3 | Edited |
| H-3 | HIGH | Concurrency cap counts planning-phase issues as well as implementation-phase issues | `src/providers/linear.ts:178-179`, `src/providers/jira.ts:171` | `reference/labels.mdx` | 1/3 | Edited |
| M-1 | MEDIUM | Admin UI sidebar Note omits five stub panels visible to users | `src/admin-ui/sidebar.ts:14-34` | `reference/admin-ui.mdx` | 3/3 | Report |
| M-2 | MEDIUM | `teamKey` ParamField description is Linear-only; no guidance for Jira operators | `src/config.ts:22-52` | `configuration/team-repo-mappings.mdx` | 2/3 | Report |
| M-3 | MEDIUM | Setup guide presents manual `sync-workflow.yml` dispatch as primary method; admin UI Sync button is the designed path | `ai-implement/CLAUDE.md:150` | `setup/target-repo.mdx` | 3/3 | Report |
| M-4 | MEDIUM | Comment trigger accepts `maintain` and `admin` roles, not just `write` | `workflows/comment-trigger.yml:61` | `reference/labels.mdx` | 2/3 | Report |

## Edits applied (HIGH findings)

| Docs file | Findings | Edit source |
|---|---|---|
| `reference/admin-ui.mdx` | H-1 | run-1 (only candidate) |
| `customize/runner-image.mdx` | H-2 | run-3 (semantically stronger Warning; variable table more scannable) |
| `reference/labels.mdx` | H-3 | run-3 (only candidate) |

## HIGH-priority finding details

### H-1: Secrets panel is a stub; per-team and global secrets are in different panels  (1/3 runs)

- **Docs**: `reference/admin-ui.mdx` — Platform → Secrets section presents per-team and global secrets as if they live in a dedicated, functional Secrets panel; Platform → Settings section describes only `flySessionsApp` and `flySessionsRegion`, omitting the Global Machine Secrets UI that lives there.
- **Source**: `src/admin-ui/pages/stubs.ts:79-86` — the Secrets sidebar item is a "Partially implemented" stub that redirects users to Projects and Settings; `src/admin-ui/pages/projects.ts:241` — per-team secrets are accessible via a **Secrets** button on each project row; `src/admin-ui/pages/settings.ts:35-57` — global machine secrets have a full UI section on the Settings page.
- **Edit**: Replaced the Secrets section to direct readers to Projects (per-team) and Settings (global), noting the standalone panel is under development. Added a Global Machine Secrets description to the Settings section — selected from run-1 (only candidate).
- **Priority agreement**: HIGH in 1/3 runs (run-1 only).

### H-2: `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` undocumented for GitHub Actions mode  (2/3 runs)

- **Docs**: `customize/runner-image.mdx` — the closing Note stated "GitHub Actions runs use the `ubuntu-latest` runner provided by GitHub and are not affected by `.ai-implement/image.yml`," which is incorrect: GHA runs also use a container image with its own override mechanism.
- **Source**: `workflows/claude-implement.yml:87-100` — reads `vars.AI_IMPLEMENT_RUNNER_IMAGE` and `vars.AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`; enforces a prefix allowlist, failing fast if the image is from a disallowed registry; `workflows/comment-trigger.yml:107-115` — mirrors the same pattern.
- **Edit**: Corrected the Note to remove the false GHA claim; added a new "Custom runner image for GitHub Actions mode" section with a variable reference table, setup guidance, and a Warning about the prefix allowlist — selected from run-3 of 2 candidates.
- **Priority agreement**: HIGH in 2/3 runs (run-2 as H-1, run-3 as H-2).

### H-3: Concurrency cap counts planning-phase issues, not just implementation-phase  (1/3 runs)

- **Docs**: `reference/labels.mdx` — Concurrency section stated `maxInProgressAiIssues` controls issues in flight with `AI-Working` (Linear) or `Implementing` (Jira), implying only implementation-phase issues consume capacity slots.
- **Source**: `src/providers/linear.ts:178-179` — `if (labelNames.has("AI-Working") || labelNames.has("AI-Planning"))` increments the in-progress count; `src/providers/jira.ts:171` — capacity JQL is `in (Planning, Implementing)`. Planning-phase issues consume capacity alongside implementation-phase issues.
- **Edit**: Replaced the single-line concurrency description with a paragraph and a two-item list making clear that both phases consume capacity, with a diagnostic hint for operators seeing unexpected throttling — selected from run-3 (only candidate).
- **Priority agreement**: HIGH in 1/3 runs (run-3 only).

## MEDIUM-priority finding details

### M-1: Admin UI sidebar Note omits five stub panels visible to users  (3/3 runs)

- **Docs**: `reference/admin-ui.mdx:8-17` — the introductory Note lists the Configure group as "Projects, Pipelines & steps, Models & providers" (3 items) and Developer as "Audit log, Customizations" (2 items).
- **Source**: `src/admin-ui/sidebar.ts:14-34` — Configure also includes "Triggers & channels" (`channels`) and "Policies & risk" (`policies`); Developer also includes "MCP server" (`mcp`), "Webhooks" (`webhooks`), and "Updates" (`updates`). All five are stub/roadmap placeholders visible in the sidebar.

### M-2: `teamKey` ParamField description is Linear-only; no guidance for Jira operators  (2/3 runs)

- **Docs**: `configuration/team-repo-mappings.mdx:15-17` — ParamField body says "Your Linear team identifier (e.g. `ENG`). This is the short prefix shown in Linear issue identifiers." A Jira operator reading this receives no guidance on what to enter.
- **Source**: `src/config.ts:22-52` — `team_key TEXT PRIMARY KEY` is a free-form string with no provider constraint; for Jira mappings the key is an arbitrary operator-chosen label with no Linear team concept involved.

### M-3: Setup guide presents manual `sync-workflow.yml` dispatch as the primary path  (3/3 runs)

- **Docs**: `setup/target-repo.mdx:13-35` — Step 2 instructs operators to run `gh workflow run sync-workflow.yml -f target_repo=your-org/your-repo` from the AI-Implement repository as the sync step.
- **Source**: `ai-implement/CLAUDE.md:150` — "`.github/workflows/sync-workflow.yml` remains as a manual fallback, but normal distribution should happen from the orchestrator." The **Sync workflows** button on each Projects row is the designed primary path.

### M-4: Comment trigger accepts `maintain` and `admin` roles, docs say "write access" only  (2/3 runs)

- **Docs**: `reference/labels.mdx:110` — "The comment must be exactly `/ai-implement` (no extra text) and must be posted by a collaborator with write access to the repo."
- **Source**: `workflows/comment-trigger.yml:61` — permission check accepts `["write", "maintain", "admin"]`; `maintain`- or `admin`-role collaborators who read the docs would not know their comment is authorized.
- **Priority agreement**: MEDIUM in 1/3 runs (run-2); LOW in 1/3 runs (run-3). Kept MEDIUM per no-downgrade rule.

## LOW-priority findings (aggregate, numbered as L-1 … L-5)

- **L-1** — `claude-implement.yml` defaults the GHA container image to `:next` (`ghcr.io/builddownai/ai-implement-runner:next`) while `comment-trigger.yml` defaults to `:latest` — tag inconsistency between the two synced workflow files (`workflows/claude-implement.yml:100` vs `workflows/comment-trigger.yml:115`). (2/3 runs)
- **L-2** — `REAPER_ALERT_THRESHOLD` has no default shown in `configuration/environment-variables.mdx`; `.env.example:117` shows the default is `10`. (1/3 runs)
- **L-3** — `reference/admin-ui.mdx` Customizations Note contains the developer-facing sentence "The team has it tracked to restore as a hard fail" — internal tracking metadata that reads awkwardly in operator docs. (1/3 runs)
- **L-4** — `reference/labels.mdx` Jira Sessions note says destroying a session "removes the in-progress marker" but omits that the status transitions to `Plan Approved` rather than `Ready` (`src/providers/jira.ts:251-254`), meaning the next poll dispatches implementation only (planning is not re-run). (1/3 runs)
- **L-5** — `PATCH /api/mappings/:teamKey` rejects requests that include both `maxInProgressAiIssues` and `paused` in the same body, but the API reference table does not document this constraint (`src/admin.ts:595-599`). (1/3 runs)

---

## Synthesis decisions log

### H-1

- **Present in:** Run 1 HIGH
- **Dedup reasoning:** Only run-1 surfaced this finding. Run-2 and run-3 did not audit the Secrets/Settings panel area.
- **Selected edit from:** Run 1 (only candidate — no selection needed).

### H-2

- **Present in:** Run 2 HIGH, Run 3 HIGH
- **Dedup reasoning:** Both runs cite the same source behavior (`workflows/claude-implement.yml` reads `AI_IMPLEMENT_RUNNER_IMAGE` and `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`; the closing Note in `customize/runner-image.mdx` falsely claims GHA uses `ubuntu-latest`) and target the same docs surface. Run-2 titled it "GHA runs use a container image, not ubuntu-latest"; run-3 titled it "AI_IMPLEMENT_RUNNER_IMAGE and AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES are undocumented for GHA mode." Same drift.
- **Priority resolution:** Both runs rated this HIGH — no conflict.
- **Selected edit from:** Run 3
- **Selection reasoning:** Run-3's `<Warning>` for the prefix allowlist is semantically correct (runs fail if disallowed — a blocking constraint), whereas run-2 used `<Tip>` which is semantically wrong for a blocking error. Run-3's variable table is also more scannable. Both approaches correct the false ubuntu-latest claim.

**Edit candidates compared:**

Run 2's approach (not selected):
```mdx
## Fly Machines execution mode

Create a file at `.ai-implement/image.yml` in the default branch of your target repo:
...

## GitHub Actions execution mode

When a repo mapping uses **GitHub Actions** execution, Claude still runs inside a container image — not a bare `ubuntu-latest` job. The default is the same base image as Fly Machines: `ghcr.io/builddownai/ai-implement-runner:latest`.

To override the container image, set the **`AI_IMPLEMENT_RUNNER_IMAGE`** repository variable ...

<Tip>
  By default, only images in the `ghcr.io/builddownai/` namespace are allowed. To permit images from a different registry or namespace, set the **`AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES`** ...
</Tip>
```

Run 3's approach (selected):
```mdx
<Note>
  The `.ai-implement/image.yml` file only controls the image used for **fly-machines** sessions. For GitHub Actions execution mode, see the section below.
</Note>

...

## Custom runner image for GitHub Actions mode

When your target repo uses the **github-actions** execution mode, AI-Implement runs Claude inside the `ai-implement-runner` container image on each workflow run. You can override the default image using two GitHub Actions repository (or organization) variables.

| Variable | Effect |
|---|---|
| `AI_IMPLEMENT_RUNNER_IMAGE` | Overrides the default container image for all runs in this repo or org |
| `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` | Comma-separated registry prefixes allowed in addition to `ghcr.io/builddownai/` |

<Warning>
  By default, only images from `ghcr.io/builddownai/` are allowed. If your custom image is hosted under a different registry or namespace, add its prefix via `AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` ...
</Warning>
```

### H-3

- **Present in:** Run 3 HIGH
- **Dedup reasoning:** Only run-3 surfaced this finding. Run-1 and run-2 did not flag the concurrency section.
- **Selected edit from:** Run 3 (only candidate — no selection needed).

### M-1

- **Present in:** Run 1 MEDIUM, Run 2 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** All three runs cite `src/admin-ui/sidebar.ts` and flag the same five missing items (Triggers & channels, Policies & risk, MCP server, Webhooks, Updates) from the same introductory Note in `reference/admin-ui.mdx`. Identical drift in all three runs.

### M-2

- **Present in:** Run 1 MEDIUM, Run 3 MEDIUM
- **Dedup reasoning:** Both runs cite the same `<ParamField body="teamKey">` in `configuration/team-repo-mappings.mdx` and the same underlying issue (the field body is written for Linear team identifiers only, leaving Jira operators without guidance). Run-1 cited `src/config.ts:24-52` and `src/admin.ts:104-115`; run-3 cited `src/config.ts:22`. Same drift.

### M-3

- **Present in:** Run 1 MEDIUM, Run 2 MEDIUM, Run 3 LOW
- **Dedup reasoning:** All three runs identify that `setup/target-repo.mdx` Step 2 instructs the manual `gh workflow run sync-workflow.yml` dispatch while the orchestrator Sync button is now the designed primary path. Run-3 classified it LOW; run-1 and run-2 classified it MEDIUM. Kept MEDIUM per no-downgrade rule.
- **Priority resolution:** Kept MEDIUM; Run 3 rated this LOW. MEDIUM wins per no-downgrade rule.

### M-4

- **Present in:** Run 2 MEDIUM, Run 3 LOW
- **Dedup reasoning:** Both runs cite `workflows/comment-trigger.yml:61` and flag that the permission check accepts `["write", "maintain", "admin"]` while the docs say "write access." Same code line, same docs sentence.
- **Priority resolution:** Kept MEDIUM; Run 3 rated this LOW. MEDIUM wins per no-downgrade rule.
