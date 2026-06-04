# AI-Implement Documentation Audit — Reference

This file grounds the audit by example. It defines the priority rubric, shows what well-formed findings look like across different drift shapes, and lists the anti-patterns to avoid. Distilled from manual audits performed in Block 2 (against AI-Implement `main`) and Block 3 (against `testing`).

## Priority rubric

Apply these criteria when categorizing drift between AI-Implement source and the docs:

- **HIGH** — A reader following the current docs would take an incorrect action. Either the docs claim something the code no longer supports, or a load-bearing feature exists in code with no documentation at all. HIGH findings receive **draft doc edits** in the audit output.
- **MEDIUM** — Coverage gaps or outdated phrasing where the reader can mostly still succeed but the docs lag the code. New optional features, additions to existing tables, renamed fields with backward-compat shims. MEDIUM findings are **report-only** — no edits.
- **LOW** — Polish, structure, internal-only changes that don't affect reader behavior. Refactoring, code-comment improvements, dependency bumps. LOW findings are **counted in aggregate** — not individually enumerated.

## Required shape for every finding

- One-line title
- `file:line` citation pointing to source-code evidence in AI-Implement
- What the docs currently say (with the docs path)
- What the code actually shows
- Priority tier
- For HIGH only: a brief suggested edit (1–3 sentences)

## Worked examples

These illustrate the audit shape across three different drift types. Use them as a template for findings produced during the audit.

### Example A — Feature removal (HIGH)

**Title:** `ORCHESTRATOR_URL` no longer read; consolidated into `RUNNER_CALLBACK_BASE_URL`

**Source evidence:** `.env.example` (removed lines in the Orchestrator-callback section); `src/index.ts` (no longer reads `process.env.ORCHESTRATOR_URL` anywhere)

**Docs claim:** `configuration/environment-variables.mdx` documents `ORCHESTRATOR_URL` as a separate orchestrator-runtime variable under "Runner callbacks."

**Code reality:** The variable is no longer present in `.env.example` and is not read in `src/`. `RUNNER_CALLBACK_BASE_URL` now serves the same purpose.

**Priority:** HIGH — readers setting `ORCHESTRATOR_URL` would expect it to do something; it has no effect.

**Suggested edit:** Remove the `ORCHESTRATOR_URL` `<ParamField>` block from the Orchestrator-runtime tab. Add a one-line transition note in the surrounding prose: "Previously, `ORCHESTRATOR_URL` was a separate variable; it has been consolidated into `RUNNER_CALLBACK_BASE_URL`."

### Example B — Feature addition (HIGH)

**Title:** Per-project run caps (`maxTurns`, `maxIterations`, `maxJobMinutes`) added to `RepoMapping`

**Source evidence:** `src/config.ts` (new fields on the `RepoMapping` interface around line 52); `src/admin-ui/pages/projects.ts` (stepper exposes the three fields around line 273)

**Docs claim:** `configuration/team-repo-mappings.mdx` lists the mapping fields but does not include the three caps. `reference/admin-ui.mdx` describes the Projects panel without mentioning the Capacity step's new inputs.

**Code reality:** Fully wired through schema, database migration, dispatch payload, and admin UI. Operators using the latest version see and can set these three fields.

**Priority:** HIGH — operators reading the docs won't learn about a feature explicitly designed to control cost and reliability.

**Suggested edit:** Add three `<ParamField>` blocks to `configuration/team-repo-mappings.mdx` after `provider`, with per-provider defaults noted (Bedrock=2 vs Anthropic=3 for `maxIterations`). Mention the Capacity-step additions in the Projects panel description of `reference/admin-ui.mdx`.

### Example C — Structural refactor (HIGH)

**Title:** Planning is now a separate executor; five planning-pipeline steps removed

**Source evidence:** `src/run-planning.ts` (new entry point); `src/pipeline/steps/` (deletion of `test-plan.ts`, `work-unit-decomposition.ts`, `architecture-analysis.ts`, `cross-story-context.ts`, `post-to-ticketing.ts`); `src/pipeline/types.ts` (`StepType` enum lost six values)

**Docs claim:** `customize/custom-steps.mdx` AccordionGroup lists the deleted step IDs as override points readers can implement.

**Code reality:** The step files are deleted from the codebase. Custom override files using those filenames would never execute. Planning runs through a dedicated executor instead.

**Priority:** HIGH — readers following the custom-steps guide would write code that has no effect.

**Suggested edit:** Remove the five planning-step accordions from the overridable-step list. Add a `<Note>` explaining that planning now runs through a dedicated `run-planning.ts` executor and is not part of the customizable autonomous pipeline.

### LOW findings — what aggregate format looks like

LOW findings are terse bullets, max ~25 words each, no per-finding headings, no detailed evidence sections. The goal is to surface that they exist for a human reviewer to skim — not to make a case for each one.

Example:

- Stepper docs combine three UI steps into one (`reference/admin-ui.mdx:104`)
- Workflow YAMLs use different default image tags (`:next` vs `:latest`) across `claude-implement.yml` and `comment-trigger.yml`
- Comment-trigger permission check accepts `maintain`/`admin` roles beyond docs' "write access" wording
- `ensureTeamLabel()` is a Linear-internal helper; no operator docs gap

LOW findings do **not** need:
- Source / Docs claim / Code reality sub-sections
- Suggested edits
- File-and-line context beyond an inline parenthetical

## Anti-patterns

Do not produce findings that match any of these patterns:

- **Do not fabricate documentation pages.** If a feature exists in code and no corresponding docs page exists, flag it as a finding describing the gap. Do not create new docs files.
- **Do not document code that does not exist.** Every claim must point to a real `file:line` citation. If the source cannot be located, the finding is invalid.
- **Do not categorize stylistic improvements as HIGH.** "This sentence could be clearer" is LOW. HIGH means a reader takes a wrong action.
- **Do not propose changes outside the docs repo.** AI-Implement source is read-only context. Never suggest source-code edits as part of an audit finding.
- **Do not flag intentional simplification as drift.** Docs sometimes omit internal implementation detail for reader clarity. If user-facing behavior matches what the docs describe, no finding — even if the source has internal complexity not surfaced in the docs.

## Areas commonly affected by drift

These are the source areas where drift typically appears first. Start the audit by surveying these:

- Environment variables (`.env.example` vs `configuration/environment-variables.mdx`)
- Pipeline steps (`src/pipeline/steps/` directory vs `customize/custom-steps.mdx` accordions)
- Admin UI panels and endpoints (`src/admin-ui/sidebar.ts` + `src/admin.ts` vs `reference/admin-ui.mdx`)
- Mapping fields and configuration schema (`src/config.ts` `RepoMapping` interface vs `configuration/team-repo-mappings.mdx`)
- Provider behavior (`src/providers/linear.ts`, `src/providers/jira.ts` vs `providers/*.mdx`)
- Workflow templates synced to target repos (`workflows/*.yml` vs `setup/target-repo.mdx`)
- Status markers and labels (provider-specific constants vs `reference/labels.mdx`)

Other source areas (`src/log.ts`, `src/webhook.ts`, low-level utilities) often have changes with little reader-facing impact — survey them only if time permits.

## Note on file:line citations

Line numbers in the worked examples above were accurate at the time of writing but may shift as the codebase evolves. The shape — file path + approximate location + nature of evidence — is what matters. In actual audit output, cite the line numbers you find at audit time, not these examples.
