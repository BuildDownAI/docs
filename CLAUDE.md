# docs — AI-Implement + BuildDown bindings

This repo is an AI-Implement target (Linear issues → documentation PRs) and uses the
BuildDown skills for issue planning. **This file holds bindings only** — it maps the BuildDown
skills' placeholders to this project's concrete tools, and is read by the skills and by the
AI-Implement runner.

**`WORKFLOW.md` is the source of truth for documentation work.** How to write the docs
(reader-focused MDX, components, anchors), which version to edit, scope, and the quality checklist
all live there. Do **not** restate those rules here — this file only supplies what `WORKFLOW.md`
doesn't: the tracker / repo / label bindings below.

## Issue tracker — Linear
- tracker.kind: linear
- MCP server: `linear-builddownai-docs` (declared in `.mcp.json`, pre-approved in `.claude/settings.json`)
- Workspace: `eudoxus` (bound at OAuth time)
- Team: `Documentation` (key `DOC`) — subteam of `AI-Implement`; docs issues filed/listed/searched here
- Team URL: https://linear.app/eudoxus/team/DOC/overview

## GitHub
- Repo (PRs land here): `BuildDownAI/docs`
- MCP server: `github` (declared + pre-approved; OAuth deferred until bd-build-down needs it)

## AI-Implement pickup
- Label the orchestrator dispatches on: `AI-Implement`
- PR-comment mention that re-triggers the agent: `/ai-implement`

## Skill bindings not used by this project
- bd-smoke-jumper preview-host / preview-auth — N/A (no app preview to log into).
- Build/verify command — none (no `package.json`). How the runner treats checks is covered in `WORKFLOW.md`.
