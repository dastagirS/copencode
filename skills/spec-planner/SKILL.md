---
name: spec-planner
description: Analyze task and generate task folder with AGENTS.md, plans.md, and prd.json. Usage: /spec-planner [task description]
---

# spec-planner

Analyze a task description and generate a complete task folder with AGENTS.md (task info), plans.md (implementation outline), and prd.json (user story breakdown).

## Usage

```
/spec-planner <task description>
```

Examples:
- `/spec-planner add load creators button to admin campaign`
- `/spec-planner implement user authentication flow`

---

## Workflow

### Phase 1: Clarify Requirements (Grill-Me Phase)

**CRITICAL: Before planning, interview the user relentlessly about every aspect of the task until reaching shared understanding.**

Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. **Explore the codebase first** to answer questions that can be determined from existing code.

**Investigate these areas thoroughly:**

1. **Scope Clarification**
   - What is the exact boundary of this task?
   - What is explicitly out of scope?
   - What are the success criteria?

2. **Dependency Resolution**
   - What prerequisites must be completed first?
   - Are there external API dependencies?
   - Are there design/mockup dependencies?

3. **Technical Approach** (explore codebase for these)
   - Which existing patterns should be followed?
   - Are there similar features already implemented?
   - What is the recommended data flow?
   - Where should new code be placed?

4. **Edge Cases & Risks**
   - What could go wrong?
   - How should errors be handled?
   - What are the performance considerations?

**For each question, provide your recommended answer based on codebase exploration while seeking user confirmation.**

**Document all findings:** Capture scope decisions, dependencies, technical approaches, and risk mitigations to include in AGENTS.md under a "Clarification Notes" section.

### Phase 2: Analyze Task

1. Parse the task description from command arguments
2. Identify:
   - Main action (add, implement, create, etc.)
   - Target (what is being modified/created)
   - Context (where, for whom, etc.)
3. Determine scope and requirements from context
4. Document findings from Phase 1

### Phase 3: Generate Task Folder Name

1. Check if task description contains a Jira ticket reference (format: XX-123 or similar):
   - If yes, extract the ticket ID (e.g., "IN-1817", "PROJ-123")
   - Use ticket ID as folder name
   - If no ticket found, proceed to step 2
2. If no Jira ticket, create slugified folder name from task description:
   - Take first 5-7 words of task description
   - Convert to lowercase
   - Replace spaces with hyphens
   - Remove special characters (keep only alphanumeric and hyphens)
   - Example: "Add load creators button to admin campaign" → "add-load-creators-button"
3. Check if folder exists in current directory
4. If exists, append incrementing number ("-2", "-3", etc.)

### Phase 4: Create Task Files

Create folder in current working directory and generate 3 files:

#### 1. AGENTS.md

Task-specific information document with:
- Task title and description
- Project context (name, ticket, branch)
- Acceptance criteria checklist
- Technical requirements
- Dependencies
- Current status

#### 2. plans.md

High-level implementation outline with:
- Overview (2-3 sentences)
- Prerequisites checklist
- Implementation steps (numbered, with actions, files, details)
- Testing strategy
- Risk mitigation
- Completion criteria

#### 3. prd.json

Task breakdown with:
- `project`: Detect from git remote or ask user
- `ticket`: Placeholder like "TASK-XXX"
- `branchName`: Suggested branch name (feature/TASK-XXX-description)
- `description`: Overall task summary
- `userStories`: Array of user stories with:
  - `id`: "US-001", "US-002", etc.
  - `title`: Brief story title
  - `description`: Detailed description "As a..., I want..., So that..."
  - `acceptanceCriteria`: Array of checkable criteria
  - `priority`: 1-10 (1 highest)
  - `passes`: false (not completed)
  - `notes`: Empty string

### Phase 5: Output

1. Create task folder in current directory
2. Write AGENTS.md, plans.md, and prd.json to folder
3. Display folder path and file summary

### Phase 6: Architecture Review

After creating task files, launch the-architect subagent to review the plan:

**Launch the-architect using Task tool:**
- subagent_type: `the-architect`
- description: "Review task architecture"
- Prompt should include:
  - Task folder path (absolute path to task folder)
  - AGENTS.md content (task description, acceptance criteria, technical requirements)
  - plans.md content (implementation steps, files to modify)
  - Request to verify: Does the plan follow existing codebase patterns?

**Review Focus:**
1. Does the implementation approach match existing patterns in the codebase?
2. Are the suggested file locations consistent with project structure?
3. Are there existing utilities/components that should be reused?
4. Does the plan account for project-specific constraints?
5. Are there scalability or architectural concerns?

**Response Handling:**
- If architect identifies pattern mismatches: Update plans.md with corrections
- If architect suggests better approaches: Incorporate recommendations
- Display architect's findings to user with specific recommendations

---

## Output Format

### Folder Structure

```
<current-directory>/
└── <task-slug>/
    ├── AGENTS.md
    ├── plans.md
    └── prd.json
```

### AGENTS.md Template

```markdown
# Task: <Task Title>

## Description
<Detailed task description from user input>

## Context
- Project: <project-name>
- Ticket: <ticket-id>
- Branch: <branch-name>

## Acceptance Criteria
- [ ] <Checkable criterion 1>
- [ ] <Checkable criterion 2>
- [ ] <Checkable criterion 3>

## Technical Requirements
- <requirement 1>
- <requirement 2>

## Dependencies
- [ ] <dependency 1>
- [ ] <dependency 2>

## Clarification Notes
**Scope:**
- In scope: <what's included>
- Out of scope: <what's excluded>

**Dependencies Resolved:**
- <prerequisite discovered during grill-me phase>

**Technical Approach:**
- <patterns confirmed during grill-me phase>
- <similar features found in codebase>

**Edge Cases Identified:**
- <edge case 1> → <mitigation>

**Open Questions:**
- <any remaining clarifications needed>

## Status
Not Started
```

### plans.md Template

```markdown
# Plan: <Task Title>

## Overview
<2-3 sentence summary of the implementation approach>

## Prerequisites
- [ ] <prerequisite 1>
- [ ] <prerequisite 2>

## Implementation Steps

### Step 1: <Step Name>
**Priority:** <1-10>
**Estimated Time:** <time estimate>

**Actions:**
1. <specific action>
2. <specific action>

**Files to Modify:**
- `<file-path-1>`
- `<file-path-2>`

**Details:**
<technical details>

**Verification:**
- [ ] <verification item>

---

### Step 2: <Step Name>
...

## Testing Strategy
- [ ] <test approach 1>
- [ ] <test approach 2>

## Risk Mitigation
- <risk 1> → <mitigation 1>
- <risk 2> → <mitigation 2>

## Completion Criteria
- [ ] All acceptance criteria met
- [ ] All tests passing
- [ ] Code reviewed
```

### prd.json Format

```json
{
  "project": "<project-name>",
  "ticket": "TASK-XXX",
  "branchName": "feature/TASK-XXX-description",
  "description": "<task description>",
  "userStories": [
    {
      "id": "US-001",
      "title": "<title>",
      "description": "As a <user>, I want <action>, so that <benefit>.",
      "acceptanceCriteria": [
        "<criterion 1>",
        "<criterion 2>"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

---

## Implementation Notes

### Folder Name Generation

**Priority 1: Jira Ticket Detection**
- Check task description for Jira ticket pattern (e.g., "IN-1817", "PROJ-123", "ABC-4567")
- Pattern: 2+ uppercase letters, hyphen, 1+ digits
- If found, use ticket ID as folder name directly
- Example: "Implement feature IN-1817 for loading creators" → folder name: "IN-1817"

**Priority 2: Slugified Task Name**
If no Jira ticket found:
1. Take first 5-7 words of task description
2. Convert to lowercase
3. Replace spaces with hyphens
4. Remove special characters (keep only a-z, 0-9, hyphens)
5. Remove leading/trailing hyphens
6. Example: "Add load creators button to admin campaign" → "add-load-creators-button"

### Collision Handling

If folder already exists in current directory:
1. Check for folder with same name
2. If exists, append "-2", "-3", etc. until unique name found
3. Example: "add-load-creators-button-2", "add-load-creators-button-3"

### Content Generation

- Use git to detect project name from remote URL
- Reference root AGENTS.md if exists for project conventions
- Generate 3-5 user stories per task for adequate breakdown
- Follow acceptance criteria format: verb + specific outcome
- Order stories logically: UI first, then backend, then integration
- AGENTS.md should be task-specific (not project-level guidelines)
- plans.md should be high-level outline (not integrated with plan execution system)

### Calling the-architect

After Phase 4, launch the-architect subagent to review the generated plan:
- Use Task tool with subagent_type: `the-architect`
- description: "Review plan architecture"
- Prompt should include task folder name and request to analyze scalability

---

## Example Output

### Command

```
/spec-planner add load creators button to admin campaign
```

### Created Folder Structure

```
add-load-creators-button/
├── AGENTS.md
├── plans.md
└── prd.json
```

### AGENTS.md

```markdown
# Task: Add Load Creators Button to Admin Campaign

## Description
Add a "Load Creators" button in the admin campaign section to trigger a search for creators matching campaign eligibility criteria.

## Context
- Project: infloso
- Ticket: TASK-001
- Branch: feature/TASK-001-add-load-creators-button

## Acceptance Criteria
- [ ] Button visible only to admin users with campaign manager permissions
- [ ] Button positioned below Creator Notifications section
- [ ] Clicking button triggers creator search based on campaign criteria
- [ ] Results displayed in modal for review
- [ ] Admin can select and add creators to campaign

## Technical Requirements
- Reuse existing Button component with primary variant
- Integrate with existing deepsearch functionality
- Follow existing campaign management patterns

## Dependencies
- [ ] Campaign management permissions system
- [ ] Deepsearch API integration
- [ ] Creator selection modal component

## Status
Not Started
```

### plans.md

```markdown
# Plan: Add Load Creators Button to Admin Campaign

## Overview
Add a button to the admin campaign page that allows campaign managers to search for and load creators matching the campaign's eligibility criteria. The implementation will reuse existing UI components and backend services.

## Prerequisites
- [ ] Review existing CampaignInfo component structure
- [ ] Understand current creator eligibility criteria system
- [ ] Check existing deepsearch integration

## Implementation Steps

### Step 1: Add Button to CampaignInfo Component
**Priority:** 1
**Estimated Time:** 30 minutes

**Actions:**
1. Import Button component in CampaignInfo.tsx
2. Add "Load Creators" button below CampaignPushNotification
3. Add visibility check for campaign manager permissions

**Files to Modify:**
- `src/components/campaign/CampaignInfo.tsx`

**Details:**
- Button should only show when view='admin' and user has campaign manager permissions
- Use existing Button component with primary variant
- Position after Creator Notifications section

**Verification:**
- [ ] Button visible to campaign managers only
- [ ] Button uses correct styling
- [ ] Typecheck passes

---

### Step 2: Create API Service Function
**Priority:** 2
**Estimated Time:** 45 minutes

**Actions:**
1. Add loadCreatorsFromCampaign() function to admin.ts service
2. Function accepts campaignId parameter
3. Use existing ifetch utility for API call

**Files to Modify:**
- `src/services/admin.ts`

**Details:**
- POST request to /api/v1/admin/campaign/load-creators/:campaignId
- Handle loading states and errors appropriately

**Verification:**
- [ ] Function defined with correct signature
- [ ] API endpoint called correctly
- [ ] Error handling implemented

## Testing Strategy
- [ ] Test button visibility with different user roles
- [ ] Test with various campaign eligibility criteria
- [ ] Test country filtering in both database and deepsearch
- [ ] Verify creators can be added to campaign successfully

## Risk Mitigation
- API endpoint not available yet → Create backend endpoint first
- Deepsearch results too slow → Add loading state and timeout
- Duplicate creators in results → Implement deduplication logic

## Completion Criteria
- [ ] All acceptance criteria met
- [ ] Typecheck passes for both projects
- [ ] Build completes without errors
- [ ] Manual browser testing completed
```

### prd.json

```json
{
  "project": "infloso",
  "ticket": "TASK-001",
  "branchName": "feature/TASK-001-add-load-creators-button",
  "description": "Add a button in admin campaign section to load creators based on eligibility criteria using deepsearch",
  "userStories": [
    {
      "id": "US-001",
      "title": "Add Load Creators button to admin campaign page",
      "description": "As an admin user, I want to see a 'Load Creators' button in the campaign page so I can trigger a search for creators matching the campaign criteria.",
      "acceptanceCriteria": [
        "Add 'Load Creators' button in CampaignInfo component below CampaignPushNotification",
        "Button only visible when view='admin' and user has campaign manager permissions",
        "Button positioned after Creator Notifications section",
        "Button uses existing Button component with primary variant",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-002",
      "title": "Create API service function for load creators",
      "description": "As a developer, I need a frontend service function to call the backend API for loading creators based on campaign eligibility.",
      "acceptanceCriteria": [
        "Add loadCreatorsFromCampaign() function to admin.ts service following existing patterns",
        "Function accepts campaignId as parameter",
        "Function makes POST request to /api/v1/admin/campaign/load-creators/:campaignId",
        "Function uses existing ifetch utility",
        "Function handles loading states and errors appropriately"
      ],
      "priority": 2,
      "passes": false,
      "notes": ""
    }
  ]
}
```
