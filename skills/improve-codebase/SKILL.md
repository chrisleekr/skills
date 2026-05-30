---
name: improve-codebase
description: Explore a codebase, a documentation folder, or a running web frontend through one or more lenses to surface improvement opportunities, design alternatives in parallel, and emit an RFC. Code lenses architecture, testability, performance, security, and quality (readability, duplication, dead-code, maintainability). Docs lenses accuracy, structure, clarity, consistency. UI/UX lenses responsive layout, usability heuristics, visual consistency. Use when the user wants to find refactoring opportunities, improve testability, fix performance hotspots, harden security surfaces, improve readability, remove duplication or dead code, pay down maintainability debt, improve documentation, or improve the rendered UI/UX of a running web app.
---

# Improve Codebase

Explore a codebase, a documentation folder, or a running web frontend like an AI would, surface friction across selected lenses, design alternatives in parallel, and produce an RFC.

The skill runs in one of three **modes**, picked per run and mutually exclusive:

- **`code`** — target is a source repository. Lenses: architecture, testability, performance, security, plus the quality lenses readability, duplication, dead-code, maintainability.
- **`docs`** — target is a documentation folder (specs, ADRs, runbooks, READMEs, wikis). Lenses: accuracy, structure, clarity, consistency.
- **`ui`** — target is a running web frontend, driven through the Playwright MCP. Lenses: responsive & layout, usability heuristics, visual consistency.

The explore → present → design → RFC → loop spine is identical across modes; only the lenses, heuristics, categories, design constraints, and RFC template differ. `ui` mode additionally drives a live browser session during exploration and verification (it still modifies no source files until step 7).

Per-lens heuristics, dependency / risk / fix-dependency categories, solution-presentation rules, and RFC templates: see [REFERENCE.md](REFERENCE.md).

## When to use

- `code` mode: "find refactoring opportunities", "improve testability", "find performance hotspots", "audit security surfaces", "improve readability", "find duplication / code smells", "remove dead code", "pay down maintainability debt"
- `docs` mode: "improve the docs", "find documentation gaps", "the docs are stale / hard to navigate"
- `ui` mode: "review the UI/UX of http://localhost:3000", "the app feels clunky or inconsistent", "find layout / responsive problems", "the mobile view is broken"
- The user wants discovery, not a fix they've already scoped
- The target is large enough that organic exploration adds value

## When NOT to use

- The user has already named the file, function, or doc to change (use direct edit)
- The user wants a code review on a diff (use the review skill)
- The user wants a single fix shipped (use `pr-auto-github` or `pr-auto-gitlab`)

## Process

### 0. Pre-flight

- Confirm working directory is a git repo. If not, ask the user where to point the skill.
- Detect remote: parse `git remote -v` to decide whether `gh` (GitHub) and / or `glab` (GitLab) are viable output targets in step 7. Cache the result.
- Confirm scope. By default explore the whole repo. If the user invoked the skill inside a subdirectory, ask: "Scope to `<subdir>` or whole repo?"
- **Detect mode.** `ui` mode is **intent-selected**, not composition-detected: a web repo is also valid `code`, so file composition cannot disambiguate it. Otherwise classify by file composition:
  - User explicitly asks to review the rendered UI/UX of a web app → `ui` mode
  - Predominantly prose (`.md`, `.mdx`, `.rst`, `.adoc`, `.txt`) → `docs` mode
  - Predominantly source → `code` mode
  - Mixed or ambiguous → ask the user which the run targets; a run is one mode, never both
  State the detected mode back to the user and confirm before proceeding.
- **Gather recent-context bundle** to feed the explorers in step 2. Mature repos churn fast and cold-context lenses repeatedly re-surface yesterday's fixes. Collect, but do NOT show the user:
  - Last 50 commit subjects: `git log --oneline -50`
  - If the project has auto-memory at `.claude/projects/.../memory/MEMORY.md` or `~/.claude/projects/.../memory/MEMORY.md` keyed by this working directory, read it (it's an index of one-liners).
  - If `CHANGELOG.md` or `docs/decisions/` exists at the repo root, grab the most recent ~20 entries.
  Cache the bundle as a single string under a few hundred lines. Step 2 will paste it into each explorer brief verbatim.
- **`ui` mode pre-flight** (skip in `code` / `docs` modes):
  - **Non-web bail.** If the target is mobile / native / a Figma file or has no browser entry point, explain that Playwright drives only web browsers and stop. Offer `code` mode on the same repo instead.
  - **Verify Playwright is reachable.** Discover the browser tools with `ToolSearch` (e.g. query `playwright browser navigate snapshot`, or `select:mcp__playwright__browser_navigate`). The exact namespace varies by environment (`mcp__playwright__browser_*` or `mcp__plugin_playwright_playwright__browser_*`); use whatever resolves. If nothing resolves, tell the user the Playwright MCP is not loaded this session and stop.
  - **Get the app under test.** Ask the user for the base URL (e.g. `http://localhost:3000`) and the routes / screens to review. Assume the app is already running; do NOT launch a dev server. If no URL is given, ask for one.
  - **Screenshot run dir.** Create a run-scoped temp dir `/tmp/improve-ui-<YYYYMMDD-HHMM>/`; explorers and verification save evidence there.
  - **Breakpoint set.** The responsive lens uses viewports 360, 768, 1024, 1440 px unless the user names others.
- This skill is read-only on the codebase until step 7. In `ui` mode it drives a live browser (navigate, resize, screenshot, read computed styles) and writes screenshots to the run temp dir, but modifies no source files and launches no servers.

### 1. Pick lenses

Ask via `AskUserQuestion` with `multiSelect: true`. The lens set for the current mode is in REFERENCE.md § Mode Matrix.

### 2. Explore in parallel

Spawn one `Agent` PER selected lens, in a single message. **Pick the subagent_type by what the lens needs:**

- `general-purpose` — default for every lens in this skill. Lenses here audit *whether existing code/docs are sound*, which requires reading whole files (mitigations often live below the read window of an excerpt-based search). The `Explore` agent's own description warns against using it for "code review, design-doc auditing, cross-file consistency checks, or open-ended analysis" — those describe every lens in this skill.
- `Explore` — only if a lens degenerates into pure lookup ("where is X defined", "which files import Y"). Otherwise prefer `general-purpose`.
- **`ui` mode**: always `general-purpose` — the explorer must both read component source and drive the browser. `Explore` cannot drive Playwright.

Each agent's brief includes:

- Scope (paths from step 0) and the mode from step 0
- **The recent-context bundle from step 0**, prefixed with: "These commits / memory entries describe work already shipped. Do NOT surface candidates that re-discover a fix that already landed. If a finding overlaps with a listed entry, either drop it or explain in one line why it's still incomplete."
- The lens's heuristics block from REFERENCE.md, verbatim — but framed as *signals to investigate*, not a checklist to grind through.
- A "friction IS the signal" instruction. Explore organically; do NOT pattern-match heuristics mechanically.
- **A read-whole-files instruction**: "Before flagging any candidate, read each cited file end-to-end. If the file already mitigates the concern (existing retry/backoff, existing batching, existing memoization, existing gate that prevents the hot path), drop the candidate. Cite the line range you read."
- Output shape: a numbered list of 3 to 7 candidates, each carrying the step 3 fields (see § Mode Matrix for what each resolves to in the current mode)
- An explicit instruction: "If nothing meaningful surfaces for this lens, return an empty list and say so. Do not invent weak candidates to fill quota — a 7/8 false-positive rate is worse than a 1/1 hit rate."

**In `ui` mode**, each explorer brief instead includes the base URL, the route list, the screenshot run dir, the breakpoint set, and its lens heuristics from REFERENCE.md. Instruct the agent to:

- Discover the Playwright browser tools via `ToolSearch`, then drive the app — never reason about the UI from source alone.
- Navigate each route. For the responsive lens, resize to each breakpoint (360 / 768 / 1024 / 1440) and screenshot. For the usability lens, drive the app into loading / empty / error states. For the visual lens, read computed styles via `browser_evaluate` (colors, spacing, font sizes) to detect off-scale values.
- Save every screenshot to the run dir and, for each candidate, cite **screenshot path + route + viewport** (the render is the evidence, not a file:line).
- "Render IS the signal" — judge the rendered page, not the code. Same empty-list-is-fine rule: do not invent weak candidates.
- Note: `browser_take_screenshot` and `browser_resize` may not be pre-allowlisted and can prompt for permission on first use.

### 3. Present candidates

Merge the per-lens lists into one numbered list. For each candidate show:

- **Lens** tag
- **Cluster**: the files, modules, docs, or sections involved
- **Why it surfaced**: the friction observed
- **Category**: the category for this lens — see § Mode Matrix
- **Replaced**: what existing coverage the change makes redundant — see § Mode Matrix

If every lens returned empty, tell the user honestly and stop. Do NOT propose solutions yet. Ask: "Which would you like to explore?"

### 4. User picks one

### 5. Frame the problem space

User-facing explanation:

- Constraints any solution must satisfy
- Dependencies it would rely on
- A rough illustrative sketch (NOT a proposal, only grounding); sketch type per § Mode Matrix

Show this, then immediately proceed to step 6 so the user reads while sub-agents work.

### 6. Design 3+ alternatives in parallel

Spawn 3+ `Agent` sub-agents in a single message. Each brief includes the cluster (file or doc paths), coupling or risk detail, lens, category, and ONE differentiating constraint drawn from REFERENCE.md § Design Constraints for the candidate's lens.

Each sub-agent output: see "Solution Presentation" in REFERENCE.md. A mermaid before/after diagram is required.

Present designs sequentially with their diagrams. Compare in a table (see REFERENCE.md). Give an **opinionated** recommendation. The user wants a strong read, not a menu. If elements from different designs combine well, propose a hybrid. User accepts or picks an alternative.

### 7. Emit the RFC

Ask via `AskUserQuestion` where to write the RFC. Show only the options that pre-flight (step 0) deemed viable:

- **GitHub issue** → `gh issue create` (only if GitHub remote present and `gh` authenticated)
- **GitLab issue** → `glab issue create` (only if GitLab remote present and `glab` authenticated)
- **Markdown file** → ask the user for the target directory, then write `<directory>/<YYYYMMDD>-<kebab-slug>.md` (date is today)
- **Inline only** → print to chat, no file

RFC body template: see § Mode Matrix. Do NOT ask the user to review the RFC before creating. Share the URL or path.

### 8. Offer to loop

Ask: "Want to tackle another candidate?" If yes, return to step 3 with the same candidate list (minus the one just shipped). If no, end.
