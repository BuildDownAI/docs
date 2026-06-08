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

---

## Block 6 — Prompt v2 enrichment + bloat audit

### What landed in v2

Built on top of v1.1 baseline (commit `295ae0c` and on):

- **File-scope rule (new H2 in prompt)** — branch-aware editing: `main` audits edit root files only, `testing` audits edit `latest/**` only. Explicit path patterns in the prompt; checklist item verifies.
- **Reader-focused MDX principles (HIGH-edit step sub-bullets)** — no source paths or internal jargon; semantic component choice (`Note`/`Tip`/`Warning`/`Info`/`Check`); scannable chunking; anchor + cross-link discipline (incl. `/latest/` prefix on `latest/*` pages for both `]( )` Markdown and `href="..."` props).
- **Internal-only → changelog routing** — for testing audits, internal-only findings get `<Update>` entries in `latest/changelog.mdx` rather than per-page edits. Main audits omit them (no stable changelog by design).
- **Restructured audit-report shape** — top-level **Findings at a glance** + **Edits applied** tables for scannability across unioned reviews. Bullet-format finding details with bolded labels: HIGH = 3 bullets (`**Docs**:` / `**Source**:` / `**Edit**:`); MEDIUM = 2 bullets (no Edit since report-only); LOW numbered as `**L-N** —` for cross-reference.
- **Batch tool calls (Task step 2 addition)** — instructs Claude to parallelize independent `Read`/`Grep`/`Glob` calls. Highest-ROI single addition.
- **Reference Examples D–F** — BAD-vs-GOOD pairs showing (D) reader-focused vs source-citing edit, (E) version-prefix for latest/ cross-links covering both Markdown + href= props, (F) internal-only change as changelog entry.
- **Reference: Reader-focused principles section + expanded anti-patterns** — supplementary deterrents listing source-paths-in-MDX, internal jargon, default-Note, un-prefixed latest/ links, links to non-anchored elements (`<Step title>`/`<Tab title>`).
- **Examples A/B/C in reference reformatted** to match the new bullet-format finding shape — same content as v1.1, ported to demonstrate consistent shape.
- **Quality checklist (4 new items)** — verifying file-scope rule, reader-focused principles, `/latest/` prefix on cross-links, internal-only routing.

### Iteration history (v2)

| Round | What was tested | Result |
|---|---|---|
| 1 (local) | First v2 run on `automation/audit-workflow` | Branch-state issue: audit-workflow on stale pre-Block-5 base; findings against outdated docs. Useful only for shape verification. |
| 2 (local) | v2 prompt + rebased onto `docs-initial-update` | ~7–8m wall time; 2 HIGH / 3 MEDIUM / 6 LOW; file-scope held; reader-focused edits; substantive findings (`AI_IMPLEMENT_ALLOWED_RUNNER_IMAGE_PREFIXES` runner-image allowlist, `defaultBranch` required mismatch). |
| 3 (local) | + bullet labels + Examples A-C reformat | ~12–13m wall time; 7 HIGH / 4 MEDIUM / 5 LOW; bullet labels (`**Docs**: / **Source**: / **Edit**:`) landed; new substantive findings (WORKFLOW.md/PLANNING.md template drift, `autoApprovePlans` never-read by dispatch, Bedrock 4-hour STS session, Secrets-panel-stub). **Runner-image allowlist (H-1 in round 2) surfaced AGAIN as H-6 — high-impact findings recur across runs.** |

Key v2 takeaway: cross-run HIGH variance is structural (subjective categorization sampling). MEDIUM variance is lower (less subjective). The "run 2–3 times and union findings" operational pattern from Block 4 continues to apply — it's the right mitigation, not a bug to fix in the prompt.

### Bloat audit (Step 4.5)

Performed against 2 same-state baseline runs (rounds 2 + 3). The discipline guard requires ≥3 runs of pre-cut data; most candidates therefore landed as "inconclusive, defer."

| Addition | Verdict | Reason |
|---|---|---|
| File-scope rule | Keep | Held in both runs; zero out-of-scope edits |
| Batch tool calls | Keep | 3–4× wall-time improvement vs Block 4 baseline |
| Reader-focused MDX principles (prompt) | Keep | All round-3 edits user-facing; zero source paths in MDX |
| Component semantic choice | Keep | Round 3 used `Warning` (blocking), `Tip` (relief), `Info` (context) — not defaulting to `Note` |
| Scannable chunking | Keep | Multi-paragraph component bodies + bulleted lists observed |
| Anchor + cross-link discipline | Keep | Round-3 H-5 used `/latest/configuration/team-repo-mappings#param-auto-approve-plans` — correct `/latest/` prefix + `#param-<slug>` anchor pattern |
| Report shape (tables + bullets) | Keep | Both runs produced full table-driven shape |
| Bullet labels | Keep | Round 3 confirmed |
| Quality checklist additions | Keep | Self-checking evident via the "Notes on scope" output the audit added to its report |
| Examples A/B/C reformat | Keep | Reference-quality consistency with D-F shape |
| **Reader-focused principles section (reference)** | **Cut candidate — DEFERRED** | Mostly duplicates the prompt's HIGH-edit sub-bullets; cut test would need ≥3 baseline runs |
| **Expanded anti-patterns (5 new)** | **Cut candidate — DEFERRED** | Audit avoided these violations across 2 runs, but can't attribute prevention to the anti-pattern list vs the prompt's positive rules; cut test would need ≥3 baseline runs |
| **Internal-only → changelog routing** | **Inconclusive — DEFERRED** | Never exercised in 2 runs (no internal-only findings surfaced). Audit explicitly acknowledged the rule (round 3 "Notes on scope" said *"every drift surfaced this cycle was operator-visible"*). Cut would be premature; future-proofing value when a true internal-only finding does appear |

**Decision:** Defer all cut candidates to a follow-up bloat-audit block once weekly CI produces ≥3 same-state baseline runs. Current state pushes to BuildDownAI with all additions intact.

### Production handoff checklist additions (delta from v1)

The v1 production handoff checklist (above, under "Production handoff checklist") still applies. Block 6 adds:

- **Cron cadence consideration** — at round 3's pace (7 HIGH/run), a weekly cron could produce ~20–40 HIGH findings/month for mentor review. Worth discussing cadence (weekly / bi-weekly / monthly / on-demand) before enabling cron in production. Defaults to weekly Mondays 14:00 UTC currently; can be disabled by removing the `schedule:` block from `audit.yml` until cadence is decided.
- **Multi-run for confidence on important findings** — important drift consistently recurs across runs (e.g., runner-image allowlist surfaced in both rounds 2 and 3). When a pre-release audit needs maximum coverage, dispatch 2–3 times manually and union the findings.

### Open questions added by Block 6

- Are the 5 new anti-patterns and the Reader-focused-principles section in the reference earning their place? Need ≥3 baseline runs to test cut safely.
- Does the internal-only → changelog routing rule work in practice when a true internal-only finding surfaces? (Round 3 had none.) First testing-CI run that surfaces a runner-telemetry-style finding will tell us.
- Should the prompt have a soft cap on HIGH findings per run? At ~7 per run, mentor-review load adds up; a cap (e.g., 5–10) with overflow logged as MEDIUM might smooth review pace. Trade-off: lower HIGH count vs lost coverage signal.
