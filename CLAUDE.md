# BuildDown Docs

The Mintlify documentation site for **BuildDown**, covering two products: **AI-Implement** (the orchestration service that turns tracker issues into pull requests) and the **BuildDown skills** plugin.

**This file is the shared single source of truth for documentation work** — what the repo is, which version to edit, how to verify a claim against source, how the docs should read, and what is out of bounds. Two lanes write MDX to these pages, and both read this file:

| Lane | Trigger | Lane file |
|---|---|---|
| AI-Implement | a DOC issue labeled `AI-Implement` dispatches a runner | `WORKFLOW.md` |
| Audit | a scheduled or manually dispatched GitHub Actions run | `.github/audit-reference.md` |

Each lane file covers only its own mechanics — issue handling and gap-fill on one side, the priority rubric and finding shape on the other — and points here for everything else. **A rule stated here is not restated in a lane file.** One home per rule is what lets an improvement made in either lane reach both.

## What this repo is

- A Mintlify site. Navigation, versions, and theme live in `docs.json`; pages are `.mdx`.
- **No build, no `package.json`, no test suite.** Changes here are edits to documentation pages, nothing more.
- **Two products, versioned independently.** `docs.json` nests versions inside products, so **AI-Implement** and **Skills** each carry their own version selector and their own sidebar.

## Versioning

Versions are nested inside products, so each product carries its own two versions across one shared tree:

- **stable** — the root-level pages. AI-Implement's are `introduction.mdx`, `quickstart.mdx`, `how-it-works.mdx`, `releases.mdx`, and the `setup/`, `configuration/`, `providers/`, `customize/`, and `reference/` directories; the Skills product's are `skills/`. This is each product's default version and what most readers see.
- **latest** — the same trees mirrored under `latest/`.

Each version describes a different state of both source repos:

- **stable** describes each product's current release — its latest release tag, not the tip of `main`, which can carry commits no release has shipped.
- **latest** describes the source repos' `testing` branches.

A version pill in the navigation shows what that version's pages have been brought up to, and it can trail the release while corrections are pending. Never read the pill as the version to write for.

**Write each version's pages as though they were the only ones.** A reader has one tree in front of them, so "on this version", "in this release" or "unlike the other version" asks for a comparison they cannot make. State what is true and let the other tree state its own truth. You will have both checkouts open while writing; they will not.

**The two products' version numbers are not comparable.** AI-Implement's track release cadence; the skills plugin's track delivery, since a plugin change reaches nobody without a version bump. Never write prose implying one product is ahead of, behind, or in step with the other.

`snippets/` is shared across both versions. Touch it only for a change that is genuinely cross-version.

**Which version a given change lands in is decided differently by each lane, and each lane file states its own rule** — the audit lane is told its scope by its dispatch, while the AI-Implement lane derives it from the issue. Follow your own lane's rule; don't assume the other's applies.

## Out of bounds

Do not edit: `.github/` (the audit lane is a separate system), `WORKFLOW.md`, `PLANNING.md`, `AGENTS.md`, this file, `.mintignore`, or `docs.json` unless the work is explicitly about navigation.

Neither automated lane has `mint` installed, and neither needs it — verify a link or an anchor by reading the target page and confirming the heading exists, not by running a command. Don't install it.

Working locally is the other case: `mint dev`, `mint validate`, and `mint broken-links` are available there and are the right check to run before a change ships.

## Search locates, reading verifies

**Search locates; `Read` verifies.** A match from a search — `grep`, `rg`, `find`, or a dedicated search tool where the run has one — is a candidate, never a confirmation.

This holds for everything you check: a source file, an existing docs page, a link target, an anchor.

Before stating anything about what a search found, `Read` the whole enclosing function, section, or block — or the whole file, when it is short.

Three search results pass for verification and are not:

- a match count standing in for the lines it counted
- no match, from a search narrower than the claim — a file `Read` only in part, a truncated result, or a filter that excluded more than intended
- a match on wording you chose, which shows your phrasing appears, not that the claim is true

## Verifying against source

Both lanes run with the source repositories checked out beside the docs. Each repository has two fixed paths, one for each docs version:

| Source | Confirms stable (current release) | Confirms latest (`testing`) |
|---|---|---|
| AI-Implement | `./ai-implement/` | `./ai-implement-testing/` |
| BuildDown skills | `./skills-source/` | `./skills-source-testing/` |

All four are read-only reference material. Never edit them.

**Confirm a claim only against the checkout for the version you are writing.** A stable claim confirmed against a `-testing` checkout can describe behavior no release has shipped, and a latest claim confirmed against a release checkout can miss what `testing` has changed.

**A run does not always carry every checkout.** When the one for your version is absent, you cannot confirm that version's claims — say so rather than confirm against the other.

**Each checkout carries its own `CLAUDE.md`, and it enters your context the first time you read a file there.** Those files describe their own codebases, which helps in finding where a behavior lives.

They also bind their own repos' issue-tracker teams, labels, and handoffs. None of that applies to documentation work in this repo — this file governs.

### How to check a claim

**Diff environment-variable docs against their canonical sources, in both directions.** Two machine-readable sources are maintained alongside the code and usually run ahead of the docs: `.env.example` for orchestrator variables, and the `# Optional repository or organization variables:` header block at the top of each synced workflow for target-repo variables. Compare the documented set against both:

- In a source but not the page → an undocumented variable
- On the page but not in a source → a removed or renamed variable
- In both, but the source comment says more than the page → the page is stale on semantics

The third direction is the most valuable and the easiest to skip. Asking whether a variable appears in source finds a stale reference and confirms it, while a set difference cannot be satisfied by one confirming hit.

**A key being accepted is not the same as a key being used.** Before documenting a configuration value, check separately that a parser accepts it and that something outside the parser reads it. A key can sit on an interface and in an accepted-key list while nothing consumes it, round-tripping silently. A documented default that appears only in test fixtures is strong evidence the consumer is gone.

**Read the guard clauses, not just the happy path.** An early return above the logic you are reading may fire routinely rather than rarely. Ask which path is typical, and whether an external mechanism races the one you are describing.

**In troubleshooting content, the cause may be inferred but the remediation must be verified.** A remediation names something the reader operates, so check:

- the exact label the reader sees, not the internal field name
- which surface exposes it — the creation flow, the edit dialog, a config file, or an environment variable
- whether it is configurable at all, and whether the surface can express what you are asking of it

The surfaces diverge, and readers usually reach the docs during the creation flow, so check that flow first. Inherited text gets the same check: an instruction to preserve a sentence reads as an assurance that someone already established it, which is exactly when nobody has.

## How the docs should read

Write the docs the way a reader uses them, not the way the product is built.

### Reader focus

**No internal vocabulary in published MDX.** Drop source paths and `file:line` citations, internal identifiers (type, interface, function, variable, or state names), and Mintlify component names spelled out in prose ("uses the Note component"). The reader doesn't have the codebase open. Describe user-relevant behavior in user-relevant language.

**Reuse the words the page already uses.** Before introducing a term for something the corpus already names, search for it. A second word for one concept makes a reader stop to work out whether two things are meant — *routes* beside *endpoints*, or a credential named one way in its reference and another on the page that tells you to create it. A coined term reads naturally to whoever coined it, which is why this needs a search rather than a judgment.

**Don't document baseline-expected behavior.** If a reader would assume it without being told — that a Docker command needs Docker running, that a local process doesn't notify a team channel — stating it costs attention and buys nothing. This also means: don't invent setup steps for controls a vendor doesn't expose.

**Don't restate what another section or page already owns.** Before writing a fact, check whether it is already stated at its natural home — a summary table and a reassurance paragraph are the usual shapes, and a correction landing on several pages is the one that catches writers out. If the fact is already home, the copy is maintenance burden that will drift, and it is the copy that drifts, because the natural home is what gets updated. When a fact legitimately belongs in two places, keep it at the **point of action** (the field the reader fills in) and drop it at the point of consequence.

**Re-read the page after editing it, not only the edit.** Changing what a page asserts about a set can falsify a sentence nowhere near the change — a count, an enumeration, a summary of what the set used to be. Removing content leaves its own residue: a list with one item, an example that no longer contrasts with anything, a callout guarding against something the page no longer describes. Neither is a wrong claim *about* the change, which is why both survive review. After changing or removing anything, search the page for whatever counted, enumerated, or characterized what you touched, and read the block a removal left behind. Repairing what your own edit broke is not widening its scope.

**One change, several pages: pick the owner first.** Correcting a claim everywhere it appears is required; writing the same corrected sentence everywhere it appears is not. Decide which page owns the mechanism before drafting — that page carries the explanation, and every other page carries only what its own reader needs in order to keep reading, plus a link. The tell is a sentence you are about to write for the second time on a different page.

**Provider-neutral phrasing on shared surfaces.** Anything covering both ticketing providers uses a neutral concept plus a parenthetical naming both. "Label" is Linear's word and "status field" is Jira's — a sentence built on either excludes half the readers.

**"Optional" needs a consequence clause.** When omitting a setting silently disables documented behavior, say what is lost, at the point where the reader decides. "Optional" alone reads as "safe to skip."

### Structure

**Break parallel items into a list.** Three or more items of the same kind in a sentence — accepted values, file names, prerequisites — become a bulleted list, anywhere on the page. This applies to ordinary prose, accordion bodies, and component bodies alike, not just parameter descriptions.

**One beat per paragraph.** A beat is one thing the reader learns: what a field does, how it behaves in an edge case, what it defaults to, why they would change it. Before finishing any block of prose, count its beats — if there is more than one, put a blank line at each boundary. Two tight sentences carrying two beats are still two paragraphs.

Two tells that a boundary is being papered over: an em-dash or semicolon joining two complete thoughts, and a sentence that opens by restating the subject of the one before it. Both mean the split was already there and got written as punctuation instead.

This governs every block of prose on a page — ordinary paragraphs, list items, accordion bodies, `<Step>` bodies, and `<ParamField>` bodies alike. In a list item the fix is a second item rather than a blank line: a bullet carrying two beats is two bullets written as one.

**Splitting has an opposite failure, and a run of one-sentence paragraphs is its tell.** Facts the reader takes in together are one beat however separably they can be stated — what a field does and what it defaults to, a fact and the pointer to the section that owns its detail. Before splitting, ask whether either half is usable on its own: if it is not, you have one beat written as two. This applies wherever prose does, and it is the half of this rule that gets missed, because a draft that is too fragmented still looks like it obeyed the instruction.

**Use `<Steps>` for ordered procedures.** Any sequence where each action depends on the previous one completing — install → restart → verify, a setup walkthrough — goes in a `<Steps>` block, not a bare numbered list or a run of separate paragraphs. The reader is following along as they go.

**Order a troubleshooting entry context → cause → resolution → what obviously doesn't work.** Context is what should have happened and what the reader saw instead, including the absence of any error. Keep provider-specific behavior together in the cause rather than scattered through the other segments. Name the plausible fix that fails, and why, whenever one exists — a reader often arrives having already tried it.

Those four segments are the entry's beats: each is one move the reader makes, so each is one paragraph however many facts it rests on. Mechanism is not a beat here, and the page that documents it keeps the full account — but the entry still has to stand on its own. Carry as much of the mechanism as the fix depends on and link for the rest, because a reader who must leave the page to finish the fix has been handed a detour rather than a resolution.

**Enumerate completely.** When listing a set — every command, every item in a suite — list the **whole** set, not a representative two or three. If the set differs by version, list each version's actual set on its own page. If you can't determine the full set, list what you can and say so rather than silently truncating.

**Before finishing, check the pages your own edit's vocabulary points at.** Some subjects are split across two pages by convention, so changing one leaves the other stale, and the page owning the other half is rarely the one you were editing. Two moves cover it, and neither needs you to already know which pages are paired:

- **An exact term** — a status value, an error code, a configuration key — is searchable. Search every `.mdx` file for it. Where it already appears, check whether your change made that copy wrong or incomplete.
- **A general one** — a notification, a label, a run status — is not. List the pages under `reference/` and open any whose name matches the thing you mentioned. Those pages are named after what they enumerate, so a sentence saying the orchestrator sends an alert has `reference/notifications.mdx` to answer to, whether or not you thought of that page.

Where the term belongs on the page you open and is missing, that absence is the half you left out. Each page still keeps its own half — the admin UI reference lists endpoints and never their status codes, which the error-codes reference owns — so what you are checking is whether the other half has to move with your change, not whether to copy it across.

**Delete the number in front of a list.** A count beside its own enumeration states one fact twice, and the number is the half that rots — the next edit that adds a member makes it false. Giving a size *instead* of the members is the same rule from the other side, and refuses the reader the membership entirely.

Name the members, or name the set by its shape — *every planning and landing skill, plus `bd-project-setup`*. Keep a number only when it carries something the list cannot, such as a split the sentence is about.

Before finishing, re-read each paragraph you wrote from scratch and check whether a number is sitting in front of a list. This rule misses far more often on new prose than on an edit to existing prose.

**Don't scaffold examples.** In a `**term** — examples` list the dash already signals that what follows are examples, so list them directly: `**Cloud credentials** — an AWS credentials file, a GCP service-account JSON`, not `— for example an AWS credentials file …`.

### Components

**Pick the component by intent** — don't default to `<Note>`:

- `<Note>` — a neutral but important fact
- `<Tip>` — an operator benefit, or a "you can skip the hard way" relief
- `<Warning>` — a blocking issue; something breaks if ignored
- `<Info>` — permissions or context-setting
- `<Check>` — a success state or confirmation

**Callouts must earn their box, and adding one is often a swap.** Before adding a callout, count the components already in the enclosing step, tab, or section. If yours would make three or more, rank them by how *surprising* each is to a reader who has read the surrounding prose — keep the least predictable one boxed and write the rest as prose. Baseline prerequisites lose that ranking every time. The test for whether something belongs in a box at all: could a reader delete it and still follow the section? If not, it is the argument — write it as prose with bold for weight.

**A fact that costs the reader a broken run goes in a box, and that half is easier to miss.** A value that fails the run, a permission the product cannot grant itself, a prerequisite whose absence breaks something later — each earns a `<Warning>` even where the prose around it reads fine. Weighing only whether a callout is *unnecessary* leaves the default on prose, which is the more common mistake.

Keep the box to the hazard. Once a callout runs longer than the prose it annotates, the section has moved inside it — leave the blocking part boxed and put the rest back out.

**`<ParamField>` defaults belong in the `default` prop**, not in body prose. Always quoted (`default="90"`), since the value may be conditional (`default="3 (Anthropic), 2 (Bedrock)"`), which `{}` cannot express.

### Cross-links and anchors

Verify the destination anchor exists before linking. Slug rules:

- `##`/`###`/`####` headings, `<Accordion title="…">`, `<Update label="…">` → `#<slugified-text>`
- `<ParamField body="X">` → `#param-<slugified-x>` (note the `#param-` prefix); the slug splits camelCase and turns underscores into hyphens, so `autoMerge` is `#param-auto-merge`, never `#param-automerge`, and `NOTIFY_WEBHOOK_URL` is `#param-notify-webhook-url`
- `<Step title="…">` and `<Tab title="…">` generate **no anchor** — link to the page instead
- A `/` inside a heading stays in the slug and must be URL-encoded as `%2F` in the link — a "Projects (team/repo mappings)" heading becomes `#projects-team%2Frepo-mappings`

On a `latest/…` page, every cross-link to another docs page must carry the `/latest/` prefix — in **both** `](/path)` Markdown links **and** `href="/path"` component props such as `<Card>`. An unprefixed link on a `latest/` page silently sends the reader to the stable copy. A prior sweep that handled only Markdown links left twelve `href=` props unprefixed across five pages.

### What never becomes a page edit

**Internal-only changes.** A change with no operator-visible behavior — an internal refactor, a database column, an auth-plumbing shift, a telemetry foundation — is not a per-page edit on either version. Release notes are written when a version ships, from the whole set of changes in it.

**Claims you haven't verified.** Every claim must be true of the product. Confirm the behavior at its source before writing it down, as *Verifying against source* describes — don't infer a feature from a configuration key, a name, or what an adjacent page implies.

This bites hardest on numbers. When a correction appears to contradict a figure already on the page, the usual cause is two mechanisms rather than one error — a retention ceiling and a page size, a summary window and a lookup guard. Establish what each figure measures before replacing either.

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
