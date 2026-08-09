# AI-Implement Documentation Audit — Reference

This file grounds the audit by example. It defines the priority rubric, shows what well-formed findings look like across different drift shapes, and lists the anti-patterns to avoid. Distilled from manual audits performed in Block 2 (against AI-Implement `main`) and Block 3 (against `testing`).

## Priority rubric

Apply these criteria when categorizing drift between source and the docs:

- **HIGH** — A reader following the current docs would take an incorrect action. Either the docs claim something the code no longer supports, or a load-bearing feature exists in code with no documentation at all.
- **MEDIUM** — Coverage gaps or outdated phrasing where the reader can mostly still succeed but the docs lag the code. New optional features, additions to existing tables, renamed fields with backward-compat shims.
- **LOW** — Polish, structure, internal-only changes that don't affect reader behavior. Refactoring, code-comment improvements, dependency bumps.

### Tier → action

**This table is the single source of truth for which tiers produce edits.** Both prompts and the report templates derive from it and deliberately do not restate it — changing it here changes the whole pipeline.

| Tier | Audit output | Report detail per finding |
|---|---|---|
| **HIGH** | draft MDX edits applied | Docs / Source / Edit |
| **MEDIUM** | report-only, no edits | Docs / Source |
| **LOW** | counted in aggregate | one line |

"Edit-receiving tiers" throughout the prompts means whichever tiers this table marks as producing edits.

## Required shape for every finding

- One-line title
- `file:line` citation pointing to source-code evidence in AI-Implement
- What the docs currently say (with the docs path)
- What the code actually shows
- Priority tier
- For edit-receiving tiers: a brief suggested edit (1–3 sentences)

## How edits should read

The docs repo's own `./CLAUDE.md` holds the rules for how documentation here is written — reader focus, component choice by intent, chunking, callout density, anchor and version-prefix discipline, and what never becomes a page edit. Read it before applying any edit.

Those rules live there and are deliberately not restated here, so there is one home to improve.

Do not confuse it with `./ai-implement/CLAUDE.md`, which describes the source codebase's architecture and is read for a different purpose.

This file covers what is specific to auditing: how to prioritize a finding, what a finding must contain, how to hunt for claims that have gone stale, and the failure modes to avoid.

## Worked examples

These illustrate the audit shape across three different drift types. Use them as a template for findings produced during the audit.

### Worked finding (HIGH)

**Title**: Per-project run caps (`maxTurns`, `maxIterations`, `maxJobMinutes`) added to `RepoMapping`

- **Docs**: `configuration/team-repo-mappings.mdx` lists the mapping fields but not the three caps. `reference/admin-ui.mdx` describes the Projects panel without mentioning the Capacity step's inputs.
- **Source**: `src/config.ts` (new fields on the `RepoMapping` interface); `src/admin-ui/pages/projects.ts` (the edit dialog exposes them) — wired through schema, migration, dispatch payload, and admin UI.
- **Edit**: Add three `<ParamField>` blocks to `configuration/team-repo-mappings.mdx`, with per-provider defaults noted. Mention the Capacity additions in the Projects panel description of `reference/admin-ui.mdx`.

**Priority rationale**: HIGH — operators won't learn about a feature designed to control cost and reliability.

The same shape applies to removals (docs describe a variable the code no longer reads) and structural refactors (docs list override points that no longer execute). What changes is the evidence, not the format.

### Reader-focused vs source-citing edit (BAD vs GOOD)

A HIGH finding identifies that `reviewProviders` is a new optional field in `.ai-implement/config.yml` (verified at `src/pipeline/steps/install.ts:25-71`). The audit's draft edit should look like this:

**BAD (source-citing, jargon-heavy):**

```mdx
<ParamField body="reviewProviders" type="string[]">
  Controls whether the `post-push-review` step (handler at `src/pipeline/steps/post-push-review.ts:235`) waits on external review providers. The implementation uses an `ExternalReviewState` enum with values "skipped" | "absent" | "running" | "completed". When the key is absent, `resolveExternalReview()` performs check-name matching; an empty array short-circuits it.
</ParamField>
```

Issues: cites `file:line`, names the internal `ExternalReviewState` enum and the `resolveExternalReview()` function. The reader doesn't have the codebase open; this is hostile to them.

**GOOD (reader-focused):**

```mdx
<ParamField body="reviewProviders" type="string[]">
  Controls whether the `post-push-review` step waits for an external review check on the pull request and folds its findings into its own review.

  Leave the field out and the step waits, detecting the external check on its own. This is the default.

  Set it to an empty list to turn the wait off, so the step reviews the pull request by itself. Set it to `github-claude-code-review` to wait explicitly; unrecognized entries are ignored.
</ParamField>
```

Why: it drops the source path and the internal type names, chunks the body into one-beat paragraphs, and — the part the earlier version got wrong — states omitting the field and setting it to an empty list as the opposite outcomes they are, rather than collapsing them into one clause.

### Version-prefix on `latest/` cross-links

Cross-links from a `latest/*` page must carry the `/latest/` prefix, or they resolve to the stable page and bounce the reader out of their version.

The non-obvious part is that this applies to **two** pattern families, and a sweep handling only the first misses the second:

````mdx
See [Project mappings](/latest/configuration/team-repo-mappings) for the admin-UI side.

<Card title="Customize WORKFLOW.md" href="/latest/customize/workflow-md">
````

A prior bulk sweep that handled only Markdown links left twelve `<Card href=>` props unprefixed across five pages.

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

These are failure modes the positive rules above don't obviously forbid. Do not produce findings that match any of these:

- **Do not fabricate documentation pages.** If a feature exists in code and no corresponding docs page exists, flag it as a finding describing the gap. Do not create new docs files.
- **Do not document code that does not exist.** Every claim must point to a real `file:line` citation. If the source cannot be located, the finding is invalid.
- **Do not categorize stylistic improvements as HIGH.** "This sentence could be clearer" is LOW. HIGH means a reader takes a wrong action.
- **Do not propose changes outside the docs repo.** Source is read-only context. Never suggest source-code edits as part of a finding.
- **Do not flag intentional simplification as drift.** Docs sometimes omit internal implementation detail for reader clarity. If user-facing behavior matches what the docs describe, there is no finding — even where the source has internal complexity the docs don't surface.
- **Do not announce findings you are not listing.** A note telling the reviewer that some set of findings was excluded — as already-fixed, out of scope, or uncertain — is unactionable: they cannot tell what is missing or judge whether the exclusion was sound. Either report the finding at its tier, or leave it out with no reference to it. This is distinct from the `## Audit incomplete` section, which names work you did not reach and *is* actionable.

## Verifying a claim before documenting it

These checks catch drift that a prose-versus-source comparison structurally cannot. Each one exists because a real audit missed something.

**Diff environment-variable docs against their canonical sources, in both directions.** Two machine-readable sources are maintained alongside the code and are usually ahead of the docs: `.env.example` for orchestrator variables, and the `# Optional repository or organization variables:` header blocks in each synced workflow for target-repo variables. Compare the documented set against both:

- In a source but not the page → undocumented variable.
- On the page but not a source → removed or renamed variable.
- In both, but the source comment says more than the page → the page is stale on semantics.

The third direction is the valuable one and the easiest to skip. Four findings came out of this in one pass, from two file reads, after five prior audits had missed all four — because asking "does this variable appear in the source?" finds a stale reference and confirms it, while a set difference cannot be satisfied by a single confirming hit.

**A key being accepted is not the same as a key being used.** Before documenting a configuration value, check two things separately: that a parser accepts it, and that something *reads* it outside the parser. A front-matter key sat in the accepted-key list and on the interface while no pipeline step consumed it — so it round-tripped silently, and an operator setting it to a cheaper model was ignored and paid full rate. A documented default that appears nowhere in production source (only in test fixtures) is strong evidence the consumer is gone.

**Read the guard clauses, not just the happy path.** An early return above the logic you are reading may fire routinely rather than rarely. A label-removal path was documented as unconditional; a terminal-state guard sat above the label filter and fired systematically, because the tracker's own integration closed issues faster than the poll ran. Ask which path is *typical*, and whether an external mechanism races the one you are describing.

**In troubleshooting content, the cause may be inferred but the remediation must be verified.** Causes often can only come from source. Remediations name something the reader operates, and are always checkable — verify the **exact label** the user sees (not the internal field name), **which surface exposes it** (creation flow, edit dialog, config file, environment variable — these diverge, and docs are usually read during the creation flow), and **whether it is configurable at all**. A fix pointed at "Max Job Minutes on the project mapping" when the control is labelled "Job Timeout (min)" and appears only in the edit dialog, not the creation stepper.

### Configuration surfaces

Deriving reachability per key is expensive; the set of *surfaces* is small and changes rarely. Use this map, and run the acceptance-and-consumption check above only for a surface not listed here.

| Surface | Where the user writes it | Accepted keys |
|---|---|---|
| `.ai-implement/config.yml` | target repo root | `packageManager`, `models.implement`, `models.review`, `reviewProviders` |
| Pipeline YAML (`custom/pipelines/*.yml`) | orchestrator install | **only** `id`, `type`, `moduleId` per step — an `inputs:` key is silently dropped |
| `WORKFLOW.md` front matter | target repo root | accepted: `model`, `setup`, `verify`, `teardown`, `gap_analysis_model` — **consumed: the first four only** |
| `.ai-implement/image.yml` | target repo root | runner image pin |
| Project mapping | admin UI stepper, edit dialog, or API | the documented mapping fields — note the stepper exposes fewer than the edit dialog |
| Orchestrator environment | `.env` or deployment secrets | as per `.env.example` |
| Target-repo CI | GitHub Actions secrets and variables | as referenced by the synced workflows |
| Issue description block | tracker issue body | `feature_branch.mode` |

### Sweep for claims that age

Surveying source-first answers "is this documented?" and finds gaps. It cannot find a claim that is documented *wrongly* — that still answers yes. On an actively-developed codebase most drift is of the second kind: a statement that was accurate when written and was silently invalidated by a change elsewhere, with nobody editing the doc.

Grep the docs for these shapes directly, then verify each hit against current source. The greps produce candidates, not findings — some hits will be fine, and that is cheap to establish.

| Shape | Grep for | Why it rots |
|---|---|---|
| Exclusivity | `the only`, `only branch`, `sole` | false the moment a second option exists |
| Temporal hedge | `currently`, `at present`, `for now`, `not yet` | the hedge admits the author expected change |
| Enumeration | `two endpoints`, `four groups`, `eight`, `three steps` | any addition breaks the count without touching the sentence |
| Specific default | a version, model ID, timeout, or cap stated as a value | changes independently of the prose around it |
| Negation | `does not support`, `cannot be`, `is not` | the cheapest thing for a release to falsify |
| Collapsed states | a state word — `omitted`, `unset`, `blank`, `empty`, `absent`, `missing` — joined to another by `or` | a guard is usually written that way *because* the two states differ |

Real instances: "this is the only branch that currently supports plugin installation" — true when written, false once the default branch gained a catalog. "Two endpoints sit outside the namespace" — there were three. A documented default that existed only in test fixtures.

Of one manual pass's thirteen highest-priority findings, **seven were false rather than missing**. Nothing in the source announces that a doc sentence became wrong, so only a docs-first pass surfaces them.

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

Findings with no operator-visible behavior are report-only on both branches — see *What never becomes a page edit* in `./CLAUDE.md`.

## Note on file:line citations

Line numbers in the worked examples above were accurate at the time of writing but may shift as the codebase evolves. The shape — file path + approximate location + nature of evidence — is what matters. In actual audit output, cite the line numbers you find at audit time, not these examples.
