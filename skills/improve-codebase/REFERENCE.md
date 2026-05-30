# Reference

The heuristics, categories, constraints, and templates below are grouped by
mode. `code`-mode sections apply when the target is a source repository;
`docs`-mode sections apply when the target is a documentation folder; `ui`-mode
sections apply when the target is a running web frontend driven through
Playwright. A run uses one mode only.

## Mode Matrix

Step 0 resolves the run to one mode. Every later step reads the column for that
mode from this table.

| Aspect | `code` mode | `docs` mode | `ui` mode |
|---|---|---|---|
| Target | source repository | documentation folder | running web frontend (Playwright) |
| Lenses (step 1) | architecture, testability, performance, security; quality: readability, duplication, dead-code, maintainability | accuracy, structure, clarity, consistency | responsive & layout, usability heuristics, visual consistency |
| Cluster (step 3) | files or modules | docs or sections | screen / route + component(s) behind it |
| Category (step 3) | dependency category (architecture / testability), risk severity (performance / security), or quality severity (readability / duplication / dead-code / maintainability) | fix-dependency category | UX severity (blocker / friction / polish) |
| Replaced (step 3) | tests or checks the change makes redundant; for quality lenses, the duplicated or dead code the change removes | docs to delete or merge | one-off style or duplicated component to consolidate, or n/a |
| Problem sketch (step 5) | code sketch | doc outline / IA sketch | annotated screenshot or wireframe |
| Design constraints (step 6) | § Design Constraints — `code` | § Design Constraints — `docs` | § Design Constraints — `ui` |
| Diagram (step 6) | flowchart or class diagram, before / after | doc-map / IA-tree flowchart, before / after | before / after user-flow flowchart or layout map |
| RFC template (step 7) | Templates A–C, selected by output channel | Template D | Template E |

## Per-Lens Exploration Heuristics — `code` mode

### Architecture (Ousterhout, *A Philosophy of Software Design*)

A deep module has a small interface hiding a large implementation. Look for shallow modules with interfaces nearly as complex as their implementations. Note where understanding one concept requires bouncing between many small files. Spot pure-function extractions that hide real bugs in how they're called. Identify integration risk in seams between tightly-coupled modules.

For cross-process dependencies, frame using Cockburn's Ports & Adapters.

### Testability (Khorikov, *Unit Testing*)

Tests should describe behaviour at the observable boundary, not implementation. Look for code only reachable through heavy mocking, tests asserting on internal state, missing tests at module boundaries, code extracted "for testability" with no real boundary, integration risk uncovered by the current suite. Prefer boundary tests. Replace, don't layer.

### Performance (Gregg USE method; Wilkie RED method)

Read for structural hot paths without profiling first: loops over network or DB calls, N+1 query patterns, sync I/O on request paths, redundant allocations, missing memoisation at obvious points, unnecessary serialisation. Frame resources as Utilization / Saturation / Errors (USE). Frame request paths as Rate / Errors / Duration (RED). Tie any recommendation to a believed hot path, not a cold one.

### Security (OWASP Top 10; OWASP ASVS; STRIDE)

Enumerate threats at trust boundaries using STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege). Cross-check against the current OWASP Top 10. Calibrate severity against OWASP ASVS. Look for input crossing trust boundaries without validation, authn or authz scattered or absent at a boundary, secrets in code or logs, unsafe deserialisation, SSRF, SQLi, command injection surfaces, overly-broad IAM or RBAC, missing rate-limiting on sensitive endpoints. Severity drives whether to file an RFC or recommend an immediate fix.

### Quality lenses — boundary vs architecture

The four quality lenses judge *within-module hygiene*: how the code reads, repeats itself, accumulates dross, and resists change at the function and file level. The `architecture` lens judges *between-module design*: interface depth, coupling, the seams. If a finding is really about module boundaries or interface shape, it belongs to `architecture`, not here. Quality findings are graded by quality severity (see § Quality Severity Categories), not the dependency or risk categories.

### Readability (Fowler, *Refactoring* 2nd ed. smell catalog; Ousterhout on naming and comments)

Code should reveal intent on first read. The friction signal is anything a reader must decode. Look for cryptic or misleading names, names that lie about what the code does, deep nesting (arrow code), long functions doing several things at mixed levels of abstraction, magic numbers and unexplained literals, boolean flag parameters that pack two behaviours into one function, and comments that restate the code or have drifted from it. A comment compensating for a bad name is itself the signal: prefer renaming, extracting, and flattening over explaining.

### Duplication & complexity (Fowler smells: Duplicated Code, Long Function, Large Class, Feature Envy; McCabe cyclomatic complexity)

Look for copy-pasted blocks that will drift apart, the same logic expressed in several places, god functions or classes accreting unrelated responsibilities, high branching (deeply nested conditionals, long if-else or switch chains), and feature envy (a method reaching repeatedly into another object's data). Tie complexity to change-risk, not raw line count: duplication that *will* drift and branching that *hides paths from the tests* are the real signal, not length alone.

### Dead code & consistency (dead-code elimination; Fowler "Speculative Generality"; idiom and error-handling adherence)

Look for unused exports, functions, variables, and imports; commented-out code; unreachable branches; speculative abstraction kept "just in case" with a single caller; flags or config that nothing reads; idiom drift (the same task done two different ways across the repo); and inconsistent error handling (some paths swallow, some throw, some log-and-continue). Replace, don't annotate: delete dead code, unify divergent idioms on the dominant existing one. Before flagging a symbol as unused, grep its call sites and check for dynamic or reflective use — a false "dead code" claim is expensive.

### Maintainability debt (Fowler tech-debt quadrant; Shotgun Surgery; Primitive Obsession)

Look for clusters of TODO / FIXME / HACK markers, shotgun-surgery coupling (one logical change forces edits across many files), missing abstraction where churn is known (cross-reference the recent-context bundle and git history), configuration sprawl, primitive obsession that pushes the same validation into every caller, and leaky error handling that makes every caller re-handle. Distinguish deliberate debt (tracked, with a paydown trigger) from accidental debt; for any candidate, the RFC names the paydown trigger or the churn evidence that justifies acting now.

## Per-Lens Exploration Heuristics — `docs` mode

### Accuracy

Documentation must match current reality. The friction signal is any claim a
reader would act on that is no longer true. Look for stale setup or run steps,
outdated versions, commands, flags, or env vars, broken internal and external
links, screenshots or output samples that no longer match, instructions that
contradict the code they describe, references to removed or renamed files.

### Structure (Diátaxis, Procida)

Every doc serves one of four needs: tutorial (learning), how-to (a task),
reference (lookup), explanation (understanding). A doc that mixes types is the
friction signal: a tutorial padded with reference tables, a how-to that detours
into rationale. Look also for missing index or table of contents, deep nesting
that buries content, pages a reader cannot find from where they start,
duplicated coverage of the same topic across pages.

### Clarity (progressive disclosure; plain-language principles)

A reader should reach what they need without decoding. Look for ambiguity,
undefined jargon and acronyms, missing prerequisites or context a first-time
reader needs, gaps where a step is assumed but not stated, walls of unstructured
text, the most important information buried below the fold. Prefer the simplest
wording that is still precise.

### Consistency (single-source-of-truth)

Each fact should live in exactly one place. Look for contradictory statements
across pages, terminology drift (the same concept named differently), the same
fact maintained in multiple places so copies fall out of sync, inconsistent
formatting, heading style, or voice across the doc set.

## Per-Lens Exploration Heuristics — `ui` mode

These lenses judge the **rendered** app, not the source. Each explorer discovers the
Playwright browser tools via `ToolSearch` (likely `browser_navigate`, `browser_resize`,
`browser_take_screenshot`, `browser_snapshot`, `browser_evaluate`; namespace varies by
environment) and drives the live app. `browser_take_screenshot` / `browser_resize` may
prompt for permission on first use. Cite screenshot path + route + viewport per finding.

### Responsive & layout (WCAG 1.4.10 reflow; 2.5.5 / 2.5.8 target size)

Resize across the breakpoint set (360 / 768 / 1024 / 1440 px) and screenshot each. The
friction signal is any viewport where the layout breaks. Look for horizontal overflow,
clipped or overlapping content, broken reflow (content lost or requiring two-axis
scroll), fixed pixel widths that force scrolling on narrow screens, touch targets under
~44px, content that does not adapt between mobile and desktop.

### Usability heuristics (Nielsen's 10)

Drive the rendered flows, not the markup. Check visibility of system status (loading,
progress, saved-state feedback), match between system and the real world, user control
and freedom (cancel / undo / back), consistency and standards, error prevention,
recognition over recall, flexibility, aesthetic and minimalist design, and clear error
recovery. Explicitly navigate the app into loading, empty, and error states and judge
what the user sees in each.

### Visual consistency (design-system / token adherence; Gestalt)

Read computed styles via `browser_evaluate` (colors, spacing, font sizes, radii) and
flag values that fall off the design scale. Look for one-off components where a shared
one exists, spacing and typographic scale drift across screens, inconsistent button /
control styling, and weak Gestalt grouping or alignment. One off-scale value on one
screen is the signal that the design language is not enforced.

## Dependency Categories (Architecture / Testability) — `code` mode

### 1. In-process

Pure computation, in-memory state, no I/O. Always deepenable. Merge modules and test directly.

### 2. Local-substitutable

Has a local test stand-in (PGLite for Postgres, in-memory filesystem, testcontainers). Deepenable if the substitute exists. Tested with the stand-in in the suite.

### 3. Remote but owned (Ports & Adapters)

Your own services across a network boundary. Define a port at the module boundary. The deep module owns the logic; the transport is injected. Tests use an in-memory adapter. Production uses HTTP, gRPC, or queue.

### 4. True external (Mock)

Third-party services you don't control (Stripe, Twilio). Mock at the boundary. The deep module takes the dependency as an injected port; tests provide a mock.

## Risk Categories (Performance / Security) — `code` mode

### Performance severity

- **Hot-path confirmed**: code on a known critical path
- **Hot-path suspected**: structurally hot but unmeasured
- **Cold-path cleanup**: tidy only; defer or skip

### Security severity (calibrated to OWASP ASVS)

- **Exploitable now**: file an issue AND recommend an out-of-band hotfix
- **Latent risk**: file an issue at normal priority
- **Defence-in-depth**: file only if the change is cheap relative to value

## Quality Severity Categories — `code` mode

Applies to the four quality lenses (readability / duplication / dead-code /
maintainability), parallel to performance and security severity. Drives whether a
candidate is filed and at what priority.

- **bug-risk**: the smell can hide or cause a defect — duplicated logic that will drift out of sync, a dead branch masking a live path, error handling that silently swallows. File and prioritise.
- **change-amplifier**: makes future change slower or riskier without being a defect today — high complexity, shotgun-surgery coupling, missing abstraction over known churn. File at normal priority.
- **cosmetic**: pure readability or consistency polish, no functional or change-cost risk. Batch; file only if cheap relative to value.

## UX Severity Categories — `ui` mode

Drives whether a candidate is filed and at what priority, parallel to performance /
security severity.

- **blocker**: a core task cannot be completed, or content is unusable, at a supported viewport. File and recommend an out-of-band fix.
- **friction**: the task is completable but harder than necessary; noticeable inconsistency or layout strain. File at normal priority.
- **polish**: cosmetic refinement only. File only if cheap relative to value; batch.

## Fix-Dependency Categories — `docs` mode

Answers "what does fixing this candidate depend on", parallel to the code
dependency categories. Drives whether the change can ship as a copy-edit or
needs verification or a decision first.

### 1. `copy-edit-only`

A prose fix with no external verification needed. Wording, structure of a single
page, formatting. Ship directly.

### 2. `restructure`

Moving, merging, or splitting content; no facts change. Reorganising an
information architecture, consolidating duplicated pages, retyping a doc to a
Diátaxis type.

### 3. `needs-SME-or-code-verification`

A claim must be checked against the code or with a domain expert before the
rewrite is correct. The RFC names exactly what to verify.

### 4. `needs-owner-decision`

Requires an owner call before proceeding, e.g. deprecate a page, change which
doc is the canonical source. The RFC surfaces the decision; it does not make it.

## Testing Strategy

Replace, don't layer.

- Old unit tests on shallow modules are waste once boundary tests exist. Delete them.
- Write new tests at the deepened module's interface boundary.
- Tests assert on observable outcomes through the public interface, not internal state.
- Tests should survive internal refactors.

In `docs` mode the same principle holds: redundant or stale pages get deleted or
merged, not annotated with a correction. One fact, one place.

## Design Constraints

In step 6 each design sub-agent gets ONE differentiating constraint for the
candidate's lens, so the alternatives genuinely diverge.

### `code` mode

- **Architecture / testability**: minimal interface | maximum flexibility | common-case-trivial | ports & adapters
- **Performance**: algorithmic rewrite | cache layer | batching or pipelining | lazy evaluation
- **Security**: validate-at-boundary | defence-in-depth | eliminate-the-surface | framework-native primitives
- **Readability**: rename-in-place | extract-function | flatten-control-flow | introduce-explaining-variable
- **Duplication**: extract-shared-helper | parameterize-the-variant | polymorphism-or-strategy | inline-then-simplify
- **Dead-code**: delete-outright | unify-on-existing-idiom | lint-rule-enforced | centralize-error-handling
- **Maintainability**: pay-down-now | isolate-behind-a-seam | document-the-decision | defer-with-tracked-ticket

### `docs` mode

- **Structure**: flat | hierarchical | Diátaxis-typed | single-page consolidation
- **Clarity**: rewrite-in-place | layered progressive-disclosure | split-by-audience
- **Accuracy / consistency**: single-source rewrite | cross-reference | deprecate-and-redirect

### `ui` mode

- **Responsive & layout**: mobile-first | fluid / clamp scaling | explicit breakpoint set | container-query
- **Usability heuristics**: reduce-steps | inline-feedback | progressive-disclosure | forgiving-defaults
- **Visual consistency**: design-token-first | component-extraction | systematic scale | match-existing-pattern

## Solution Presentation

Every alternative produced in step 6 must include:

1. **Interface / approach**: types, methods, params, or algorithmic shape
2. **Usage example**: how callers integrate
3. **What it hides**: complexity or risk now encapsulated
4. **Dependency or risk strategy**: which category applies
5. **Trade-offs**: what gets worse, not only better
6. **Mermaid before/after diagram**: two diagrams, type per § Mode Matrix. Follow `~/.claude/rules/mermaid.md`: classDef hex pairs meeting WCAG AA (e.g. `fill:#2c3e50,color:#ffffff`), `<br/>` not `\n`, no parentheses in labels, `:::className` inline, single subgraph, 3+ character node IDs.

In `ui` mode the required mermaid before/after is a user-flow flowchart or a layout
map, and each alternative references its before screenshots by saved path.

After all alternatives are presented, write a comparison table:

| Alternative | Hides complexity | Test impact | Integration cost | Risk |
|---|---|---|---|---|

Then give an opinionated recommendation (possibly a hybrid).

## RFC Templates

### Template A: GitHub / GitLab issue

    ## Lens
    <architecture | testability | performance | security | readability | duplication | dead-code | maintainability>

    ## Problem
    <the friction observed; affected files or modules; severity or dependency category>

    ## Proposed Solution
    <chosen alternative's interface or approach + usage example>

    ## Diagram
    <mermaid before/after, required>

    ## Dependency, Risk, or Quality Severity
    <which dependency category, risk severity, or quality severity applies; how it is handled>

    ## Testing / Verification Strategy
    - New boundary tests to write: <list>
    - Old tests to delete: <list>
    - Verification steps: <how to confirm the change holds>

    ## Implementation Recommendations
    <durable architectural guidance NOT coupled to current file paths: what the module should own, hide, expose; how callers should migrate>

### Template B: Markdown file

Same body as Template A, with YAML frontmatter:

    ---
    lens: <lens>
    severity_or_category: <value>
    created: <YYYY-MM-DD>
    ---

### Template C: Inline

Same body as Template A, printed to chat. No file is created.

### Template D: `docs` mode RFC

Used in `docs` mode for every output channel. Written to a GitHub issue, GitLab
issue, Markdown file, or inline exactly as Templates A–C are; only the body
differs. For a Markdown file, prepend the same YAML frontmatter as Template B,
with `lens` drawn from the docs lenses.

    ## Lens
    <accuracy | structure | clarity | consistency>

    ## Problem
    <the friction observed; docs or sections affected; fix-dependency category>

    ## Proposed Rewrite or Restructure
    <chosen alternative's outline or approach; the new shape of the docs>

    ## Diagram
    <mermaid doc-map / IA-tree before/after, required>

    ## Fix-Dependency Strategy
    <which fix-dependency category applies; what verification or decision is needed before the rewrite is correct>

    ## Verification Strategy
    - How to confirm docs match reality: <follow the steps, check links, diff against code>
    - Docs to delete or merge: <list>

    ## Authoring Recommendations
    <durable guidance NOT coupled to current file paths: what each doc should own, which Diátaxis type it should be, where the canonical source of each fact lives, how readers should navigate>

### Template E: `ui` mode RFC

Used in `ui` mode for every output channel (GitHub issue, GitLab issue, Markdown file,
inline), exactly as Templates A–C choose a channel. For a Markdown file, prepend the
same YAML frontmatter as Template B, with `lens` drawn from the UI/UX lenses.

    ## Lens
    <responsive & layout | usability heuristics | visual consistency>

    ## Problem
    <friction observed; screen / route + component; UX severity; screenshot evidence paths>

    ## Evidence
    <before screenshots, linked or embedded by saved path; route + viewport for each>

    ## Proposed Solution
    <chosen alternative's approach; component / style change; usage where relevant>

    ## Diagram
    <mermaid before/after user-flow flowchart or layout map, required>

    ## UX Severity & Scope
    <blocker | friction | polish; which screens / breakpoints are affected>

    ## Verification Strategy
    - Re-drive the route at each breakpoint via Playwright; compare before/after screenshots
    - Confirm the friction is gone in the rendered app
    - Visual-regression snapshot to add or update: <list>

    ## Implementation Recommendations
    <durable guidance NOT coupled to current file paths: design token to introduce, component to extract, breakpoint policy, where the canonical style lives>
