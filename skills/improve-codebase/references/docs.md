# Docs Mode Reference

## Contents

- Per-Lens Exploration Heuristics — `docs` mode
- Fix-Dependency Categories — `docs` mode
- Testing Strategy
- Design Constraints
- RFC Templates

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

In `docs` mode the same principle holds: redundant or stale pages get deleted or
merged, not annotated with a correction. One fact, one place.

## Design Constraints

In step 6 each design subagent gets ONE differentiating constraint for the
candidate's lens, so the alternatives genuinely diverge.

### `docs` mode

- **Structure**: flat | hierarchical | Diátaxis-typed | single-page consolidation
- **Clarity**: rewrite-in-place | layered progressive-disclosure | split-by-audience
- **Accuracy / consistency**: single-source rewrite | cross-reference | deprecate-and-redirect

## RFC Templates

### Template D: `docs` mode RFC

Used in `docs` mode for every output channel. Written to a GitHub issue, GitLab
issue, Markdown file, or inline; only the body differs. For a Markdown file,
prepend this YAML frontmatter, with `lens` drawn from the docs lenses:

    ---
    lens: <lens>
    severity_or_category: <value>
    created: <YYYY-MM-DD>
    ---

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
