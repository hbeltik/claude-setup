---
description: Shred markdown specs into granular beads tasks with hierarchical structure, domain labels, and minimal dependencies - optimized for multi-agent parallel execution
model: opus
---

# Spec Shredder - Atomize Specs for Parallel Agent Execution

You are tasked with **shredding** specification documents into the smallest possible atomic tasks, tracked via the Beads (`bd`) issue tracker. The goal is maximum granularity and parallelism - each task should be completable in minutes to an hour, and agents should be able to work independently.

## The Job

Take a spec file (markdown) and decompose it into:
1. An epic task representing the entire spec
2. Hierarchical child tasks under the epic (auto-numbered: `.1`, `.2`, etc.)
3. Domain labels for filtering and agent assignment
4. **Minimal** blocking dependencies (only when truly required)
5. Priority ordering based on logical flow

## Shredding Philosophy

**Atomic = Better.** A task should be:
- Completable in one focused session (15min - 1hr)
- Testable with a single verification step
- Independent enough to work on without other context
- Small enough that failure is cheap to retry

**Parallel = Better.** Tasks should be:
- Independent by default (no dependency unless truly blocking)
- Labeled by domain for easy filtering and assignment
- Assignable to different agents simultaneously

### Right-sized tasks (good):
- "Add `status` field to User schema"
- "Create empty `UserCard` component skeleton"
- "Wire `onClick` handler to open modal"
- "Add validation for email format"
- "Write unit test for `calculateTotal` function"

### Too big (shred these further):
- ❌ "Build user profile page" → Split into: schema, API endpoint, component skeleton, data fetching, styling, validation, tests
- ❌ "Add authentication" → Split into: schema, password hashing, login endpoint, session middleware, login form, logout, tests
- ❌ "Implement search" → Split into: query builder, index setup, API endpoint, search input, results display, pagination

**Rule of thumb:** If describing the task takes more than one sentence, shred it more.

## Workflow

### Step 1: Read and Analyze the Spec

```bash
# Read the spec file provided by the user
cat path/to/spec.md
```

Identify:
- Main feature/goal (becomes the epic)
- Major components/phases
- Individual requirements
- **True** blocking dependencies (rare!)

### Step 2: Create the Epic

```bash
# Create the top-level epic with explicit type
bd create "Epic: [Spec Title]" -t epic -p 1
# Returns: bd-a1b2c3
```

### Step 3: Create Hierarchical Child Tasks

Use `--parent` to create tasks under the epic. This gives them hierarchical IDs that show structure **without** creating blocking dependencies:

```bash
# Foundation tasks (P0)
bd create "Define UserProfile type interface" -p 0 --parent bd-a1b2c3
# Returns: bd-a1b2c3.1

bd create "Add activity field to User schema" -p 0 --parent bd-a1b2c3
# Returns: bd-a1b2c3.2

# Backend tasks (P1)
bd create "Create getUserProfile query" -p 1 --parent bd-a1b2c3
# Returns: bd-a1b2c3.3

bd create "Create updateUserName mutation" -p 1 --parent bd-a1b2c3
# Returns: bd-a1b2c3.4

# Frontend tasks (P2)
bd create "Create UserProfile component skeleton" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.5

bd create "Wire avatar/name/email to UserProfile" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.6
```

**Layer breakdown by priority:**

| Priority | Layer | Examples |
|----------|-------|----------|
| P0 (critical) | Foundation | Schema, types, config |
| P1 (high) | Backend | API, queries, business logic |
| P2 (normal) | UI/Frontend | Components, styling, wiring |
| P3 (low) | Polish | Tests, error handling, docs |

### Step 4: Label Tasks by Domain

Labels enable filtering by domain and agent assignment:

```bash
# Domain labels (required)
bd label add bd-a1b2c3.1 "schema"
bd label add bd-a1b2c3.2 "schema"
bd label add bd-a1b2c3.3 "backend"
bd label add bd-a1b2c3.4 "backend"
bd label add bd-a1b2c3.5 "frontend"
bd label add bd-a1b2c3.6 "frontend"

# Agent affinity labels (optional, for multi-agent setups)
bd label add bd-a1b2c3.3 "agent:backend-specialist"
bd label add bd-a1b2c3.5 "agent:ui-specialist"
```

**Standard domain labels:**
- `schema` - Data models, types, migrations
- `backend` - API endpoints, queries, business logic
- `frontend` - Components, styling, UI wiring
- `infra` - Config, deployment, CI/CD
- `test` - Unit tests, integration tests
- `docs` - Documentation, comments

### Step 5: Add Only REAL Dependencies

**Key principle:** Don't add dependencies unless one task truly cannot start without another completing.

```bash
# DO: Add when there's a true blocker
bd dep add bd-a1b2c3.3 bd-a1b2c3.1  # Query needs type definition to compile

# DON'T: Add preference-based or implied deps
# Frontend CAN work with mocked data while backend is built
# Tests CAN be written before implementation (TDD)
```

**When to add dependencies:**
- Schema field must exist before query references it
- API must exist before integration test calls it
- Type must be defined before code imports it

**When NOT to add dependencies:**
- "Backend should be done before frontend" → No, they can mock/parallel
- "Tests should come after code" → No, TDD is valid
- "Design before implementation" → No, iterate together

### Step 6: Verify Structure

```bash
# Show the epic with full tree
bd dep tree bd-a1b2c3

# Show ALL tasks that can be worked on in parallel
bd ready

# Filter by domain
bd list --label backend --status open
bd list --label frontend --status open
```

### Step 7: Multi-Agent Assignment (Optional)

For parallel execution across multiple agents:

```bash
# Pin tasks to specific agents
bd pin bd-a1b2c3.1 --for agent-1 --start
bd pin bd-a1b2c3.3 --for agent-2 --start
bd pin bd-a1b2c3.5 --for agent-3 --start

# Check what each agent has
bd hook --agent agent-1
bd hook --agent agent-2

# See all in-progress work
bd list --status in_progress
```

## Output Format

After shredding, provide a summary:

```markdown
## Spec Shredded: [Spec Title]

**Epic:** bd-XXXX
**Total Tasks:** N

**Breakdown by Priority:**
- P0 (foundation): X tasks
- P1 (backend): Y tasks
- P2 (frontend): Z tasks
- P3 (polish): W tasks

**Breakdown by Domain:**
- `schema`: A tasks
- `backend`: B tasks
- `frontend`: C tasks

**Blocking Dependencies:** M (should be minimal!)

**Ready for Parallel Work:**
`bd ready` shows N unblocked tasks
```

## Example Shred

**Input spec excerpt:**
```markdown
## User Profile
- Display user avatar, name, email
- Allow editing name
- Show activity history
```

**Shredded output:**
```bash
# Create Epic
bd create "Epic: User Profile" -t epic -p 1
# Returns: bd-a1b2c3

# Foundation (P0)
bd create "Define UserProfile type interface" -p 0 --parent bd-a1b2c3
# Returns: bd-a1b2c3.1

bd create "Add activity field to User schema" -p 0 --parent bd-a1b2c3
# Returns: bd-a1b2c3.2

# Backend (P1)
bd create "Create getUserProfile query" -p 1 --parent bd-a1b2c3
# Returns: bd-a1b2c3.3

bd create "Create updateUserName mutation" -p 1 --parent bd-a1b2c3
# Returns: bd-a1b2c3.4

bd create "Create getUserActivity query" -p 1 --parent bd-a1b2c3
# Returns: bd-a1b2c3.5

# Frontend (P2)
bd create "Create UserProfile component skeleton" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.6

bd create "Create ActivityList component skeleton" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.7

bd create "Create EditNameModal component skeleton" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.8

bd create "Wire avatar/name/email to UserProfile" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.9

bd create "Wire activity data to ActivityList" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.10

bd create "Wire EditNameModal to updateUserName" -p 2 --parent bd-a1b2c3
# Returns: bd-a1b2c3.11

# Polish (P3)
bd create "Add loading state to UserProfile" -p 3 --parent bd-a1b2c3
# Returns: bd-a1b2c3.12

bd create "Add empty state to ActivityList" -p 3 --parent bd-a1b2c3
# Returns: bd-a1b2c3.13

bd create "Add validation to EditNameModal" -p 3 --parent bd-a1b2c3
# Returns: bd-a1b2c3.14

# Labels for domain filtering
bd label add bd-a1b2c3.1 "schema"
bd label add bd-a1b2c3.2 "schema"
bd label add bd-a1b2c3.3 "backend"
bd label add bd-a1b2c3.4 "backend"
bd label add bd-a1b2c3.5 "backend"
bd label add bd-a1b2c3.6 "frontend"
bd label add bd-a1b2c3.7 "frontend"
bd label add bd-a1b2c3.8 "frontend"
bd label add bd-a1b2c3.9 "frontend"
bd label add bd-a1b2c3.10 "frontend"
bd label add bd-a1b2c3.11 "frontend"
bd label add bd-a1b2c3.12 "frontend"
bd label add bd-a1b2c3.13 "frontend"
bd label add bd-a1b2c3.14 "frontend"

# MINIMAL dependencies - only true blockers
bd dep add bd-a1b2c3.3 bd-a1b2c3.1  # query needs type to compile
bd dep add bd-a1b2c3.5 bd-a1b2c3.2  # activity query needs schema field

# That's it! Other tasks can work in parallel
```

**Result:** 14 atomic tasks from 3 bullet points, with only 2 true dependencies.

Most tasks are immediately available for parallel execution via `bd ready`.

## Checklist Before Finishing

- [ ] Epic created with `-t epic` flag
- [ ] All tasks use `--parent` for hierarchical IDs
- [ ] Every task is atomic (one sentence description)
- [ ] Domain labels applied to all tasks
- [ ] Dependencies are **minimal** (only true blockers)
- [ ] No circular dependencies
- [ ] `bd ready` returns many parallel-ready tasks

## Invocation

When the user provides a spec file:

```bash
# 1. Read the spec
cat path/to/spec.md

# 2. Shred it using the workflow above

# 3. Verify parallel readiness
bd ready
bd dep tree <epic-id>

# 4. Optional: assign to agents
bd pin <task-id> --for <agent-name> --start
```
