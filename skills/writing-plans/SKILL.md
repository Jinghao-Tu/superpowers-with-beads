---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans as beads dependency graphs. Each task becomes a beads issue with full implementation details, and dependencies between tasks are explicitly modeled. This replaces markdown plan files with a queryable, stateful task graph.

Assume the engineer executing each task has zero context for the codebase and questionable taste. Document everything: which files to touch, complete code, testing, docs to check, how to verify. DRY. YAGNI. TDD. Frequent commits.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan as a beads dependency graph."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Prerequisites:**
- **REQUIRED SUB-SKILL:** superpowers:using-beads
- A root feature issue must exist (created by brainstorming skill)
- `bd` CLI available and project initialized (`bd init`)

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**No longer a markdown file.** The plan is a beads dependency graph. But the root feature issue (created by brainstorming) serves as the plan header, with its body containing:
- Goal
- Architecture
- Tech Stack
- Link to design document

## Creating Tasks as Beads Issues

### Step 1: Identify the root feature issue

```bash
bd list --json --status open --priority 0
```

Record the feature issue ID (e.g., `bd-a1b2`).

### Step 2: Create task issues

For each task, create a beads issue with complete implementation details in the body:

```bash
bd create "Task 1: [Component Name]" -p 1 --body "$(cat <<'TASKEOF'
**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**
...complete code...

**Step 2: Run test to verify it fails**
Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**
...complete code...

**Step 4: Run test to verify it passes**
Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

**Step 5: Commit**
```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
TASKEOF
)"
```

### Step 3: Set up dependencies

```bash
# Every task is a child of the root feature issue
bd dep add <task-id> <feature-id> --type parent

# Task 2 depends on Task 1 (sequential)
bd dep add <task2-id> <task1-id> --type blocks

# Independent tasks: no blocks dependency → both appear in bd ready
```

### Step 4: Verify the dependency graph

```bash
bd ready --json           # Tasks with no blockers
bd list --json --status open --parent <feature-id>  # All tasks
```

## Dependency Modeling Guidelines

**Use `blocks` when:**
- Task B imports/uses code created by Task A
- Task B modifies files created by Task A
- Task B's tests depend on Task A's implementation

**Leave unlinked (both ready) when:**
- Tasks touch different files/modules
- Tasks can be implemented in any order

**Prefer shallow dependency trees** — avoid long chains. Structure as a wide graph where many tasks are independently ready.

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD, frequent commits
- **One beads issue per task** — keep tasks atomic
- **Set all dependencies during plan creation**, not during execution

## Execution Handoff

After creating all tasks and dependencies:

```
Plan created as beads dependency graph.
Root feature: <feature-id> — "<feature name>"
Total tasks: N
Currently ready (no blockers): M tasks

Run `bd ready` to see available tasks.
```

**Two execution options:**

**1. Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration

**2. Parallel Session (separate)** - Open new session in worktree, batch execution with checkpoints

**Which approach?**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Stay in this session
- Fresh subagent per task + code review

**If Parallel Session chosen:**
- Guide them to open new session in worktree
- **REQUIRED SUB-SKILL:** New session uses superpowers:executing-plans
