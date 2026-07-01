---
# Claude model used for implementation. Passed through verbatim to
# `claude-code --model`, so any ID your configured provider accepts is fine.
# Examples:
#   Anthropic API / OAuth: claude-sonnet-4-6, claude-opus-4-8, claude-haiku-4-5-20251001
#   AWS Bedrock:           anthropic.claude-sonnet-4-6-20250805-v1:0
#                          or an inference-profile ARN (arn:aws:bedrock:...)
# The default below works for the Anthropic provider. If this repo's mapping
# is switched to provider=bedrock in the orchestrator admin UI, replace this
# with a Bedrock model ID — the workflow will hard-fail otherwise, since
# Bedrock IDs are account- and region-specific and have no safe default.
# Opus chosen for documentation writing quality
model: claude-opus-4-8

# Optional: model used for the post-PR gap-analysis step. Pass through as above.
# Default (if omitted): claude-haiku-4-5-20251001 for anthropic, same as `model`
# above for bedrock (override if you want a cheaper model for gap analysis).
# gap_analysis_model: claude-haiku-4-5-20251001
---

<!--
  WORKFLOW.md — Claude AI Implementation prompt template
  =======================================================
  This file is seeded into your repo by the ai-implement sync workflow.
  It is YOURS to customise — future syncs will never overwrite it.

  When claude-implement.yml runs, it renders this file as the prompt sent to
  Claude Code. The YAML front matter block (between the --- lines) is stripped
  before Claude sees it. The rest of the file is passed through envsubst, which
  substitutes the following variables:

    ${ISSUE_IDENTIFIER}   Linear identifier, e.g. ENG-42
    ${ISSUE_TITLE}        Issue title
    ${ISSUE_DESCRIPTION}  Full issue description (Markdown)
    ${ISSUE_ID}           Linear UUID (useful if you want Claude to call the Linear API)
    ${PR_NUMBER}          Set on gap-fill re-runs; empty on first run
    ${PLANNING_CONTEXT}   Rendered planning comments (empty if none); expands to a full "## Planning Context" block or nothing

  FRONT MATTER (the --- block at the top)
  ----------------------------------------
  Stripped before sending to Claude. Supported keys:

    model                Model ID for implementation (see above).
    gap_analysis_model   Model ID for the post-PR gap-analysis step (see above).
    setup      Path (relative to repo root) to a shell script that runs BEFORE Claude.
               Use this to start services, install dependencies, and run migrations.
               Export env vars via `echo "VAR=value" >> "$GITHUB_ENV"` — they persist
               to Claude and all subsequent steps. Only the simple `VAR=value` form
               is supported; GitHub Actions' heredoc multiline syntax (`VAR<<EOF`) is
               NOT — such lines are ignored with a warning.
    verify     Path to a shell script that runs AFTER Claude, only on success.
               Use this to run tests or smoke checks.
    teardown   Path to a shell script that runs AFTER Claude, even on failure.
               Use this to stop containers or clean up resources.

    NOTE: Hooks run only in GitHub Actions execution mode. On Fly/local modes the
    workspace is empty when WORKFLOW.md is read (the repo is cloned later), so
    setup/verify/teardown are silently skipped.

  SETUP AND TEARDOWN HOOKS
  ------------------------
  Repos that need a database or other services should define scripts instead of
  relying on the workflow-level `services:` block. GitHub-hosted runners have
  Docker available, so start containers with `docker run -d` in your setup script.

  Example front matter:
    setup:    scripts/ci/ai-setup.sh
    verify:   scripts/ci/ai-verify.sh
    teardown: scripts/ci/ai-teardown.sh

  Example setup script (Django + PostgreSQL):
    #!/usr/bin/env bash
    set -euo pipefail
    docker run -d --name postgres \
      -e POSTGRES_DB=app -e POSTGRES_USER=app -e POSTGRES_PASSWORD=app \
      -p 5432:5432 postgres:16
    for i in $(seq 1 30); do
      docker exec postgres pg_isready -q && break
      [ "$i" -eq 30 ] && { docker logs postgres; exit 1; }
      sleep 1
    done
    echo "DATABASE_URL=postgresql://app:app@localhost:5432/app" >> "$GITHUB_ENV"
    echo "DJANGO_SETTINGS_MODULE=config.settings_ci" >> "$GITHUB_ENV"
    echo "DJANGO_SECRET_KEY=ci-secret-key-not-for-production" >> "$GITHUB_ENV"
    pip install -r django/requirements.txt
    cd django && python manage.py migrate_schemas --shared
    python manage.py create_public_tenant --domain_url=localhost

  Example teardown script:
    #!/usr/bin/env bash
    set -euo pipefail
    docker stop postgres && docker rm postgres || true

  NEW IMPLEMENTATION vs GAP-FILL RUNS
  -------------------------------------
  When ${PR_NUMBER} is empty  → Claude creates a new branch and PR.
  When ${PR_NUMBER} is set    → Claude pushes gap-fill commits to the existing PR.
  Both scenarios use this same file. The conditional sections below handle both.

  HOW TO CUSTOMISE THIS FILE
  ---------------------------
  1. Fill in the "Repo context" section with your stack, test commands, conventions.
  2. Adjust the quality checklist to match your standards.
  3. Add any repo-specific constraints (e.g. "never modify migration files directly").
  4. Change the model in the front matter if this repo needs more (opus) or less (haiku).
  5. Remove these HTML comments once you're done — Claude won't see them anyway.

  CLIENT-SPECIFIC CODE: USE custom/
  -----------------------------------
  This repo uses a path-precedence extension mechanism. When implementing
  client-specific behaviour, place new files under custom/ rather than
  modifying built-in modules:

    custom/steps/<id>.ts       — override a built-in pipeline step
    custom/pipelines/<name>.yml — override a built-in pipeline definition
    custom/providers/<id>.ts   — override a built-in provider module

  Files in custom/ are never overwritten by upstream syncs, so they survive
  upgrades. Editing built-in modules directly causes merge conflicts on every
  upstream update. See CLAUDE.md §"Custom extensions" for the full contract.
-->

Read CLAUDE.md if it exists for repo-specific context and conventions.

---

## New implementation

Implement the feature described in the issue below in the current checkout.
Do NOT create or switch branches. Do NOT commit, push, or open a pull request.
Leave your file changes unstaged and uncommitted. The AI-Implement pipeline
will create the implementation commit, push an issue-scoped branch, and open
the PR after review passes. The generated PR body includes
`Fixes ${ISSUE_IDENTIFIER}` so the ticketing system automatically closes the
issue when the PR is merged (Linear behaviour; Jira ignores it harmlessly).

After making the code changes, write a brief implementation summary to
`ai-output/comments/01-summary.md` (e.g. a paragraph describing what
changed plus a checklist of what was tested). The orchestrator reads this
file and posts it back to the ticketing issue via the configured provider.
Do NOT post comments directly to Linear or Jira from this workflow —
that pathway is handled by the orchestrator's runner-callback.

---

## Gap-fill instructions _(only when PR_NUMBER is set)_

You are adding missing work to existing PR #${PR_NUMBER}.
**Do NOT create a new branch or PR.** Commit your changes to the current
branch and push. Review the gap analysis comment on the PR to understand
what is still missing.

After your changes are pushed, write a short note about what you addressed
to `ai-output/comments/01-gap-fill-summary.md`. The orchestrator reads
this file and posts it back to the ticketing issue.

External review tools should communicate findings through native GitHub
review surfaces: submit `CHANGES_REQUESTED` for blocking feedback, use inline
PR review comments for file-specific issues, or post a structured PR review
summary comment. Do not ask Copilot or another bot to fix the PR in comments;
AI-Implement ingests GitHub review events and dispatches its own gap-fill run.

---

## Issue

**Identifier:** ${ISSUE_IDENTIFIER}
**Title:** ${ISSUE_TITLE}
**Description:**
${ISSUE_DESCRIPTION}

${PLANNING_CONTEXT}

---

## Repo context

This is the **AI-Implement documentation site** — Mintlify MDX, not application code. There is no
build, no `package.json`, and no test suite; "implementing" an issue means editing `.mdx` docs pages.

- **Site:** Mintlify. Config and navigation live in `docs.json`. Pages are `.mdx` (a few `.md`).
- **Versioning (site-wide):** two versions share the repo —
  - **stable** (the default): root-level pages — `introduction` / `quickstart` / `how-it-works` plus
    the `setup/`, `configuration/`, `providers/`, `customize/`, `reference/` directories.
  - **latest** (in-development): the same tree mirrored under `latest/`, plus `latest/changelog.mdx`.
- **Choosing which version to edit** (in priority order):
  1. **Explicit in the issue** — the issue names a version ("latest" / "stable"), links or names a
     page path (a `latest/…` path → latest; a root path like `reference/…` → stable), or cites an
     AI-Implement branch: **`testing` → latest**, **`main` / "released" → stable**.
  2. **Otherwise, let the content decide** — find the affected page/section in the repo:
     - only under `latest/` → edit latest; only at root → edit stable;
     - present in **both** trees and the change is true in both (a correction / clarification) →
       edit **both**, so the versions don't drift.
  3. **Last resort** — if still ambiguous, edit **stable** (what most readers see) AND state the
     assumption in `ai-output/comments/01-summary.md` so the reviewer can redirect.
- **Out of bounds:** do not touch `.github/` (the docs-audit workflows are a separate system),
  `WORKFLOW.md` / `PLANNING.md` / `custom/` / `.mintignore` (AI-Implement plumbing), or `docs.json`
  unless the issue is explicitly about navigation.
- **No build/test, and no Mintlify CLI.** `mint` is **not** installed in this environment and you
  don't need it — verify links and anchors by reading the target page (does the heading / anchor
  exist?), not by running a command. Don't install it; `mint broken-links` / `mint validate` are for
  a human to run locally.

---

## Documentation standards

Write the docs the way a reader uses them, not the way the code is built.

**Reader-focused.** Describe user-relevant behavior in user-relevant language. Do NOT put into the
published `.mdx`: source file paths or `file:line` references, internal/code identifiers (type,
interface, function, or variable names), internal state names, or Mintlify component names spelled
out in prose ("uses the Note component"). The reader doesn't have the codebase open.

**Pick the component by intent** (don't default to `<Note>`):
- `<Note>` — a neutral but important fact
- `<Tip>` — an operator benefit, or a "you can skip the hard way" relief
- `<Warning>` — a blocking issue; something breaks if ignored
- `<Info>` — permissions or context-setting
- `<Check>` — a success state / confirmation

**Use `<Steps>` for ordered procedures.** Any sequence where each action depends on the previous one
completing — install → restart → verify, a setup walkthrough — goes in a `<Steps>` block, not a bare
numbered list or a run of separate paragraphs. The reader is following along as they go; the
do-this-then-that structure is the point.

**Enumerate completely.** When the issue asks you to list a set — every command, every item in a
suite — list the **whole** set, not a representative two or three. If the set differs by version (more
items in the in-development version than the released one), list each version's actual set on its own
page. If you can't determine the full set from the issue or the repo, list what you can and flag the
gap in your summary — don't silently truncate.

**Keep it scannable.** A `<ParamField>` (or any component body) covering what-it-does +
accepted-values + default + when-to-use should be 2–4 short paragraphs with blank lines between them,
not one dense block. Use bullet lists for parallel items (accepted values, file lists).

**Cross-links and anchors.** Verify the destination anchor exists before linking. Slug rules:
- `##`/`###`/`####` headings, `<Accordion title="…">`, `<Update label="…">` → `#<slugified-text>`
- `<ParamField body="X">` → `#param-<slugified-x>` (note the `#param-` prefix)
- `<Step title="…">` and `<Tab title="…">` generate **no anchor** — link to the page instead
- A `/` inside a heading stays in the slug and must be URL-encoded as `%2F` in the link
  (e.g. a "Projects (team/repo mappings)" heading → `#projects-team%2Frepo-mappings`)
- On a `latest/…` page, every cross-link to another docs page must include the `/latest/` prefix —
  in **both** `](/path)` Markdown links **and** `href="/path"` component props (Cards, Buttons). An
  unprefixed link on a `latest/` page silently sends the reader to the stable copy instead.

**Internal-only changes.** If a change has no reader-visible effect (e.g. an internal refactor that
doesn't alter what an operator does), it belongs in `latest/changelog.mdx` as an `<Update>` entry —
not scattered across pages; on stable it's usually nothing at all.

**Avoid:**
- Source paths or `file:line` in the published docs.
- Internal jargon (code/type/state names; component names in prose).
- An unprefixed cross-link on a `latest/` page.
- Linking to a `<Step>` / `<Tab>` title (no anchor exists).
- Inventing pages or documenting features that don't exist — every claim must be true of the product.
- Cosmetic rewrites of text that's already correct, or changes to pages the issue didn't ask about (see the Scope section).
- Rendering a dependent, ordered procedure as a bare numbered list instead of `<Steps>`.
- Listing a partial or "representative" subset when the issue asks to enumerate a full set.

---

## Scope

Create or update **only** the documentation this issue describes. This is a targeted edit, **not a
broad audit** — do not reword text that already reads correctly, reformat or reorganize sections, or
"fix drift" in pages the issue doesn't mention, and don't expand beyond what's asked.

If you notice an unrelated problem while working, **don't fix it** — add a short "Noticed (out of
scope)" note to your `ai-output/comments/01-summary.md` summary so a human can triage it.

---

## Quality checklist

Before finishing, verify:

- [ ] Edits limited to what the issue asked — no unrelated pages, no rewording of text that's already correct
- [ ] Correct version target (stable root vs `latest/**`), per the version rule above
- [ ] Reader-focused: no source paths / `file:line` / internal or code names / component names in prose
- [ ] Components chosen by intent (not defaulting to `<Note>`)
- [ ] Cross-links resolve: anchor exists; `#param-` prefix where needed; `/latest/` prefix on any
      `latest/` page (both `](…)` links and `href=` props); no links to `<Step>` / `<Tab>` titles
- [ ] New or edited pages have `title` (and ideally `description`) frontmatter
- [ ] Summary written to `ai-output/comments/01-summary.md`
- [ ] Ordered/dependent procedures use `<Steps>`; any "list the set" is complete (or flags the gap), correct per version
