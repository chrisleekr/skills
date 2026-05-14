---
name: improve-codebase
description: Explore a codebase through one or more lenses (architecture, testability, performance, security) to surface improvement opportunities, design alternatives in parallel, and emit a refactor RFC. Use when the user wants to find refactoring opportunities, improve testability, fix performance hotspots, harden security surfaces, or make a codebase more maintainable and AI-navigable.
---

# Improve Codebase

Explore a codebase like an AI would, surface friction across selected lenses, design alternatives in parallel, and produce a refactor RFC.

Per-lens heuristics, dependency and risk categories, solution-presentation rules, and RFC templates: see [REFERENCE.md](REFERENCE.md).

## When to use

- "Find refactoring opportunities", "improve testability", "find performance hotspots", "audit security surfaces"
- The user wants discovery, not a fix they've already scoped
- The repo is large enough that organic exploration adds value

## When NOT to use

- The user has already named the file or function to change (use direct edit)
- The user wants a code review on a diff (use the review skill)
- The user wants a single fix shipped (use `pr-auto-github` or `pr-auto-gitlab`)

## Process

### 0. Pre-flight

- Confirm working directory is a git repo. If not, ask the user where to point the skill.
- Detect remote: parse `git remote -v` to decide whether `gh` (GitHub) and / or `glab` (GitLab) are viable output targets in step 7. Cache the result.
- Confirm scope. By default explore the whole repo. If the user invoked the skill inside a subdirectory, ask: "Scope to `<subdir>` or whole repo?"
- This skill is read-only until step 7. No code is modified.

### 1. Pick lenses

Ask via `AskUserQuestion` with `multiSelect: true`. Options: `architecture`, `testability`, `performance`, `security`.

### 2. Explore in parallel

Spawn one `Agent` with `subagent_type=Explore` PER selected lens, in a single message. Each agent's brief includes:

- Scope (paths from step 0)
- The lens's heuristics block from REFERENCE.md, verbatim
- A "friction IS the signal" instruction. Explore organically, do NOT follow rigid heuristics mechanically.
- Output shape: a numbered list of 3 to 7 candidates, each with cluster, why-it-surfaced, category, and tests-or-checks-replaced
- An explicit instruction: "If nothing meaningful surfaces for this lens, return an empty list and say so. Do not invent weak candidates."

### 3. Present candidates

Merge the per-lens lists into one numbered list. For each candidate show:

- **Lens** tag
- **Cluster**: files or modules involved
- **Why it surfaced**: the friction observed
- **Category**: dependency category (architecture / testability) or severity (performance / security)
- **Tests or checks replaced**: what existing coverage becomes redundant

If every lens returned empty, tell the user honestly and stop. Do NOT propose solutions yet. Ask: "Which would you like to explore?"

### 4. User picks one

### 5. Frame the problem space

User-facing explanation:

- Constraints any solution must satisfy
- Dependencies it would rely on
- A rough illustrative code sketch (NOT a proposal, only grounding)

Show this, then immediately proceed to step 6 so the user reads while sub-agents work.

### 6. Design 3+ alternatives in parallel

Spawn 3+ `Agent` sub-agents in a single message. Each brief includes file paths, coupling or risk detail, lens, category, and ONE differentiating constraint:

- **Architecture / testability**: minimal interface | maximum flexibility | common-case-trivial | ports & adapters
- **Performance**: algorithmic rewrite | cache layer | batching or pipelining | lazy evaluation
- **Security**: validate-at-boundary | defence-in-depth | eliminate-the-surface | framework-native primitives

Each sub-agent output: see "Solution Presentation" in REFERENCE.md. A mermaid before/after diagram is required.

Present designs sequentially with their diagrams. Compare in a table (see REFERENCE.md). Give an **opinionated** recommendation. The user wants a strong read, not a menu. If elements from different designs combine well, propose a hybrid. User accepts or picks an alternative.

### 7. Emit the RFC

Ask via `AskUserQuestion` where to write the RFC. Show only the options that pre-flight (step 0) deemed viable:

- **GitHub issue** → `gh issue create` (only if GitHub remote present and `gh` authenticated)
- **GitLab issue** → `glab issue create` (only if GitLab remote present and `glab` authenticated)
- **Markdown file** → ask the user for the target directory, then write `<directory>/<YYYYMMDD>-<kebab-slug>.md` (date is today)
- **Inline only** → print to chat, no file

Use the templates in REFERENCE.md. Do NOT ask the user to review the RFC before creating. Share the URL or path.

### 8. Offer to loop

Ask: "Want to tackle another candidate?" If yes, return to step 3 with the same candidate list (minus the one just shipped). If no, end.
