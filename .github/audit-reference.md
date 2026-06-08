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

## Reader-focused edit principles

When applying HIGH-priority edits to MDX, follow these principles. Examples D–F below illustrate them in practice; this section lists the rules.

- **No internal vocabulary in MDX.** Drop source paths, `file:line` citations, TypeScript type-system terms (enum/interface/field names), Mintlify component names in prose ("uses the `<Update>` component"), and meta-mechanic explanations ("the runner-callback handles this"). The reader doesn't know the implementation. Per-page MDX describes user-relevant behavior in user-relevant language. (The changelog has more leeway — see Example F.)
- **Component choice by semantic intent.** `<Note>` for neutral important facts. `<Tip>` for operator-benefit / relief callouts. `<Warning>` for blocking issues that prevent functionality. `<Info>` for permissions or context-setting. `<Check>` for success states. Don't default to `<Note>` — pick by what the callout is *for*.
- **Scannable chunking within components.** A `<ParamField>` body that covers what-it-does + accepted-values + defaults + when-to-use should be 2–4 short paragraphs separated by blank lines, not one wall of prose. Use bulleted lists for parallel items (accepted values, file lists). The eye should land on structure before reading.
- **Cross-link discipline.** Verify the destination anchor exists. Mintlify anchor rules:
  - H2/H3/H4 markdown headings, `<Accordion title="X">`, `<Update label="X">` → `#<slugified-text>`
  - `<ParamField body="X">` → `#param-<slugified-x>` (note the `#param-` prefix)
  - `<Step title>` inside `<Steps>`, `<Tab title>` inside `<Tabs>` → **no anchor generated**; fall back to a page-level link
  - For `latest/*` pages, every cross-link to another docs page must use the `/latest/` prefix. This covers both `]( )` Markdown links AND `href="..."` component props (e.g., `<Card href=...>`)
- **Internal-only changes go to changelog (testing audits) or are omitted (main audits).** Per the prompt's File-scope section. A change with no operator-visible behavior is not a per-page edit; it's a changelog entry (testing) or no entry at all (main, until a stable changelog page exists).

## Worked examples

These illustrate the audit shape across three different drift types. Use them as a template for findings produced during the audit.

### Example A — Feature removal (HIGH)

**Title**: `ORCHESTRATOR_URL` no longer read; consolidated into `RUNNER_CALLBACK_BASE_URL`

- **Docs**: `configuration/environment-variables.mdx` documents `ORCHESTRATOR_URL` as a separate orchestrator-runtime variable under "Runner callbacks."
- **Source**: `.env.example` (variable removed); `src/index.ts` (no longer reads `process.env.ORCHESTRATOR_URL`) — `RUNNER_CALLBACK_BASE_URL` now serves the same purpose.
- **Edit**: Remove the `ORCHESTRATOR_URL` `<ParamField>` block from the Orchestrator-runtime tab. Add a one-line transition note in the surrounding prose: "Previously, `ORCHESTRATOR_URL` was a separate variable; it has been consolidated into `RUNNER_CALLBACK_BASE_URL`."

**Priority rationale**: HIGH — readers setting `ORCHESTRATOR_URL` would expect it to do something; it has no effect.

### Example B — Feature addition (HIGH)

**Title**: Per-project run caps (`maxTurns`, `maxIterations`, `maxJobMinutes`) added to `RepoMapping`

- **Docs**: `configuration/team-repo-mappings.mdx` lists the mapping fields but does not include the three caps. `reference/admin-ui.mdx` describes the Projects panel without mentioning the Capacity step's new inputs.
- **Source**: `src/config.ts:52` (new fields on the `RepoMapping` interface); `src/admin-ui/pages/projects.ts:273` (stepper exposes the three fields) — fully wired through schema, database migration, dispatch payload, and admin UI.
- **Edit**: Add three `<ParamField>` blocks to `configuration/team-repo-mappings.mdx` after `provider`, with per-provider defaults noted (Bedrock=2 vs Anthropic=3 for `maxIterations`). Mention the Capacity-step additions in the Projects panel description of `reference/admin-ui.mdx`.

**Priority rationale**: HIGH — operators reading the docs won't learn about a feature explicitly designed to control cost and reliability.

### Example C — Structural refactor (HIGH)

**Title**: Planning is now a separate executor; five planning-pipeline steps removed

- **Docs**: `customize/custom-steps.mdx` AccordionGroup lists the deleted step IDs (`test-plan`, `work-unit-decomposition`, `architecture-analysis`, `cross-story-context`, `post-to-ticketing`) as override points readers can implement.
- **Source**: `src/run-planning.ts` (new entry point); `src/pipeline/steps/` (the five step files are deleted from the codebase); `src/pipeline/types.ts` (`StepType` enum lost six values) — planning now runs through a dedicated executor; custom override files using those filenames would never execute.
- **Edit**: Remove the five planning-step accordions from the overridable-step list. Add a `<Note>` explaining that planning now runs through a dedicated planning workflow (`claude-plan.yml`) and is not part of the customizable autonomous pipeline.

**Priority rationale**: HIGH — readers following the custom-steps guide would write code that has no effect.

### Example D — Reader-focused vs source-citing edit (BAD vs GOOD)

A HIGH finding identifies that `reviewProviders` is a new optional field in `.ai-implement/config.yml` (verified at `src/pipeline/steps/install.ts:25-71`). The audit's draft edit should look like this:

**BAD (source-citing, jargon-heavy):**

````mdx
<ParamField body="reviewProviders" type="string[]">
  Opts the `post-push-review` step (handler at `src/pipeline/steps/post-push-review.ts:235`) into waiting for external review providers. The implementation uses an `ExternalReviewState` enum with values "skipped" | "absent" | "running" | "completed". When set, the orchestrator's `resolveExternalReview()` function performs check-name matching.
</ParamField>
````

Issues: cites `file:line`, names internal `ExternalReviewState` enum, mentions internal function `resolveExternalReview()`. Reader doesn't know the codebase; this is hostile to them.

**GOOD (reader-focused):**

````mdx
<ParamField body="reviewProviders" type="string[]">
  Opts the `post-push-review` step into waiting for and integrating findings from external GitHub Actions review checks on the same PR.

  Currently the only recognized value is `github-claude-code-review` (matches the Claude Code Review GitHub Action). Unknown values are silently filtered out.

  When omitted or empty, `post-push-review` runs its own analysis without coordinating with external reviewers.
</ParamField>
````

Why: drops source paths and internal type names; chunks the body into one-beat paragraphs; describes user-relevant behavior and choices. The reader can decide whether to set this field without knowing how it's implemented.

### Example E — Version-prefix for `latest/` cross-links (BAD vs GOOD)

When editing `latest/*.mdx` files, cross-links to other docs pages must use the `/latest/` prefix. Otherwise they resolve to root-level (stable) pages and bounce the latest reader out of their version.

**BAD (un-prefixed link from a `latest/` page):**

````mdx
See [Project mappings](/configuration/team-repo-mappings) for the admin-UI side.
````

````mdx
<Card title="Customize WORKFLOW.md" icon="pen-to-square" href="/customize/workflow-md">
  Edit the Claude prompt template to match your repo's stack and conventions.
</Card>
````

Issues: both links land on root-level (stable) versions of those pages. A reader currently on `/latest/configuration/environment-variables` who clicks the first link gets bounced to `/configuration/team-repo-mappings` (stable), losing their version context.

**GOOD (prefixed, both shapes):**

````mdx
See [Project mappings](/latest/configuration/team-repo-mappings) for the admin-UI side.
````

````mdx
<Card title="Customize WORKFLOW.md" icon="pen-to-square" href="/latest/customize/workflow-md">
  Edit the Claude prompt template to match your repo's stack and conventions.
</Card>
````

Why: explicit `/latest/` prefix keeps the reader in their version. Note that **both** `]( )` Markdown link syntax AND `href="..."` component props (Cards, Buttons, etc.) need the prefix — a bulk-sweep that only handles Markdown links misses the `href=` props.

### Example F — Internal-only change → changelog entry (testing audits only)

When a `testing` audit surfaces a change that has no operator-visible behavior (internal refactor, DB column addition, auth-plumbing shift, telemetry foundation), the finding does NOT become a per-page edit. Instead, add an entry to `./latest/changelog.mdx` using the `<Update>` component.

Example: testing branch added stream-JSON telemetry for the runner — entirely internal plumbing the operator never sees directly. The audit emits:

````mdx
<Update label="Runner telemetry stream added" description="Available in latest" tags={["Internal", "Pipeline"]}>
  Runner sessions now emit structured JSON telemetry to the orchestrator over a streaming channel, providing the foundation for richer step-progress reporting in the admin UI.

  No operator action needed — entirely internal. The runner image, workflow files, and orchestrator are all updated together via the standard `sync-workflows` action.
</Update>
````

Why this works:

- It's added to `./latest/changelog.mdx` (the testing-audit target for internal-only findings), not to a per-page docs file
- The changelog audience is broader than per-page docs (operators + AI-Implement developers cross-referencing commit history), so high-level mentions of internal handling are OK
- The entry IS self-contained: a reader gets the change context without clicking elsewhere
- The `<Update label="...">` produces the anchor `#runner-telemetry-stream-added` automatically, available for future cross-references
- `tags={["Internal", ...]}` flags the entry as internal-plumbing in the filterable sidebar

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
- **Do not cite source paths in MDX edits.** `file:line` references, source file names, and internal module paths belong only in `audit-report.md`, not in the docs themselves. The reader doesn't have the source open.
- **Do not use internal jargon in MDX prose.** TypeScript type-system terms (enum/interface/field names), internal function names, internal state-machine values, and Mintlify component names in prose ("uses the `<Update>` component") all fail the reader-focus test. Either rephrase user-facingly or omit.
- **Do not default to `<Note>` for every callout.** Pick the component by semantic intent (see Reader-focused edit principles section above).
- **Do not leave cross-links unprefixed on `latest/*` pages.** Un-prefixed paths resolve to root-level (stable) pages, bouncing readers out of their version. Both Markdown `]( )` syntax and `href="..."` props need the `/latest/` prefix.
- **Do not link to non-anchored elements.** `<Step title>` and `<Tab title>` do not generate anchors. Cross-links to `#step-title-slug` silently land at page top. Fall back to a page-level link with prose guidance.

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

For `testing` audits specifically, `./latest/changelog.mdx` is the home for any finding with no operator-visible behavior (internal refactors, telemetry foundations, auth-plumbing shifts, etc.) — see Example F above. For `main` audits, internal-only findings are omitted entirely; no stable changelog page exists by design.

## Note on file:line citations

Line numbers in the worked examples above were accurate at the time of writing but may shift as the codebase evolves. The shape — file path + approximate location + nature of evidence — is what matters. In actual audit output, cite the line numbers you find at audit time, not these examples.
