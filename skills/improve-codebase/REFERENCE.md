# Reference

## Per-Lens Exploration Heuristics

### Architecture (Ousterhout, *A Philosophy of Software Design*)

A deep module has a small interface hiding a large implementation. Look for shallow modules with interfaces nearly as complex as their implementations. Note where understanding one concept requires bouncing between many small files. Spot pure-function extractions that hide real bugs in how they're called. Identify integration risk in seams between tightly-coupled modules.

For cross-process dependencies, frame using Cockburn's Ports & Adapters.

### Testability (Khorikov, *Unit Testing*)

Tests should describe behaviour at the observable boundary, not implementation. Look for code only reachable through heavy mocking, tests asserting on internal state, missing tests at module boundaries, code extracted "for testability" with no real boundary, integration risk uncovered by the current suite. Prefer boundary tests. Replace, don't layer.

### Performance (Gregg USE method; Wilkie RED method)

Read for structural hot paths without profiling first: loops over network or DB calls, N+1 query patterns, sync I/O on request paths, redundant allocations, missing memoisation at obvious points, unnecessary serialisation. Frame resources as Utilization / Saturation / Errors (USE). Frame request paths as Rate / Errors / Duration (RED). Tie any recommendation to a believed hot path, not a cold one.

### Security (OWASP Top 10; OWASP ASVS; STRIDE)

Enumerate threats at trust boundaries using STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege). Cross-check against the current OWASP Top 10. Calibrate severity against OWASP ASVS. Look for input crossing trust boundaries without validation, authn or authz scattered or absent at a boundary, secrets in code or logs, unsafe deserialisation, SSRF, SQLi, command injection surfaces, overly-broad IAM or RBAC, missing rate-limiting on sensitive endpoints. Severity drives whether to file an RFC or recommend an immediate fix.

## Dependency Categories (Architecture / Testability)

### 1. In-process

Pure computation, in-memory state, no I/O. Always deepenable. Merge modules and test directly.

### 2. Local-substitutable

Has a local test stand-in (PGLite for Postgres, in-memory filesystem, testcontainers). Deepenable if the substitute exists. Tested with the stand-in in the suite.

### 3. Remote but owned (Ports & Adapters)

Your own services across a network boundary. Define a port at the module boundary. The deep module owns the logic; the transport is injected. Tests use an in-memory adapter. Production uses HTTP, gRPC, or queue.

### 4. True external (Mock)

Third-party services you don't control (Stripe, Twilio). Mock at the boundary. The deep module takes the dependency as an injected port; tests provide a mock.

## Risk Categories (Performance / Security)

### Performance severity

- **Hot-path confirmed**: code on a known critical path
- **Hot-path suspected**: structurally hot but unmeasured
- **Cold-path cleanup**: tidy only; defer or skip

### Security severity (calibrated to OWASP ASVS)

- **Exploitable now**: file an issue AND recommend an out-of-band hotfix
- **Latent risk**: file an issue at normal priority
- **Defence-in-depth**: file only if the change is cheap relative to value

## Testing Strategy

Replace, don't layer.

- Old unit tests on shallow modules are waste once boundary tests exist. Delete them.
- Write new tests at the deepened module's interface boundary.
- Tests assert on observable outcomes through the public interface, not internal state.
- Tests should survive internal refactors.

## Solution Presentation

Every alternative produced in step 6 must include:

1. **Interface / approach**: types, methods, params, or algorithmic shape
2. **Usage example**: how callers integrate
3. **What it hides**: complexity or risk now encapsulated
4. **Dependency or risk strategy**: which category applies
5. **Trade-offs**: what gets worse, not only better
6. **Mermaid before/after diagram**: two flowcharts or two class diagrams, following `~/.claude/rules/mermaid.md`. classDef hex pairs meeting WCAG AA (e.g. `fill:#2c3e50,color:#ffffff`), `<br/>` not `\n`, no parentheses in labels, `:::className` inline, single subgraph, 3+ character node IDs.

After all alternatives are presented, write a comparison table:

| Alternative | Hides complexity | Test impact | Integration cost | Risk |
|---|---|---|---|---|

Then give an opinionated recommendation (possibly a hybrid).

## RFC Templates

### Template A: GitHub / GitLab issue

    ## Lens
    <architecture | testability | performance | security>

    ## Problem
    <the friction observed; affected files or modules; severity or dependency category>

    ## Proposed Solution
    <chosen alternative's interface or approach + usage example>

    ## Diagram
    <mermaid before/after, required>

    ## Dependency or Risk Strategy
    <which category applies; how dependencies or risks are handled>

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
