---
name: spec-builder
description: Implements task from spec-planner and reviews code. Usage: /spec-builder <task-folder>
---

# spec-builder

Implements a task that was planned by `/spec-planner` and calls code-reviewer at the end.

## Usage

```
/spec-builder <task-folder>
```

Examples:
- `/spec-builder add-load-creators-button`
- `/spec-builder IN-1817`

---

## Workflow

### Phase 1: Parse Command

1. Extract task folder name from command arguments
2. **Validate folder name:**
   - Reject if contains `..` (path traversal)
   - Reject if contains `/` or `\`
   - Reject if starts with `-` (could be interpreted as flag)
   - Only allow alphanumeric characters, hyphens, and underscores
3. Validate folder exists in current directory
4. Check for required files (AGENTS.md, plans.md)

### Phase 2: Read Task Files

1. Read `<task-folder>/AGENTS.md` for task context
2. Read `<task-folder>/plans.md` for implementation steps
3. Parse prerequisites and acceptance criteria

### Phase 3: Execute Implementation

1. Check all prerequisites are met
2. Execute each step in plans.md sequentially
3. For each step:
   - Review the files to modify
   - Make required changes
   - Verify changes align with step details

### Phase 4: Code Review

1. After implementation completes, launch code-reviewer subagent
2. Pass task context to reviewer
3. Include paths to modified files

---

## Implementation Notes

### Folder Resolution

- Task folder should be in current working directory
- Support both slugified names and Jira ticket IDs
- If folder not found, display error with suggestion

### Execution Approach

- Execute steps in order (Priority 1 first)
- Each step should be verified before proceeding
- **Error handling:**
  - If a step fails, stop and report error to user
  - Include step name and error details in error message
  - User can fix issues and re-run from failed step
- Validate plans.md has expected structure before execution

### Code Reviewer Call

Use Task tool to launch code-reviewer:
- subagent_type: `code-reviewer`
- description: "Review implemented task"
- Prompt should include:
  - Task title and description from AGENTS.md
  - Summary of what was implemented
  - List of files modified (absolute paths)
- If subagent fails, report error to user but continue

---

## Example

### Command

```
/spec-builder add-load-creators-button
```

### Execution Flow

1. Parse folder name: `add-load-creators-button`
2. Read `add-load-creators-button/AGENTS.md`
3. Read `add-load-creators-button/plans.md`
4. Execute Step 1: Add Button to CampaignInfo Component
5. Execute Step 2: Create API Service Function
6. Launch code-reviewer subagent with implementation details
