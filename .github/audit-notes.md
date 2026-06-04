# Audit Prompt Development Notes

Working log of prompt-design decisions, test runs, and operational lessons. Finalized as the mentor-handoff artifact at block close (Step 7).

## Operational fixes applied to `audit.yml` during testing

These are infrastructure changes layered onto the original Step 1 workflow YAML during Step 4 testing. Each is worth surfacing to the mentor at handoff.

- **`actions/checkout` v4 → v5 and `actions/create-github-app-token` v1 → v3** — Node 20 deprecation; June 16, 2026 forced-switch deadline. Required version bumps before that date.
- **`app-id` → `client-id`** on `create-github-app-token@v3`. The v3 major-version rename. Now references a new secret `DOCS_AUDIT_APP_CLIENT_ID` instead of the legacy `DOCS_AUDIT_APP_ID` (App ID and Client ID are different values on a GitHub App; the latter is the string `Iv23li...`-form identifier).
- **`timeout-minutes: 3` on Install Claude Code CLI** — npm postinstall on `@anthropic-ai/claude-code` can hang silently on Linux runners while downloading the platform-native binary (known issue, [#5209](https://github.com/anthropics/claude-code/issues/5209)). Three-minute cap means a future hang fails fast.
- **`claude --version` verification** appended to the Install step. Confirms the binary landed on PATH and is callable. If postinstall silently breaks, this errors with a clear message.
- **`timeout-minutes: 25` on Run audit** — bounds a hung run (network issue, OAuth token retry loop, etc.) so the job doesn't sit for the default 360 minutes.
- **`continue-on-error: true` on Run audit** — allows the `peter-evans/create-pull-request` step to run even when Claude exhausts `--max-turns`. Partial output beats no output for diagnostic value.
- **`--permission-mode acceptEdits` on the `claude -p` invocation** — required for non-interactive runs. Without it, file writes (Edit, Write tools) trigger permission prompts that have nowhere to be answered in `-p` mode, so Claude completes the audit mentally but never writes `audit-report.md` or applies edits. `acceptEdits` auto-approves edits within the working directory while still respecting protected paths (`.git`, `.claude`, etc.) — safer than `--dangerously-skip-permissions`.

## Reminder for production handoff

Before transferring App ownership to BuildDownAI:

- The `schedule:` cron (`0 14 * * 1`) is currently active on the user's fork's `main`. Verify it's intended behavior on production before transfer, or disable temporarily.
- Three workflow YAML fields must change from dev to production values:
  - `owner: rigonzal530` → `owner: BuildDownAI`
  - `repository: rigonzal530/AI-Implement` → `repository: BuildDownAI/AI-Implement`
  - `base: june-doc-update` → `base: main`
- BuildDownAI should rotate the App's private key after taking ownership so the dev-phase key (held locally) becomes inert.
- The `DOCS_AUDIT_APP_CLIENT_ID`, `DOCS_AUDIT_APP_PRIVATE_KEY`, and `CLAUDE_CODE_OAUTH_TOKEN` secrets need to be set on `BuildDownAI/docs` (not just the fork).

## Run history

### Run 1 — `main` branch, prompt v1, `--max-turns 30`

- **Outcome:** Hit `--max-turns` at 9m 7s. Exit code 1. peter-evans skipped (subsequent steps didn't run on step failure). **No PR opened.**
- **Tokens / cost:** Subscription (OAuth token); no per-token cost. Wall time 9m 7s.
- **Turn efficiency:** ~18s per turn average.
- **Diagnostic value:** Limited — no PR, no artifacts. Tells us 30 turns is insufficient for `main`-scope audit with the current prompt's 7-area survey + edit + report breadth.

### Run 2 — `main` branch, prompt v1, `--max-turns 50`

- **Outcome:** Audit completed mentally at 10m 20s, **well under the 50-turn cap**. But Claude could not write `audit-report.md` or apply edits — `-p` mode's permission prompts had nowhere to be answered. peter-evans then failed: `File 'audit-report.md' does not exist`.
- **Diagnostic value:** Very high — Claude shared the complete audit transcript in stdout. The prompt is mostly working; only the I/O surface was blocked.
- **Findings produced (held in transcript only, not committed):**
  - **H-1:** Platform → Secrets panel documented as if fully implemented (`reference/admin-ui.mdx:234-252`), but `src/admin-ui/pages/stubs.ts:75-86` renders a "coming soon" stub. Real secrets management lives in Settings (global) and Projects (per-team). Draft `<Note>` proposed.
  - **M-1:** `Settings` panel docs omit Global Machine Secrets card (`src/admin-ui/pages/settings.ts:34-56` vs `reference/admin-ui.mdx:254-264`).
  - **M-2:** Configure sidebar group listing in `reference/admin-ui.mdx:9-17` lists 3 items; `src/admin-ui/sidebar.ts:18-20` has 5 (missing Triggers & channels, Policies & risk stubs).
  - **M-3:** Developer sidebar group listing similarly incomplete (missing MCP server, Webhooks, Updates stubs at `stubs.ts:87-113`).
  - **M-4:** `setup/target-repo.mdx:13-35` directs users to `gh workflow run sync-workflow.yml` as primary sync path; `CLAUDE.md` says admin UI Sync workflows button should be preferred.
  - **LOW (3 aggregate):** stepper walkthrough combines steps 0/1/2; "Pipelines (jobs)" heading vs "Pipelines" sidebar label; `ensureTeamLabel()` developer-only.
- **Quality assessment:** Strong v1 output. Categorization is correct (HIGH for active-misleading docs; MEDIUM for coverage gaps; LOW for cosmetic). Citations are real and precise. The proposed MDX edit for H-1 uses the right component (`<Note>`) and the correct `{/* AUDIT H-1: */}` marker convention.
- **Fix applied:** Added `--permission-mode acceptEdits` to the Claude invocation.

### Run 3 — `main` branch, prompt v1, `--max-turns 50` + `acceptEdits` (next)

_Expected: PR opens with applied H-1 edit + audit-report.md as PR body. Use as the real Step 5 iteration baseline._

## Cost / rate calibration (for mentor)

Calibration data from Run 2 (50 turns, `main` branch):
- **Wall time:** 10m 20s (matches Run 1's ~18s/turn; turn count was well under cap)
- **Turn cost:** ~12s/turn average at this scope (improved over Run 1's 18s/turn — likely because Claude found the work fit comfortably and didn't need to backtrack)
- **Cost lever assessment:** `--max-turns 50` is comfortable headroom for `main`. The constraint that mattered was not turn budget but I/O permissions. **Prompt efficiency tuning isn't urgent** — the prompt already fits within budget. Save efficiency work for if `testing` runs exceed 50.
- **OAuth-token usage:** Subscription-tier; one `main`-branch audit is a small fraction of weekly quota. `testing`-branch (102 files, +9.5k LOC) likely 2–3× this run; still bounded.

## Prompt iteration history

- **v1** (current): 5-layer structure (role, inputs, repo context, task, quality checklist). Reference file grounds finding shape via three worked examples (removal, addition, refactor). HIGH-only edits with `{/* AUDIT H-<n>: */}` markers.
  - **Run 2 evidence:** prompt produces high-quality, well-cited findings. Categorization is sharp. The reference file's worked examples successfully shaped the output. Strong baseline.
  - **Known unknowns to test in Run 3 onward:** does the prompt over-produce MEDIUM findings on `testing`-branch breadth? Does the surgical-edit policy hold when 5+ HIGH findings need edits? Are MDX comment markers actually useful for human reviewers in a real PR review experience?

## Open questions for mentor / future-self

- Is 50 turns the right budget, or do we need 75+ for `testing`-branch breadth?
- Does the prompt over-explore (read too many files per finding) or under-explore (miss areas)?
- Should the prompt explicitly tell Claude to batch tool calls?
- For findings beyond a count cap (e.g. >15 HIGH findings), should the prompt split into multiple PRs vs. accept one large PR?
- Are MDX comment markers (`{/* AUDIT H-<n>: */}`) the right review-aid, or should we use a different convention?

## How to update this prompt later (5-step runbook)

_Will draft in Step 7 once iteration patterns are clearer._
