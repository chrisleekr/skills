---
name: clean-comments
description: >
  Review, clean, and improve code comments, docstrings, and inline documentation.
  Removes noise while ensuring concise reasoning remains for future developers.
  Use when reviewing comments, tidying docs, or improving code readability.
---

# Clean Comments Skill

## Core Principle

Comments are compensations for failure to express intent in code.
The code is the only source of truth — comments age, mislead, and rot.
Invest energy in expressive naming and function extraction FIRST.

However, when code alone cannot communicate the WHY — business reasoning,
non-obvious constraints, external system dependencies — a concise comment
is not a failure but a gift to the next developer reading this code cold.
The goal is not zero comments. It is zero _useless_ comments.

---

## Step 0 — Scope and Understand

**Only review comments in code changed on the current branch.**

1. **Identify the diff scope.** Determine the base branch, then gather all changed files:
   - Committed changes: `git diff main...HEAD --name-only`
   - Uncommitted staged: `git diff --cached --name-only`
   - Uncommitted unstaged: `git diff --name-only`
     Combine these into the review scope. Do NOT review files outside this scope.

2. **Read each changed file's diff.** Use `git diff main...HEAD -- <file>` to see
   exactly what changed. Understand the purpose of each change — new feature, bug fix,
   refactor — before evaluating any comment.

3. **Identify the audience.** Who will read this code next? A comment that seems obvious
   to the author may be critical for an on-call engineer at 3am or a new team member
   in a different domain.

Only review comments that are within or directly adjacent to the changed lines.
Comments outside the diff are out of scope.

---

## Decision Rule

Ask: **"Does the function name + signature fully describe what it does?"**

- YES → skip the docstring entirely
- NO → rename the function first; if still unclear, add a WHY comment

---

## C1–C5: The Five Comment Heuristics

**C1 — No metadata in comments**
Author names, dates, ticket IDs, change logs belong in Git, not source files.
DELETE: `// Added by Jane, 2024-03-01, fixes JIRA-1234`
DELETE: (FM-9), T020
DELETE: Verified 2026-04-18

**C2 — Delete obsolete comments**
If a comment describes code that no longer exists or has changed, it is
actively harmful — a "floating island of misdirection". DELETE immediately.
DO NOT update to preserve history; Git has the history.

**C3 — No redundant comments**
If the comment restates what the code says, it adds zero signal.
DELETE:
i++; // increment i
return user; // returns the user
KEEP:
i++; // compensate for 1-based display index (API contract)

**C4 — Write comments well**
If a comment is worth writing, make it count:

- One clear sentence, grammatically correct
- State the WHY, not the WHAT
- No rambling, no obvious statements, no filler
- Apply the "new team member" test: would someone reading this code cold
  understand the reasoning without the comment? If not, the comment stays.

**Transform, don't just delete.** When a comment is poorly written but
contains real signal, rewrite it into a concise WHY statement:

BEFORE: `// this handles the edge case`
AFTER: `// Upstream payment API returns duplicate webhooks within 50ms — deduplicate by idempotency key`

BEFORE: `// important! don't remove this`
AFTER: `// Rate limiter: AWS API Gateway enforces 10k req/s — this sleep prevents 429s during batch import`

**C5 — Never commit commented-out code**
DELETE it. Every time. No exceptions.
Git remembers it. Commented-out code is dead weight that rots and
distracts every future reader. Use `git stash` if you need it back.

---

## Good Comments (Keep or Write These)

| Type                    | Example                                                         |
| ----------------------- | --------------------------------------------------------------- |
| Legal/copyright header  | `// Copyright 2026 Acme Inc. Apache 2.0`                        |
| Explanation of WHY      | `// Must be 0 here — upstream API returns 1-based index`        |
| Warning of consequences | `// Not thread-safe. Callers must synchronise externally.`      |
| TODO with owner         | `// TODO(user): replace with streaming once SDK supports it`    |
| Public API docstring    | Only for exported/public interfaces consumed by others          |
| Algorithm rationale     | Explaining a non-obvious mathematical constraint or O(n) choice |

---

## When Comments Are Essential

Some contexts demand comments regardless of how clean the code is:

- **Business logic rationale**: Why this specific threshold, multiplier, or rule exists
- **External system constraints**: Upstream API quirks, vendor limitations, SLA-driven choices
- **Non-obvious performance decisions**: Why a less-readable approach was chosen for O(n) reasons
- **Temporal context**: Workarounds with expiration dates (`// TODO(owner): remove after API v3 migration`)
- **Cross-service contracts**: Assumptions about data formats from other services

Ask: "If I leave and a new developer changes this code, what will break
that they couldn't predict from the code alone?" — that answer is your comment.

---

## AI-Specific Anti-Patterns (Self-Check)

These are the most common patterns Claude and other AI agents generate
that must be actively suppressed:

❌ Boilerplate docstrings on trivial functions:
def get_user(id): ... ← no docstring needed

❌ Commenting every parameter that names itself:

# param id: the user id ← the name IS the documentation

❌ Section dividers and visual noise:

# ================== PROCESSING ==================

# // ---------------------------------------------------------------------------

❌ Restating what the next line does:

# Create a new connection

conn = create_connection()

❌ JSDoc/Javadoc on every getter/setter:

```js
/** Gets the name. @returns {string} The name. */
getName() { return this.name; }
```

Also guard against the opposite extreme — stripping useful context:

❌ Deleting a WHY comment because "the code should speak for itself":
`// Retry 3x — Stripe webhook delivery is eventually consistent` ← KEEP THIS

❌ Removing business context that no variable name can capture:
`// Legal requirement: AU consumer law mandates 30-day cooling-off` ← KEEP THIS

---

## Refactoring Over Commenting

When tempted to add an explanatory comment, try these first:

1. Rename the function to eliminate the need for the comment
2. Extract a well-named sub-function
3. Introduce an explanatory variable (`const isExpiredToken = ...`)
4. Use the Special Case pattern instead of a comment explaining a weird edge case

Only after exhausting these options should a comment be added.

---

## Quick Audit Checklist

When reviewing existing code for comment quality:

- [ ] Any metadata (author/date/ticket)? → C1: DELETE
- [ ] Any comment describing code that no longer exists? → C2: DELETE
- [ ] Any comment restating what the code says? → C3: DELETE
- [ ] Any poorly written, vague, or rambling comment? → C4: REWRITE or DELETE
- [ ] Any commented-out code? → C5: DELETE immediately
- [ ] Any boilerplate AI docstring on a trivial function? → DELETE
- [ ] Any comment that explains WHAT instead of WHY? → REWRITE or DELETE
- [ ] Any load-bearing WHY comment that was accidentally removed? → RESTORE

---

## Workflow

1. **Scope**: Identify changed files on the branch (committed + uncommitted)
2. **Understand**: Read the diffs, understand what changed and why
3. **Assess**: Apply the Decision Rule and Quick Audit Checklist (C1–C5 + AI anti-patterns) to comments within the diff
4. **Plan**: Present a structured summary of proposed changes to the user:

   For each file, list:
   | Line | Current Comment | Proposed Action | Reasoning |
   |------|----------------|-----------------|-----------|

   Actions: DELETE / REWRITE (show new text) / KEEP (explain why)

   **Wait for user approval before proceeding.**

5. **Act**: Apply only the approved changes
6. **Verify**: No commented-out code remains, no redundant docstrings, all surviving comments explain WHY not WHAT, no load-bearing context was lost

---

_Based on Clean Code (Martin, 2008), chapters 4 & 17; tars-agentic/agent-skills ch00, ch04, ch17._
