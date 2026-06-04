# Audit Prompt Development Notes

The mentor-handoff artifact for the docs-audit automation. Top section is the relay summary. Detailed sections below are reference material captured during the Block 4 dev cycle.

## Block summary (mentor handoff)

### What landed

- **Workflow** at `.github/workflows/audit.yml` — scheduled cron (Mondays 14:00 UTC) + manual dispatch with `branch` (main/testing) and `max_turns` (override) inputs. Mints a GitHub App installation token, checks out both repos, runs the Claude CLI audit, opens a draft PR via peter-evans.
- **Prompt** at `.github/audit-prompt.md` (v1.1) — 5-layer structure: role + permission gates, inputs, repo context, task instructions, quality checklist.
- **Reference** at `.github/audit-reference.md` — priority rubric + 3 worked HIGH examples (removal, addition, refactor) + 1 worked LOW-format example + anti-pattern list + areas-of-common-drift hints.
- **Tested** end-to-end against AI-Implement `main`: CI Run 3 opened a draft PR with applied HIGH edit + structured audit report. Local Run 4 reproduced clean output with the v1.1 reference.

### Key insights worth relaying

1. **Run the audit multiple times per cycle for real drift coverage.** Across three runs of the same prompt on the same source state, all three HIGH findings were different — and all three were real, substantive drift issues. A single-run audit consistently misses real findings other runs would catch. Operational implication: scheduled cron runs once weekly, but a thorough audit (e.g., pre-release) should dispatch 2–3 times manually and union the findings. Costs more API quota; yields meaningfully better coverage.

2. **Default `--max-turns` calibrated to 75 (main) / 100 (testing).** Workflow's hybrid resolver applies per-branch defaults plus a manual override input. Bump if completion rates drop; a maxed-out turn count is the right signal that prompt scope or codebase has drifted.

3. **`--permission-mode acceptEdits` is required for headless audits.** Without it, file writes silently hang on permission prompts that have nowhere to be answered. Documented in CI workflow; required in any local iteration too.

### One step deferred

Step 6 of the original block plan — a testing-branch calibration run — was deferred pending a manual pass on testing-aligned docs. Running the audit against testing now would mostly surface findings already catalogued in Block 3 Appendix A (the manual audit of testing-vs-main drift) rather than produce calibration data. The audit becomes a meaningful calibration tool *after* the manual pass narrows the known gap; reschedule Step 6 then.

### Production handoff checklist

When transferring App ownership to BuildDownAI and rolling out to production repos:

1. **Workflow YAML — three field changes** at `audit.yml`:
   - `owner: rigonzal530` → `owner: BuildDownAI` (in the token-mint step)
   - `repository: rigonzal530/AI-Implement` → `repository: BuildDownAI/AI-Implement` (in the AI-Implement checkout step)
   - `base: june-doc-update` → `base: main` (in the peter-evans step)
2. **Secrets on `BuildDownAI/docs`** — set: `DOCS_AUDIT_APP_CLIENT_ID`, `DOCS_AUDIT_APP_PRIVATE_KEY`, `CLAUDE_CODE_OAUTH_TOKEN`.
3. **Rotate the App's private key** after taking ownership so the dev-phase key (held locally during this block) becomes inert.
4. **Branch protection on `BuildDownAI/docs:main`** — confirm an approving review is required. Without it, the workflow's draft PR could technically self-merge given the App's `Pull requests: write` permission. The branch protection rule is the actual gate; the draft status is a soft signal.
5. **Schedule cron decision** — `0 14 * * 1` is currently active on the fork's `main`. Confirm intended behavior on production or disable until production is ready.

## How to update this prompt later — 5-step runbook

1. **Identify one specific issue from recent audit PRs.** Wrong categorization, missing finding type, schema drift, verbose LOW section, etc. Single observed failure mode per iteration — don't address multiple at once.

2. **Choose the lever — prompt vs. reference.** Format/shape issues → edit `audit-reference.md` (examples teach shape). Rules/criteria issues → edit `audit-prompt.md` (rules constrain criteria). Reference edits land more reliably; prompt edits are higher-leverage but easier to over-tighten.

3. **Test locally before pushing.** Setup once (sibling clone of AI-Implement + symlink into docs + `.gitignore` the symlink):
   ```bash
   cd <docs-repo>
   ln -s ../AI-Implement ai-implement
   echo "ai-implement" >> .gitignore
   ```
   Then per-iteration:
   ```bash
   git -C ../AI-Implement checkout <main|testing>
   claude -p "$(cat .github/audit-prompt.md)" \
     --max-turns 75 \
     --permission-mode acceptEdits \
     --add-dir ../AI-Implement
   ```
   Inspect `audit-report.md` and `.mdx` edits in the working tree. Revert with `git checkout -- <files>` between iterations.

4. **Run 2–3 times to assess variance.** Single runs are samples, not verdicts. The lever has landed if the targeted change is consistent across runs. If MEDIUM/LOW findings differ but HIGH stays variable, that's expected — HIGH carries more variance than MEDIUM (priority calls are more subjective).

5. **Commit and roll out.** Edit on `BuildDownAI/docs` via PR. Next scheduled cron or manual dispatch picks up the new prompt automatically; no workflow restart needed.

---

## Detailed sections (reference material)

### Operational fixes layered onto the workflow during dev

Each was discovered and patched during Step 4 testing. Listed here in case the same issue resurfaces during production rollout or future Claude Code CLI updates:

- **`actions/checkout` v4 → v5 and `actions/create-github-app-token` v1 → v3** — Node 20 deprecation; June 16, 2026 forced-switch deadline.
- **`app-id` → `client-id`** on `create-github-app-token@v3`. Different values on a GitHub App; the Client ID is the string `Iv23li...`-form identifier shown next to App ID in the App's settings.
- **`timeout-minutes: 3` on Install Claude Code CLI** — `npm install -g @anthropic-ai/claude-code` has a postinstall script that downloads a platform-native binary; can hang silently on Linux runners ([anthropics/claude-code#5209](https://github.com/anthropics/claude-code/issues/5209)). Tight timeout fails fast.
- **`claude --version` verification** appended to Install step. Confirms binary is on PATH; silent postinstall failure becomes a loud error.
- **`timeout-minutes: 45` on Run audit** — bounds a hung run (network issue, OAuth-token retry loop, etc.). 45 minutes covers up to ~125 turns at peak-load 25s/turn pace.
- **`continue-on-error: true` on Run audit** — peter-evans still runs even if Claude exhausts `--max-turns`. Partial output reaches a PR for diagnostic value; without this, an exit-code-1 step skips PR creation entirely.
- **`--permission-mode acceptEdits` on the Claude invocation** — required for non-interactive (`-p`) mode. `acceptEdits` is safer than `--dangerously-skip-permissions` (still respects protected paths like `.git`, `.claude`).
- **`add-paths` on peter-evans** — whitelists `audit-report.md` and `**/*.mdx`. Without this, the AI-Implement checkout's `.git/` directory caused the parent docs repo to stage `ai-implement` as a gitlink (mode 160000) during `git add -A`.
- **Workflow file MUST exist on the default branch** for `workflow_dispatch` to be available via `gh workflow run`. Fork's default branch (`main`) had to receive a fast-forward push of `automation/audit-workflow` before triggering became possible.

### Hybrid max-turns resolver — how the workflow picks a value

The workflow's `env.MAX_TURNS` expression:

```yaml
MAX_TURNS: ${{ github.event.inputs.max_turns != '' && github.event.inputs.max_turns || ((github.event.inputs.branch || 'testing') == 'main' && '75' || '100') }}
```

Resolves as:

| Trigger / inputs | MAX_TURNS |
|---|---|
| Schedule (no inputs) | `100` — falls through to testing default |
| Manual: `branch=main`, no override | `75` (main default) |
| Manual: `branch=testing`, no override | `100` (testing default) |
| Manual: any branch + `max_turns=125` | `125` (override wins) |

Override input lets you bump for edge cases without editing the YAML — useful when prompt scope grows or one-off heavy runs are needed.

### Run history

#### Run 1 — main, prompt v1, `--max-turns 30`
Hit cap at 9m 7s. No PR (step failure cascaded). Confirmed 30 turns is insufficient for `main`-scope audit with the v1 prompt's 7-area survey.

#### Run 2 — main, prompt v1, `--max-turns 50`, missing `acceptEdits`
Completed mentally at 10m 20s but couldn't write files; permission prompts had nowhere to be answered. Stdout transcript contained full audit findings, including H-1: Secrets-panel-stub discrepancy. Surfaced the `--permission-mode acceptEdits` requirement.

#### Run 3 — main, prompt v1, `--max-turns 50`, `acceptEdits`
First clean run. PR #1 opened on `rigonzal530/docs`. 1 HIGH (runner-image GHA customization undocumented), 5 MEDIUM, 4 LOW. Edit applied to `customize/runner-image.mdx`. Diff also staged the spurious `ai-implement` gitlink — surfaced the `add-paths` fix.

#### Run 4 — main, prompt v1.1 (with LOW worked-example), `--max-turns 50`, local
Hit max-turns at 50 — variance produced more HIGH-priority findings than Run 3 did, exceeding the previous comfortable budget. Confirmed 50 is at-the-edge for main; informed the bump to 75. Partial output included an H-1 on `custom-steps.mdx` listing 14 overridable steps when only 6 actually are. Reverted before commit pending the upcoming manual pass.

#### Run 5 — main, prompt v1.1, `--max-turns 50`, local (LOW-format iteration test)
Completed cleanly. LOW section now produces one-sentence bullets with inline parenthetical citations (~30 words each), down from Run 3's paragraph-per-finding format (~50 words each). The v1.1 reference edit landed.

### Variance observation across three completed runs (same prompt, same source state)

| Run | HIGH found | Source area |
|---|---|---|
| 2 (no-edit run) | Secrets panel stub vs. fully documented | `src/admin-ui/pages/stubs.ts` |
| 3 (CI complete) | GHA runner-image customization undocumented | `workflows/*.yml` |
| 5 (local complete) | `custom-steps.mdx` lists 14 overridable steps; only 6 actually are | `src/pipeline/default-pipeline.ts` |

All three are real, substantive HIGH findings — none overlap. Audit produces meaningfully different output across runs. Drives the "run 2–3 times and union the findings" operational guidance.

MEDIUM findings show higher cross-run overlap than HIGH (categorization is less subjective). LOW findings show the highest overlap because they catalog factual gaps rather than priority calls.

### Prompt iteration history

- **v1**: 5-layer structure with three worked HIGH examples in the reference. Strong baseline (Run 2 transcript, Run 3 PR).
- **v1.1**: Added a worked LOW-format example + anti-pattern list to `audit-reference.md`. Targets LOW-section verbosity — Run 3 produced paragraph-per-finding under v1, mismatching the prompt's "aggregate, no per-finding detail" schema directive. Reference now models one-sentence bullets with inline parenthetical citations. Run 5 confirms landing.

### Cost / rate calibration

- **Per-turn average:** 12–15s typical, 18–25s under peak API load.
- **Wall time on main:** 10–12 min completed runs at 50 turns; expect 15–19 min at 75 turns.
- **Wall time on testing (projected):** 25–32 min at 100 turns; 42–52 min worst-case at 125 turns.
- **OAuth-token consumption:** subscription-tier (no per-token cost). Each completed main-branch audit is a small fraction of weekly quota. Testing-branch likely 2–3× heavier.
- **Cost lever:** `--max-turns` cap is the right control. Tighter caps surface scope drift early; looser caps protect against spurious failures from variance. The override input gives per-dispatch flexibility for edge cases.

### Local-iteration gotchas

- **AI-Implement branch state is implicit locally.** CI dispatches with an explicit `branch:` input; local runs use whatever `git -C ../AI-Implement checkout` last did. A 50-turn cap that fits main exhausts on testing. Verify before each local audit:
  ```bash
  git -C ../AI-Implement branch --show-current
  ```
- **`--add-dir ../AI-Implement` required for the symlink to work.** Claude Code sandboxes the working directory; the symlink at `./ai-implement` resolves to `/home/.../AI-Implement` which is outside the sandbox unless explicitly added.
- **`.gitignore` the symlink** so git doesn't try to track it as untracked content.
- **Working-tree partial edits accumulate across failed runs.** Cleanup between iterations with `git checkout -- <files>` and `rm -f audit-report.md`.

### Open questions for follow-up

- Should the prompt explicitly instruct Claude to batch tool calls? (Could reduce turn count via parallel Read/Grep.)
- For findings beyond a count cap (e.g. >15 HIGH on testing branch initially), should the prompt split into multiple PRs or accept one large PR?
- Are MDX comment markers (`{/* AUDIT H-<n>: */}`) the right review-aid, or should we adopt a different convention (e.g., PR comments per finding, sidebar annotations)?
- Could a matrix-based parallel-runs setup capture the variance benefit in a single workflow dispatch (3 audits in parallel, union the findings)?
- Once the manual testing-pass lands, should the prompt's `Repo context` section be updated with `testing`-specific drift areas?
