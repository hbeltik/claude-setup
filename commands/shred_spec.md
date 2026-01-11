---
name: shred
description: Shred a spec file into atomic beads tasks with hierarchical structure and agent-assignable labels
---

# Spec Shredder

Atomize a specification document into granular, actionable beads tasks optimized for multi-agent parallel execution.

## Input

<spec_file>$ARGUMENTS</spec_file>

## Workflow

### Step 1: Read the Spec

Read and analyze the provided spec file to understand:
- Main feature/goal (becomes the epic)
- Major components/phases
- Individual requirements
- **True** blocking dependencies (not implicit ones)

### Step 2: Verify Beads is Initialized

```bash
bd ready
```

If beads isn't initialized, inform the user to run `bd init` first.

### Step 3: Create Epic

Create a top-level epic for the entire spec with `-t epic`:

```bash
bd create "Epic: [Spec Title]" -t epic -p 1
# Returns: bd-a1b2c3
```

### Step 4: Shred into Hierarchical Tasks

Create child tasks under the epic using `--parent`. This creates hierarchical IDs (`bd-a1b2c3.1`, `.2`, etc.) that show structure without blocking:

```bash
# Foundation tasks
bd create "Define [X] type interface" -p 0 --parent bd-a1b2c3
# Returns: bd-a1b2c3.1

bd create "Add [field] to [Schema]" -p 0 --parent bd-a1b2c3
# Returns: bd-a1b2c3.2

# Backend tasks
bd create "Create [X] API endpoint" -p 1 --parent bd-a1b2c3
# Returns: bd-a1b2c3.3

# UI tasks
bd create "Build [Component] skeleton" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.4
```

**Priority layers:**

| Priority | Layer | Examples |
|----------|-------|----------|
| P0 | Foundation | Schema, types, config |
| P1 | Backend | API, queries, business logic |
| P2 | UI | Components, styling, wiring |
| P3 | Polish | Tests, error handling, docs |

**Atomic = one sentence description, 15min-1hr to complete.**

### Step 5: Label Tasks by Domain

Add labels for filtering and agent assignment:

```bash
# Domain labels
bd label add bd-a1b2c3.1 "schema"
bd label add bd-a1b2c3.2 "schema"
bd label add bd-a1b2c3.3 "backend"
bd label add bd-a1b2c3.4 "frontend"

# Optional: agent affinity labels
bd label add bd-a1b2c3.3 "agent:backend-specialist"
bd label add bd-a1b2c3.4 "agent:ui-specialist"
```

### Step 6: Add Only Real Dependencies

**Only** add blocking dependencies when work truly cannot proceed without another task:

```bash
# DO: Add when there's a true blocker
bd dep add bd-a1b2c3.5 bd-a1b2c3.1  # Query truly needs schema to exist

# DON'T: Add implicit or preference-based deps
# (Frontend and backend CAN work in parallel with mocked data)
```

Dependency flow (only when truly blocking): schema → backend → UI → polish

### Step 7: Summary

After shredding, provide:

```markdown
## Spec Shredded: [Title]

**Epic:** bd-XXXX
**Total Tasks:** N

**Breakdown:**
- P0 (foundation): X tasks
- P1 (backend): Y tasks
- P2 (frontend): Z tasks
- P3 (polish): W tasks

**Labels Applied:**
- `schema`: A tasks
- `backend`: B tasks
- `frontend`: C tasks

**Ready for Parallel Work:**
[output of bd ready]
```

### Step 8: Multi-Agent Assignment (Optional)

If assigning to multiple agents:

```bash
# Pin tasks to specific agents
bd pin bd-a1b2c3.1 --for agent-backend --start
bd pin bd-a1b2c3.4 --for agent-frontend --start

# Check assignments
bd list --label backend --status open
bd list --label frontend --status open
```

### Step 9: Next Steps

Suggest:
- `bd ready` - see all unblocked tasks
- `bd list --label backend` - filter by domain
- `bd dep tree bd-XXXX` - view dependency structure
- `/work` - start executing tasks
