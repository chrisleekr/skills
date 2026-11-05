# Solution Presentation

Every alternative produced in step 6 must include:

1. **Interface / approach**: types, methods, params, or algorithmic shape
2. **Usage example**: how callers integrate
3. **What it hides**: complexity or risk now encapsulated
4. **Dependency or risk strategy**: which category applies
5. **Trade-offs**: what gets worse, not only better
6. **Mermaid before/after diagram**: two diagrams, type per § Mode Matrix. Follow these rules: classDef hex pairs meeting WCAG AA (e.g. `fill:#2c3e50,color:#ffffff`), `<br/>` not `\n`, no parentheses in labels, `:::className` inline, single subgraph, 3+ character node IDs.

In `ui` mode the required mermaid before/after is a user-flow flowchart or a layout
map, and each alternative references its before screenshots by saved path.

After all alternatives are presented, write a comparison table:

| Alternative | Hides complexity | Test impact | Integration cost | Risk |
|---|---|---|---|---|

Then give an opinionated recommendation (possibly a hybrid).
