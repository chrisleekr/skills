# UI Mode Reference

## Per-Lens Exploration Heuristics — `ui` mode

These lenses judge the **rendered** app, not the source. Each explorer discovers the
Playwright browser tools (in Claude Code, via `ToolSearch`; likely `browser_navigate`, `browser_resize`,
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

## UX Severity Categories — `ui` mode

Drives whether a candidate is filed and at what priority, parallel to performance /
security severity.

- **blocker**: a core task cannot be completed, or content is unusable, at a supported viewport. File and recommend an out-of-band fix.
- **friction**: the task is completable but harder than necessary; noticeable inconsistency or layout strain. File at normal priority.
- **polish**: cosmetic refinement only. File only if cheap relative to value; batch.

## Design Constraints

In step 6 each design subagent gets ONE differentiating constraint for the
candidate's lens, so the alternatives genuinely diverge.

### `ui` mode

- **Responsive & layout**: mobile-first | fluid / clamp scaling | explicit breakpoint set | container-query
- **Usability heuristics**: reduce-steps | inline-feedback | progressive-disclosure | forgiving-defaults
- **Visual consistency**: design-token-first | component-extraction | systematic scale | match-existing-pattern

## RFC Templates

### Template E: `ui` mode RFC

Used in `ui` mode for every output channel (GitHub issue, GitLab issue, Markdown file,
inline). For a Markdown file, prepend this YAML frontmatter, with `lens` drawn from
the UI/UX lenses:

    ---
    lens: <lens>
    severity_or_category: <value>
    created: <YYYY-MM-DD>
    ---

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
