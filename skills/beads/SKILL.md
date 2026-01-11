---
description: Distributed, git-backed graph issue tracker for AI agents - persistent, structured memory for long-horizon tasks
model: opus
---

# Beads - Task Tracking for AI Agents

You are tasked with using **Beads (bd)** for task tracking and dependency management. Beads is a distributed, git-backed graph issue tracker designed specifically for AI agents to maintain persistent memory across long-horizon tasks.

## Core Concepts

- **Git as Database**: Issues are stored as JSONL in `.beads/` directory, versioned with git
- **Hash-based IDs**: Tasks get IDs like `bd-a1b2` that prevent merge collisions
- **Dependency Graph**: Track task relationships (blocks, related, parent-child)
- **Stealth Mode**: Use `--stealth` flag to track locally without committing to repo

## Essential Commands

```bash
# List tasks ready to work on (no open blockers)
bd ready

# Create a new task with priority
bd create "Task title" -p 0    # P0 = critical
bd create "Task title" -p 1    # P1 = high
bd create "Task title" -p 2    # P2 = normal

# Link tasks together
bd dep add <child-id> <parent-id>          # child is blocked by parent
bd dep add <child-id> <parent-id> -t related  # just related, not blocking

# View task details and audit trail
bd show <id>

# Mark task status
bd close <id>        # mark as completed
bd block <id>        # mark as blocked

# Show hierarchical views
bd show <epic-id> --tree    # show epic with subtasks
```

## Hierarchy & Epics

Beads supports hierarchical task IDs for breaking down large work:

```
bd-a3f8         (Epic)
bd-a3f8.1       (Sub-task)
bd-a3f8.1.1     (Sub-sub-task)
```

## Initialization

```bash
# One-time setup (run by human)
bd init

# Stealth mode - track locally without committing to main repo
bd init --stealth
```

## Installation Options

```bash
# npm
npm install -g @beads/bd

# Homebrew
brew install steveyegge/beads/bd

# Go
go install github.com/steveyegge/beads/cmd/bd@latest

# curl install script
curl -fsSL https://raw.githubusercontent.com/steveyegne/beads/main/scripts/install.sh | bash
```

## When to Use Beads

Use Beads when:
- Working on multi-phase implementations that need persistent tracking
- Managing complex task dependencies
- Need to maintain state across context compactions
- Coordinating work across multiple agents or sessions
- Want git-backed task history with full audit trail

## Workflow Pattern

1. **Initialization**: Run `bd init` once to set up
2. **Create Epic**: `bd create "Feature implementation" -p 0`
3. **Break Down**: Create subtasks with hierarchical IDs
4. **Link Dependencies**: `bd dep add` to establish blockers
5. **Work**: Use `bd ready` to find unblocked tasks
6. **Update**: Mark tasks complete/close as you finish

## Output Format

Beads outputs JSON by default for easy parsing by AI agents. Use `--json` flag explicitly if needed:

```bash
bd ready --json
```

## Stealth Mode

For personal task tracking without polluting the shared repo:

```bash
bd init --stealth
```

This creates `.beads/` but doesn't commit files to git. Perfect for personal use on shared projects.

## Further Reading

- GitHub: https://github.com/steveyegne/beads
- Documentation: Agent workflow, sync branch mode, troubleshooting, FAQ
- Community tools: UIs, extensions, integrations in docs/COMMUNITY_TOOLS.md
