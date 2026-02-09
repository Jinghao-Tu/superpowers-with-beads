---
name: executing-plans
description: Use when you have a beads dependency graph to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Query beads for ready tasks, execute in batches, report for review between batches.

**Core principle:** `bd ready` drives task selection. Batch execution with checkpoints for architect review.

**Announce at start:** "I'm using the executing-plans skill to implement tasks from the beads dependency graph."

**Prerequisites:**
- **REQUIRED SUB-SKILL:** superpowers:using-beads
- Beads initialized with tasks (created by writing-plans skill)
- `bd` CLI available

## The Process

### Step 1: Load and Review Tasks
1. Run `bd prime` to understand overall project state
2. Run `bd ready --json` to see available tasks
3. Run `bd list --json --status open` to see all remaining tasks
4. Review task contents critically — identify any questions or concerns
5. If concerns: Raise them with your human partner before starting
6. If no concerns: Create TodoWrite from ready tasks and proceed

### Step 2: Execute Batch
**Default: First 3 ready tasks**

For each task:
1. `bd update <id> --claim` — atomically claim the task
2. `bd show <id> --json` — read full task body with implementation steps
3. Follow each step exactly (task body has bite-sized steps)
4. Run verifications as specified
5. `bd close <id> --reason "Implemented and tested"`
6. Mark as completed in TodoWrite

### Step 3: Refresh and Report
When batch complete:
1. `bd ready --json` — check for newly unblocked tasks
2. Show what was implemented
3. Show verification output
4. Report newly ready tasks (previously blocked, now unblocked)
5. Say: "Ready for feedback."

### Step 4: Continue
Based on feedback:
- Apply changes if needed
- Execute next batch of ready tasks
- Repeat until no open tasks remain

### Step 5: Complete Development

After all tasks complete and verified:
- `bd list --json --status open` — confirm no remaining tasks
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-batch (missing dependency, test fails, instruction unclear)
- Task body has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly
- `bd ready` returns no tasks but open tasks remain (circular dependency)

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates tasks based on your feedback
- Fundamental approach needs rethinking
- New tasks are added to the dependency graph

**Don't force through blockers** - stop and ask.

## Remember
- Use `bd ready` to determine task order, not assumptions
- `bd claim` before starting each task
- `bd close` immediately after task passes verification
- Follow task body steps exactly
- Don't skip verifications
- Reference skills when task body says to
- Between batches: report + refresh ready tasks
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent

## Integration

**Required workflow skills:**
- **superpowers:using-beads** - REQUIRED: Task management via beads
- **superpowers:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
- **superpowers:writing-plans** - Creates the beads dependency graph this skill executes
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
