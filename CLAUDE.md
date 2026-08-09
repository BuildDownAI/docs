# BuildDown Docs

The Mintlify documentation site for **BuildDown**, covering two products: **AI-Implement** (the orchestration service that turns tracker issues into pull requests) and the **BuildDown skills** plugin.

**This file is the shared single source of truth for documentation work** — what the repo is, which version to edit, how the docs should read, and what is out of bounds. Two lanes write MDX to these pages, and both read this file:

| Lane | Trigger | Lane file |
|---|---|---|
| AI-Implement | a DOC issue labeled `AI-Implement` dispatches a runner | `WORKFLOW.md` |
| Audit | a scheduled or manually dispatched GitHub Actions run | `.github/audit-reference.md` |

Each lane file covers only its own mechanics — issue handling and gap-fill on one side, the priority rubric and finding shape on the other — and points here for everything else. **A rule stated here is not restated in a lane file.** One home per rule is what lets an improvement made in either lane reach both.

## What this repo is

- A Mintlify site. Navigation, versions, and theme live in `docs.json`; pages are `.mdx`.
- **No build, no `package.json`, no test suite.** Changes here are edits to documentation pages, nothing more.
- Two products, navigated separately. Each version pill contains an **AI-Implement** tab and a **Skills** tab, and each tab renders its own sidebar.

## Versioning

The site uses Mintlify site-wide versioning, so two versions share one tree:

- **stable** — the root-level pages: `introduction.mdx`, `quickstart.mdx`, `how-it-works.mdx`, `releases.mdx`, and the `setup/`, `configuration/`, `providers/`, `customize/`, `reference/`, and `skills/` directories. This is the default version and what most readers see.
- **latest** — the same tree mirrored under `latest/`.

Both products track the same split in their source repos: **`main` → stable**, **`testing` → latest**. This holds for AI-Implement and for skills alike.

`snippets/` is shared across both versions. Touch it only for a change that is genuinely cross-version.

**Which version a given change lands in is decided differently by each lane, and each lane file states its own rule** — the audit lane's scope is fixed by the branch it was dispatched against, while the AI-Implement lane derives it from the issue. Follow your own lane's rule; don't assume the other's applies.

## Out of bounds

Do not edit: `.github/` (the audit lane is a separate system), `WORKFLOW.md`, `PLANNING.md`, `AGENTS.md`, this file, `.mintignore`, or `docs.json` unless the work is explicitly about navigation.

Neither automated lane has `mint` installed, and neither needs it — verify a link or an anchor by reading the target page and confirming the heading exists, not by running a command. Don't install it.

Working locally is the other case: `mint dev`, `mint validate`, and `mint broken-links` are available there and are the right check to run before a change ships.

## How the docs should read

Write the docs the way a reader uses them, not the way the product is built.

### Reader focus

**No internal vocabulary in published MDX.** Drop source paths and `file:line` citations, internal identifiers (type, interface, function, variable, or state names), and Mintlify component names spelled out in prose ("uses the Note component"). The reader doesn't have the codebase open. Describe user-relevant behavior in user-relevant language.

**Don't document baseline-expected behavior.** If a reader would assume it without being told — that a Docker command needs Docker running, that a local process doesn't notify a team channel — stating it costs attention and buys nothing. This also means: don't invent setup steps for controls a vendor doesn't expose.

**Don't restate what another section or page already owns.** Before adding a summary table or a reassurance paragraph, check whether each fact is already stated at its natural home. If it is, the copy is maintenance burden that will drift — and it is the copy that drifts, because the natural home is what gets updated. When a fact legitimately belongs in two places, keep it at the **point of action** (the field the reader fills in) and drop it at the point of consequence.

**Provider-neutral phrasing on shared surfaces.** Anything covering both ticketing providers uses a neutral concept plus a parenthetical naming both. "Label" is Linear's word and "status field" is Jira's — a sentence built on either excludes half the readers.

**"Optional" needs a consequence clause.** When omitting a setting silently disables documented behavior, say what is lost, at the point where the reader decides. "Optional" alone reads as "safe to skip."

### Structure

**Break parallel items into a list.** Three or more items of the same kind in a sentence — accepted values, file names, prerequisites — become a bulleted list, anywhere on the page. This applies to ordinary prose, accordion bodies, and component bodies alike, not just parameter descriptions.

**One beat per paragraph.** A beat is one thing the reader learns: what a field does, how it behaves in an edge case, what it defaults to, why they would change it. Before finishing any block of prose, count its beats — if there is more than one, put a blank line at each boundary. Two tight sentences carrying two beats are still two paragraphs.

Two tells that a boundary is being papered over: an em-dash or semicolon joining two complete thoughts, and a sentence that opens by restating the subject of the one before it. Both mean the split was already there and got written as punctuation instead.

This governs every block of prose on a page — ordinary paragraphs, accordion bodies, `<Step>` bodies, and `<ParamField>` bodies alike.

**Use `<Steps>` for ordered procedures.** Any sequence where each action depends on the previous one completing — install → restart → verify, a setup walkthrough — goes in a `<Steps>` block, not a bare numbered list or a run of separate paragraphs. The reader is following along as they go.

**Enumerate completely.** When listing a set — every command, every item in a suite — list the **whole** set, not a representative two or three. If the set differs by version, list each version's actual set on its own page. If you can't determine the full set, list what you can and say so rather than silently truncating.

**Don't scaffold examples.** In a `**term** — examples` list the dash already signals that what follows are examples, so list them directly: `**Cloud credentials** — an AWS credentials file, a GCP service-account JSON`, not `— for example an AWS credentials file …`.

### Components

**Pick the component by intent** — don't default to `<Note>`:

- `<Note>` — a neutral but important fact
- `<Tip>` — an operator benefit, or a "you can skip the hard way" relief
- `<Warning>` — a blocking issue; something breaks if ignored
- `<Info>` — permissions or context-setting
- `<Check>` — a success state or confirmation

**Callouts must earn their box, and adding one is often a swap.** Before adding a callout, count the components already in the enclosing step, tab, or section. If yours would make three or more, rank them by how *surprising* each is to a reader who has read the surrounding prose — keep the least predictable one boxed and write the rest as prose. Baseline prerequisites lose that ranking every time. The test for whether something belongs in a box at all: could a reader delete it and still follow the section? If not, it is the argument — write it as prose with bold for weight.

**`<ParamField>` defaults belong in the `default` prop**, not in body prose. Always quoted (`default="90"`), since the value may be conditional (`default="3 (Anthropic), 2 (Bedrock)"`), which `{}` cannot express.

### Cross-links and anchors

Verify the destination anchor exists before linking. Slug rules:

- `##`/`###`/`####` headings, `<Accordion title="…">`, `<Update label="…">` → `#<slugified-text>`
- `<ParamField body="X">` → `#param-<slugified-x>` (note the `#param-` prefix)
- `<Step title="…">` and `<Tab title="…">` generate **no anchor** — link to the page instead
- A `/` inside a heading stays in the slug and must be URL-encoded as `%2F` in the link — a "Projects (team/repo mappings)" heading becomes `#projects-team%2Frepo-mappings`

On a `latest/…` page, every cross-link to another docs page must carry the `/latest/` prefix — in **both** `](/path)` Markdown links **and** `href="/path"` component props such as `<Card>`. An unprefixed link on a `latest/` page silently sends the reader to the stable copy. A prior sweep that handled only Markdown links left twelve `href=` props unprefixed across five pages.

### What never becomes a page edit

**Internal-only changes.** A change with no operator-visible behavior — an internal refactor, a database column, an auth-plumbing shift, a telemetry foundation — is not a per-page edit on either version. Release notes are written when a version ships, from the whole set of changes in it.

**Claims you haven't verified.** Every claim must be true of the product. Confirm the behavior at its source before writing it down — don't infer a feature from a configuration key, a name, or what an adjacent page implies.

**Cosmetic rewrites of text that is already correct.**

## Tool bindings

This repo is an AI-Implement target and uses the BuildDown skills for issue planning. The bindings below map those skills' placeholders to concrete tools.

### Issue tracker — Linear

- tracker.kind: linear
- MCP server: `linear-builddownai-docs` (declared in `.mcp.json`, pre-approved in `.claude/settings.json`)
- Workspace: `eudoxus` (bound at OAuth time)
- Team: `Documentation` (key `DOC`) — subteam of `AI-Implement`; docs issues filed/listed/searched here
- Team URL: https://linear.app/eudoxus/team/DOC/overview

### GitHub

- Repo (PRs land here): `BuildDownAI/docs`
- MCP server: `github` (declared + pre-approved; OAuth deferred until bd-build-down needs it)

### AI-Implement pickup

- Label the orchestrator dispatches on: `AI-Implement`
- PR-comment mention that re-triggers the agent: `/ai-implement`

### Skill bindings not used by this project

- bd-smoke-jumper preview-host / preview-auth — N/A (no app preview to log into).
- Build/verify command — none. How each lane treats verification is covered above.
