---
name: using-beads
description: Use beads (bd) for structured, dependency-aware task management throughout the superpowers workflow
---

# Using Beads

## Overview

Beads (`bd`) is a git-backed, dependency-aware issue tracker optimized for AI agent workflows. It replaces markdown plan files with a queryable dependency graph, providing persistent task memory across sessions, automatic ready-work detection, and structured state management.

**All superpowers workflow skills (writing-plans, executing-plans, subagent-driven-development) depend on beads for task management.**

## Prerequisites

- `bd` CLI installed: `go install github.com/steveyegge/beads/cmd/bd@latest`
- Project initialized: `bd init` (creates `.beads/` directory)

If `.beads/` directory does not exist in the project root, run `bd init` before proceeding.

## Core CLI Commands

### Task Creation

```bash
# Create a task with priority and body
bd create "Task title" -p <priority> --body "detailed description"

# Priority: 0 = critical, 1 = high, 2 = medium, 3 = low
```

### Dependency Management

```bash
# Task B is blocked by Task A (B cannot start until A is done)
bd dep add <taskB-id> <taskA-id> --type blocks

# Parent-child relationship (feature → subtasks)
bd dep add <child-id> <parent-id> --type parent

# Related tasks (informational, no blocking)
bd dep add <taskA-id> <taskB-id> --type related
```

### Task Lifecycle

```bash
# See what's ready to work on (no open blockers)
bd ready --json

# Claim a task (atomically set assignee + status to in_progress)
bd update <id> --claim

# View full task details
bd show <id> --json

# Close a completed task
bd close <id> --reason "Implemented and tested"

# List tasks with filters
bd list --json --status open
bd list --json --status open --parent <feature-id>
```

### Context & Sync

```bash
# Get optimized workflow context (priority breakdown, blocking info, ready work)
bd prime

# Force sync to git
bd sync
```

## Task Body Format

When creating tasks (used by writing-plans skill), the issue body follows this structure:

```markdown
**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**

\```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
\```

**Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**

\```python
def function(input):
    return expected
\```

**Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

**Step 5: Commit**

\```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
\```
```

## Dependency Types

| Type | Meaning | Effect |
|------|---------|--------|
| `blocks` | Task A blocks Task B | B won't appear in `bd ready` until A is closed |
| `parent` | Task is a subtask of a feature | Organizational grouping |
| `related` | Tasks are related | Informational only, no blocking |

## Best Practices

- **Always use `--json`** for programmatic output parsing
- **Use `bd ready`** to determine next actions, not manual dependency analysis
- **`bd close` immediately** after a task passes all reviews
- **`bd prime`** at session start to understand current project state
- **One `bd create` per task** — keep tasks atomic (2-5 minutes each)
- **Set dependencies during plan creation**, not during execution
- **Use `blocks` type** for sequential dependencies; leave independent tasks unlinked so they appear in `bd ready` simultaneously

## Integration with Superpowers Workflow

```
brainstorming
  → bd create "Feature: <name>" (root issue)

writing-plans
  → bd create per task + bd dep add for dependencies
  → Result: dependency graph, not a markdown file

executing-plans / subagent-driven-development
  → bd ready → bd claim → execute → bd close → repeat

finishing-a-development-branch
  → bd list --status open (verify all tasks closed)
```
