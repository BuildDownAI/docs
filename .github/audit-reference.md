# BuildDown Documentation Audit — Reference

This file grounds the audit by example. It defines the priority rubric, shows what well-formed findings look like across different drift shapes, and lists the anti-patterns to avoid. Distilled from manual audits of both products, run against a released version and against the development branch.

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
- `file:line` citation pointing to source-code evidence in the source checkout for this run's scope
- What the docs currently say (with the docs path)
- What the code actually shows
- Priority tier
- For edit-receiving tiers: a brief suggested edit (1–3 sentences)

## How edits should read

The docs repo's own `./CLAUDE.md` holds the rules for how documentation here is written and verified — how to check a claim against source, reader focus, component choice by intent, chunking, callout density, anchor and version-prefix discipline, and what never becomes a page edit. Read it before applying any edit.

Those rules live there and are deliberately not restated here, so there is one home to improve.

This file covers what is specific to auditing: how to prioritize a finding, what a finding must contain, how to hunt for claims that have gone stale, and the failure modes to avoid.

## Worked examples

These illustrate the audit shape. Use them as a template for findings produced during the audit.

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

- **Do not categorize stylistic improvements as HIGH.** "This sentence could be clearer" is LOW. HIGH means a reader takes a wrong action.
- **Do not propose changes outside the docs repo.** Source is read-only context. Never suggest source-code edits as part of a finding.
- **Do not flag intentional simplification as drift.** Docs sometimes omit internal implementation detail for reader clarity. If user-facing behavior matches what the docs describe, there is no finding — even where the source has internal complexity the docs don't surface.
- **Do not announce findings you are not listing.** A note telling the reviewer that some set of findings was excluded — as already-fixed, out of scope, or uncertain — is unactionable: they cannot tell what is missing or judge whether the exclusion was sound. Either report the finding at its tier, or leave it out with no reference to it. This is distinct from the `## Audit incomplete` section, which names work you did not reach and *is* actionable.

## Sweep for claims that age

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

## Note on the examples

The examples in this file illustrate shape — a file path, a location, and the nature of the evidence — not current product behavior. Their facts and line numbers were true when written and may not be now, so never cite one as evidence — every finding's evidence comes from this run's source checkout.
