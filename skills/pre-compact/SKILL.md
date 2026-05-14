---
name: pre-compact
description: >
  Save critical session context to an auto-memory snapshot file before the user
  runs /compact. Captures task, progress, decisions, next step, blockers, git
  state, and modified files. Use when the user asks to prepare for compaction,
  checkpoint a session, or save context before /compact.
---

# Pre-Compact: Save Session Context

Save critical context before compaction. Do NOT run `/compact` — let the user decide when.

## Step 1: Get session ID

```bash
SESSION_ID=$(python3 -c "import json; print(json.load(open('$HOME/.claude/sessions/$PPID.json'))['sessionId'][:8])")
```

Fallback if session file missing: `SESSION_ID=$(date +%Y%m%d_%H%M%S)`

## Step 2: Gather and save (single pass)

Run in parallel: `git status`, `git branch --show-current`, `git diff --stat`, and TaskList check.

Then write ONE snapshot file to the project's auto-memory directory.

**Filename**: `session_snapshot_{SESSION_ID}.md`

**Template**:
```markdown
---
name: session_snapshot_{SESSION_ID}
description: Pre-compact snapshot - {brief description of current work}
type: project
---

## Task
<!-- Ticket ID + one-line goal. Why are we doing this? -->
- **Ticket**: {ticket or goal}
- **Objective**: {what we're trying to achieve}

## Progress
<!-- Bullets: what's done, what's left -->
- Done: {completed items}
- Remaining: {outstanding items}

## Decisions
<!-- Approach chosen and why. Rejected alternatives if relevant. -->
- {decision}: {reasoning}

## Next Step
<!-- The single immediate action post-compact Claude should take -->
- {next action}

## Blockers
<!-- Active errors, failing tests, open questions. Paste exact messages. Leave empty if none. -->
- {error or blocker}

## Git State
<!-- Filled from git commands — branch, uncommitted changes summary -->
- **Branch**: {branch}
- **Uncommitted**: {staged/unstaged/untracked summary}

## Modified Files
<!-- From git diff --stat output -->
- {file list}
```

## Step 3: After saving

1. Add one-line pointer to MEMORY.md (include session ID in the description)
2. Save key points to mem0 via `add_memory` (project name + "session-snapshot" + session ID tag)
3. Display prominently: **Session: {SESSION_ID}** — this line must survive compaction
4. Warn if uncommitted changes exist
5. Tell user to run `/compact` when ready
