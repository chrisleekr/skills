# Code Mode Reference

## Contents

- Per-Lens Exploration Heuristics — `code` mode
- Dependency Categories (Architecture / Testability) — `code` mode
- Risk Categories (Performance / Security) — `code` mode
- Quality Severity Categories — `code` mode
- Testing Strategy
- Design Constraints
- RFC Templates

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

## Testing Strategy

Replace, don't layer.

- Old unit tests on shallow modules are waste once boundary tests exist. Delete them.
- Write new tests at the deepened module's interface boundary.
- Tests assert on observable outcomes through the public interface, not internal state.
- Tests should survive internal refactors.

## Design Constraints

In step 6 each design subagent gets ONE differentiating constraint for the
candidate's lens, so the alternatives genuinely diverge.

### `code` mode

- **Architecture / testability**: minimal interface | maximum flexibility | common-case-trivial | ports & adapters
- **Performance**: algorithmic rewrite | cache layer | batching or pipelining | lazy evaluation
- **Security**: validate-at-boundary | defence-in-depth | eliminate-the-surface | framework-native primitives
- **Readability**: rename-in-place | extract-function | flatten-control-flow | introduce-explaining-variable
- **Duplication**: extract-shared-helper | parameterize-the-variant | polymorphism-or-strategy | inline-then-simplify
- **Dead-code**: delete-outright | unify-on-existing-idiom | lint-rule-enforced | centralize-error-handling
- **Maintainability**: pay-down-now | isolate-behind-a-seam | document-the-decision | defer-with-tracked-ticket

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
