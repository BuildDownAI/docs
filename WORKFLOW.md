---
# Claude model used for implementation. Passed through verbatim to
# `claude-code --model`, so any ID your configured provider accepts is fine.
# Examples:
#   Anthropic API / OAuth: claude-sonnet-4-6, claude-opus-4-8, claude-haiku-4-5-20251001
#   AWS Bedrock:           anthropic.claude-sonnet-4-6-20250805-v1:0
#                          or an inference-profile ARN (arn:aws:bedrock:...)
# The default below works for the Anthropic provider. If this repo's mapping
# is switched to provider=bedrock in the orchestrator admin UI, replace this
# with a Bedrock model ID — nothing validates the value, so an Anthropic ID
# passes straight through and the run fails at invocation. Bedrock IDs are
# account and region specific, so there is no safe default to fall back on.
model: claude-opus-4-8
---

**Read `CLAUDE.md` first, and follow it.** It is the shared source of truth for what this repo is, which version to edit, and how the documentation should read. Those rules are not repeated here — this file covers only what is specific to implementing a tracker issue.

---

## New implementation

Implement the feature described in the issue below in the current checkout. Do NOT create or switch branches. Do NOT commit, push, or open a pull request. Leave your file changes unstaged and uncommitted. The AI-Implement pipeline will create the implementation commit, push an issue-scoped branch, and open the PR after review passes. The generated PR body includes `Fixes ${ISSUE_IDENTIFIER}` so the ticketing system automatically closes the issue when the PR is merged (Linear behaviour; Jira ignores it harmlessly).

After making the changes, write a brief implementation summary to `ai-output/comments/01-summary.md` — a paragraph describing what changed, plus anything a reviewer should check. The orchestrator reads this file and posts it back to the ticketing issue via the configured provider. Do NOT post comments directly to Linear or Jira from this workflow; that pathway is handled by the orchestrator's runner-callback.

---

## Gap-fill instructions _(only when PR_NUMBER is set)_

You are adding missing work to existing PR #${PR_NUMBER}. **Do NOT create a new branch or PR.** Commit your changes to the current branch and push. Review the gap analysis comment on the PR to understand what is still missing.

After your changes are pushed, write a short note about what you addressed to `ai-output/comments/01-gap-fill-summary.md`. The orchestrator reads this file and posts it back to the ticketing issue.

External review tools should communicate findings through native GitHub review surfaces: submit `CHANGES_REQUESTED` for blocking feedback, use inline PR review comments for file-specific issues, or post a structured PR review summary comment. Do not ask Copilot or another bot to fix the PR in comments; AI-Implement ingests GitHub review events and dispatches its own gap-fill run.

---

## Issue

**Identifier:** ${ISSUE_IDENTIFIER}
**Title:** ${ISSUE_TITLE}
**Description:**
${ISSUE_DESCRIPTION}

${PLANNING_CONTEXT}

---

## Choosing which version to edit

`CLAUDE.md` describes the two versions and the source branch each one tracks. Pick between them in this order:

1. **The issue says so** — it names a version, links or names a page path (a `latest/…` path means latest, a root path like `reference/…` means stable), or cites a source branch.
2. **The content decides** — find the affected page or section in the repo. Present only under `latest/` → edit latest. Present only at root → edit stable. Present in **both** trees, and the change is true of both (a correction or a clarification) → edit **both**, so the versions don't drift apart.
3. **Last resort** — if it is still ambiguous, edit **stable**, since that is what most readers see, and state the assumption in `ai-output/comments/01-summary.md` so the reviewer can redirect it.

---

## Scope

Create or update **only** the documentation this issue describes. This is a targeted edit, **not a broad audit** — do not reword text that already reads correctly, reformat or reorganize sections, or "fix drift" in pages the issue doesn't mention.

Creating a new page is in scope when the issue asks for one. Give it `title` and `description` frontmatter and add it to `docs.json` under the product and version it belongs to. A new page is unreachable without that navigation entry, which is the one case where editing `docs.json` is expected rather than out of bounds.

If you notice an unrelated problem while working, **don't fix it** — add a short "Noticed (out of scope)" note to `ai-output/comments/01-summary.md` so a human can triage it.

---

## Quality checklist

Before finishing, verify:

- [ ] Edits limited to what the issue asked — no unrelated pages, no rewording of text that is already correct
- [ ] Correct version target, per the rule above
- [ ] The shared rules in `CLAUDE.md` were followed
- [ ] Any new page has `title` and `description` frontmatter and a `docs.json` navigation entry
- [ ] Summary written to `ai-output/comments/01-summary.md`
