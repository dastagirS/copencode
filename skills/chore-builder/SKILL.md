---
name: chore-builder
description: Builds small tasks from user prompts and reviews code. Usage: /chore-builder <task-description>
---

# chore-builder

Builds small tasks from user prompts and calls code-reviewer at the end.

## Usage

```
/chore-builder <task-description>
```

Examples:
- `/chore-builder add validation to login form`
- `/chore-builder fix bug in user profile`

---

## Workflow

### Phase 1: Parse Command

1. Extract task description from command arguments
2. **Validate input:**
   - Reject if empty
   - Reject if starts with `-` (could be interpreted as flag)
   - Must be at least 3 characters

### Phase 2: Analyze Task

1. Understand the user's request
2. Identify the scope - keep it small and focused
3. Determine files/actions needed
4. Set clear acceptance criteria (1-3 items max)

### Phase 3: Execute Implementation

1. Find relevant files to modify
2. Make required changes
3. Verify changes work as expected

### Phase 4: Code Review

1. After implementation completes, launch code-reviewer subagent
2. Pass task context to reviewer
3. Include paths to modified files

---

## Implementation Notes

### Scope Guidelines

- Small, self-contained changes
- Single feature or bug fix
- Max 1-3 files to modify
- If task is too large, suggest splitting it

### Execution Approach

- Make changes directly without extensive planning
- Focus on getting it done correctly
- **Error handling:**
  - If a change fails, report error to user
  - Include details in error message

### Code Reviewer Call

Use Task tool to launch code-reviewer:
- subagent_type: `code-reviewer`
- description: "Review implemented chore"
- Prompt should include:
  - Task description from user prompt
  - Summary of what was implemented
  - List of files modified (absolute paths)
- If subagent fails, report error to user but continue

---

## Example

### Command

```
/chore-builder add validation to login form
```

### Execution Flow

1. Parse task: `add validation to login form`
2. Identify target: login form component
3. Find validation needs: email format, password length
4. Implement validation
5. Launch code-reviewer subagent with implementation details
