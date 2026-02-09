# Superpowers (with Beads Integration)

This is a fork of [obra/superpowers](https://github.com/obra/superpowers) with deep integration of [beads](https://github.com/steveyegge/beads) — a git-backed, dependency-aware issue tracker optimized for AI agent workflows.

## What's Different from Upstream

### Beads replaces markdown plans

The original superpowers uses markdown files in `docs/plans/` for implementation plans. This fork replaces that with beads dependency graphs:

- **writing-plans** creates beads issues with dependencies instead of markdown files
- **executing-plans** and **subagent-driven-development** use `bd ready` to drive task selection
- **brainstorming** saves design docs to `docs/brainstorm/` and creates a root beads issue
- **finishing-a-development-branch** verifies all beads tasks are closed before completing

### Key improvements

| Feature | Original (markdown) | This fork (beads) |
| ------- | ------------------- | ----------------- |
| Task ordering | Linear list in markdown | Dependency graph with `bd ready` |
| Cross-session persistence | Lost when context window fills | Persisted in `.beads/` database |
| Parallelism detection | Manual batching | Automatic via dependency analysis |
| Task state | Agent memory / TodoWrite | Durable `bd claim` / `bd close` |
| Session startup | No task context | `bd prime` injects current state |

### New skill: using-beads

Foundation skill that all other workflow skills depend on. Covers `bd` CLI usage, task body format, dependency types, and best practices.

## Prerequisites

In addition to the original superpowers requirements:

```bash
# Install beads CLI
go install github.com/steveyegge/beads/cmd/bd@latest

# Initialize in your project
cd your-project
bd init
```

## Original README

For the full original documentation (installation, workflow, philosophy, etc.), see the [upstream repository](https://github.com/obra/superpowers).
