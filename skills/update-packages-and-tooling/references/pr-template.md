# PR/MR description template

If the repo has its own PR/MR template, keep its structure and put these sections inside it.

```markdown
## Summary
<counts by tier; what was left out>

## Updated
### Patch/minor
| Package | From | To | Declared in |
| --- | --- | --- | --- |
### Major (approved)
| Package | From | To | Declared in | Changelog |
| --- | --- | --- | --- | --- |
### Runtime
| Item | From | To | Pin locations | Source |
| --- | --- | --- | --- | --- |
Indirect dependencies refreshed: <count>. Notable movers: <runtime-relevant or large jumps>.

## Held back
| Item | Version | Reason (cooldown / pin / conflict / deprecated / declined) |
| --- | --- | --- |

## Reverted
| Item | Failing job | Diagnosis |
| --- | --- | --- |

## Verification
- Checks run: <list>
- CI-only gates (not run locally): <list>
- Baseline: <result>
- After each tier: <result>
- CI: <result>
- Unverified: <yes/no, and why>

## Impact review
| Item | Verdict (safe / needs change / risky / unknown) | Changelog range read | Breaking surface found in this repo (file:line) |
| --- | --- | --- | --- |
Not individually reviewed: <indirect dependencies, with reason>.
Runtime pin changes: <list>

## Revert guide
| Tier | Commit |
| --- | --- |
```
