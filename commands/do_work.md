---
name: work
description: Find work → claim → execute → close → compound learnings → quality gate → verify → commit → next
---

# Work

Automated workflow for completing tasks from the beads issue tracker. This command integrates with the compound-engineering workflow for systematic execution.

## Workflow

### Step 1: Find Available Work

Run `bd ready` to see all available tasks. Review the list and select the highest priority task to work on.

If multiple tasks are available, pick the highest priority task or one that is a prerequisite for other tasks.

### Step 2: Claim the Task

Once a task is selected, claim it using:
```bash
bd update <id> --status in_progress
```

### Step 3: Execute with Compound-Engineering Workflow

Use the comprehensive work execution workflow:

```bash
/compound-engineering:workflows:work <beads id>
```

### Step 4: Close the Task

After the workflow completes successfully:

```bash
bd close <id>
```

### Step 5: Compound Learnings (especially after debugging sessions)

```bash
/compound-engineering:workflows:compound
```

Document the problem, solution, and takeaways for future reference.
- Debugging sessions: root cause, fix applied, how to prevent recurrence
- Feature work: design decisions, trade-offs, patterns used
- Creates permanent team knowledge, not tribal knowledge

### Step 6: Quality Gate

```bash
qlty check --fix
```

### Step 7: Verify Changes

**If UI change:** Ask the user to verify the change in the browser.

**If backend change:** Write Convex unit tests and run them:
```bash
cd packages/backend && npx convex-test
```

### Step 8: Commit if OK

Commit directly without asking for permissions. Use `/commit` skill, NOT `git commit` directly.

### Step 9: Ask for Next Task

"Task complete. What's next?"
