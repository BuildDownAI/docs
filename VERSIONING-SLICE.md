# AI-Implement docs — Mintlify versioning evaluation

## Context

AI-Implement's `main` branch is the released/stable code; `testing` is the in-development branch with significant divergence (102 files, +9,498 / −1,661 LOC vs main). This slice evaluates Mintlify-native approaches to maintaining both versions of the docs alongside each other.

Two short-lived branches off `june-doc-update`:
- `versioning-slice/pattern-a` — site-wide versioning
- `versioning-slice/pattern-b` — nested-in-tab versioning

Both implement the same v2 deltas on `reference/admin-ui.mdx`: per-project run caps (`maxTurns`, `maxIterations`, `maxJobMinutes`) and the required `defaultBranch` change. These are real testing-branch changes, not contrived.

## TL;DR

| Pattern | Verdict |
|---|---|
| **A — Site-wide versioning** | Works as documented. Heaviest maintenance: every page duplicated per version. Cleanest reader UX. |
| **B — Nested-in-tab versioning** | Works, but forces the versioned section into its own top-level tab. The "Reference" group is removed from the Docs sidebar. Lighter overall maintenance with a surprising IA cost. |
| **C — `<Tabs>` component (rejected)** | Looks like it solves the problem; doesn't. SEO, deep-linking, persistence all break. Tab-sync collisions with existing Linear/Jira tabs. Documented for completeness. |

**Recommendation:** Pattern A if versioning is required; consider a lighter alternative (per-page version badges + a changelog page) before committing to either A or B given that the actual divergence is ~12 HIGH-priority page-level gaps, which both patterns over-provision for. Detailed reasoning at the end.

## Pattern A — Site-wide versioning

### Structure
- Top-level `versions` array in `docs.json` with two entries: `stable` (default) and `latest`
- All 22 docs pages duplicated into `stable/` and `latest/` directories
- Snippet at `snippets/experimental-banner.mdx` imported into each `latest/*.mdx` page

### Pros
- Dropdown UI persists across all navigation; pick once, stays selected
- Documented Mintlify pattern; lowest risk
- No sidebar IA disruption

### Cons
- **22 pages × 2 versions = 44 files** to maintain
- Most pages start identical and drift only when content actually diverges
- Banner import line required on every `latest/*.mdx` individually (no per-version global banner in Mintlify)

### Maintenance shape
- Adding a new docs page: 2 file creates + 2 docs.json edits
- Adding a divergence to one page: 1 file edit (latest only) + ensure banner import is present
- Removing a docs page: 2 file deletes + 2 docs.json edits

### URL shape
All pages prefixed with version slug: `/stable/reference/admin-ui`, `/latest/reference/admin-ui`. No way to land the default version at root without untested config changes.

## Pattern B — Nested-in-tab versioning

### Structure
- `versions` array inside the `Reference` tab; other tabs version-free
- 5 Reference pages duplicated into `reference/latest/`; other 17 pages exist exactly once
- Same banner snippet, imported only on versioned pages

### Pros
- Fewer files duplicated (5 vs 22)
- Versioning scoped to where divergence actually lives
- URL design naturally lands at "default at root, others prefixed" — stable Reference pages live at `/reference/admin-ui`, latest at `/reference/latest/admin-ui`

### Cons
- **Forces structural promotion**: Reference must become its own top-level tab; the Reference group disappears from the Docs sidebar. Discoverability hit confirmed during testing — even knowing what was done, finding the relocated content took conscious effort.
- Dropdown only appears when reader is in the Reference tab. Easy to miss that versioning exists.
- Adding versioning to another area (e.g., Configuration) means promoting that group to a tab too. Versioning new areas has a real structural cost.

### Experimental finding
We tested whether `versions` could nest INSIDE a `group` (keeping Reference in the Docs sidebar instead of promoting to a tab). **Mintlify accepts the structure without erroring but silently drops the entire group from the sidebar and provides no version switcher.** Tab-promotion is the only working scope for nested-in-tab versioning. Matches the [open Mintlify issue #402](https://github.com/mintlify/docs/issues/402) about coarse versioning granularity.

### Maintenance shape
- Adding a new docs page in a non-versioned area: 1 file create + 1 docs.json edit
- Adding a new versioned Reference page: 2 file creates + 2 docs.json edits
- Adding versioning to a previously-unversioned area: requires tab restructuring (significant IA shift)

### URL shape
Asymmetric by default: `/reference/admin-ui` (stable, default), `/reference/latest/admin-ui` (latest). Closest to industry norm for asymmetric-traffic docs (React, Tailwind, Docusaurus).

## Pattern C — `<Tabs>` component (rejected alternative)

### Approach
Use Mintlify's `<Tabs>` component (not `versions`) to put "Stable" / "Latest" tabs inline on the page.

### Why rejected
| Concern | Why it breaks |
|---|---|
| Deep-linkable URLs | Tab state is client-side; both versions live at the same URL. Can't share "this is what's new in latest." |
| SEO | Google indexes one page with both versions' jumbled content |
| Persistence | Each page is its own tab choice — readers re-pick on every navigation |
| Tab-sync collisions | Mintlify synchronizes tabs with matching titles. Existing Linear/Jira tabs (Block 2 pattern) would interact unpredictably with version Tabs |
| Content duplication | Inline duplication per page replaces file duplication — no real maintenance savings |

Use case where C might fit: a single standalone page with no other Tabs and no need for shareable URLs (an internal runbook). Not AI-Implement's situation.

## Cross-cutting findings

### Banner scoping
Mintlify's site-level `banner` in `docs.json` is **site-wide only** — no per-version scoping documented. The reusable-snippet workaround used in both patterns (snippets imported per page) works but requires an import line on every `latest/*.mdx`.

Mintlify's component banner (`<Warning>`, `<Info>`, etc.) renders **inline with page content**, not as a sticky header. Supabase-style sticky version banners outside the page body would require custom CSS or a Mintlify feature request.

### URL design (deferred decision)
The choice between "default version at root" (e.g., `/intro`) and "all versions prefixed" (e.g., `/stable/intro`) is orthogonal to A vs B and worth a separate decision. Pattern B implicitly delivers default-at-root URLs; Pattern A always prefixes. If Pattern A is chosen and default-at-root URLs are desired, Mintlify support for asymmetric path prefixes within `versions` needs to be tested — not documented.

### Mintlify versioning granularity
The smallest unit you can practically version is a **tab**. Smaller scopes (a single page, a group inside a tab) are not supported by Mintlify's working schema. This is the root cause of Pattern B's structural cost.

## Recommendation

In AI-Implement's context:

1. **The divergence is real enough to need *some* approach.** The audit appendix identifies ~12 HIGH-priority page-level gaps between main and testing.
2. **The audience is asymmetric.** Stable-release operators will dominate; testing-branch early adopters are a smaller cohort. Asymmetric traffic disfavors heavy versioning machinery on both sides.
3. **Mintlify's coarse granularity makes either pattern feel heavy for what's actually 12 page-level divergences.** Both A and B are over-provisioned for the actual change set.

**A lighter alternative worth considering before committing to A or B:** keep one version of pages, add `<Info>` "Available in vX.x+" badges per page where features diverge, and surface a `/changelog` page with the testing-branch news. Lower maintenance, no IA disruption, no Mintlify granularity fight. Reader experience is "current docs with deprecation/preview notes" rather than two parallel sites.

If full versioning is preferred: **Pattern A is the safer pick** — documented happy path, no IA cost — accepting the higher maintenance burden as the price of cleaner navigation.

## Deferred questions for follow-up

1. **URL design**: default-at-root vs all-prefixed? Pattern B answers this implicitly; Pattern A would need separate work.
2. **Sticky banner**: do we accept inline banners, or write custom CSS / file a Mintlify feature request?
3. **Whether to version at all**: would per-page badges + a changelog page meet AI-Implement's needs more cheaply?
4. **Cadence**: if versioning, when do we cut a new "stable" version (release tag? feature freeze?) and who owns the backport policy?
5. **Documentation automation**: independent of versioning, the open scheduled-audit-via-GitHub-Actions proposal would apply equally to either pattern.
