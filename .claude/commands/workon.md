---
description: Fetch and plan work on a GitHub or Linear task
allowed-tools: Bash(gh:*), Read, Write, Task
argument-hint: <github|linear> <task-number>
model: sonnet
---

You are starting work on a task. Parse the arguments to determine the source and task ID.

Arguments: $ARGUMENTS

## Step 1: Fetch Task Details

**If source is "github":**
Execute: `gh issue view $2 --json number,title,body,labels,assignees,url,state`

**If source is "linear":**
Use the Linear MCP server to fetch issue details for task ID $2.

**If source is invalid or missing:**
Show error: "Usage: /workon <github|linear> <task-number>"

## Step 2: Display Task Summary

Show a clear summary of the task:
- **Source**: GitHub Issue #X or Linear Task Y
- **Title**: [title]
- **Status**: [state]
- **Labels**: [labels]
- **URL**: [url]
- **Description**: [body/description summary]

## Step 3: Invoke Task Planner

Use the Task tool to invoke the `task-planner` subagent with this prompt:

```
Analyze and create an implementation plan for this task:

**Source**: [GitHub/Linear] #[number]
**Title**: [title]
**URL**: [url]

**Full Description**:
[complete body/description]

**Labels**: [labels]
**Acceptance Criteria**: [extract from description if present]

Create a detailed IMPLEMENTATION_PLAN.md following the standard format.
```

## Step 4: Wait for User Direction

After the task-planner completes, ask the user:
- "The implementation plan has been created. Would you like to review it?"
- "Which stage should we start with, or would you like to work through them in order?"

Do NOT start implementation automatically - wait for explicit user direction.
