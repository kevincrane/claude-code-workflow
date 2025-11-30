---
description: Create a GitHub PR from current branch with auto-filled template
allowed-tools: Bash(git:*), Bash(gh pr:*), Read, Task
model: sonnet
---

You are creating a pull request for completed work.

## Step 1: Check Current State

Execute these commands to understand the current state:

```bash
# Get current branch name
git branch --show-current

# Check git status
git status

# Get recent commits on this branch (vs main)
git log main..HEAD --oneline

# See the diff
git diff main..HEAD --stat
```

Verify:
- We're not on main/master branch
- There are commits to include in the PR
- Changes have been committed (no uncommitted changes)

If uncommitted changes exist, warn user:
```
⚠️  You have uncommitted changes. Please commit them first:
git add .
git commit -m "Your commit message"
```

## Step 2: Optional Code Review

Ask the user:
```
Would you like me to run a code review before creating the PR? (yes/no)
```

If yes, use the Task tool to invoke the `code-reviewer` subagent with:
```
Review the code changes on this branch before PR creation.

Compare: main..HEAD

Provide a review summary with any issues that should be addressed.
```

Wait for review completion. If critical issues found, ask user if they want to:
- Fix issues first (stop here)
- Create PR anyway (continue)

## Step 3: Gather PR Information

Read the IMPLEMENTATION_PLAN.md if it exists to get:
- Original task/issue number and URL
- Summary of what was implemented
- Which stages were completed

Get commit information:
```bash
# Get all commits with details
git log main..HEAD --pretty=format:"%h - %s"
```

## Step 4: Build PR Description

Create a PR description following this template:

```markdown
## What Changed

[Concise summary of changes - 2-3 sentences]

## Why

[Link to original issue/task and brief explanation of motivation]

Closes #[issue-number]
[or] Addresses [Linear task URL]

## Changes Made

[Bullet list derived from commits and IMPLEMENTATION_PLAN.md]
- [Change 1]
- [Change 2]
- [Change 3]

## Testing

[Describe how this was tested]
- [ ] Unit tests added/updated (coverage: X%)
- [ ] Integration tests passing
- [ ] Manual testing completed
- [ ] Edge cases verified

## Implementation Plan Status

[If IMPLEMENTATION_PLAN.md exists, include completion status]

Completed stages:
- [x] Stage 1: [name]
- [x] Stage 2: [name]
- [x] Stage 3: [name]

All acceptance criteria met: ✅

## Deployment Notes

[Optional - only if relevant]
- Database migrations: Yes/No
- Feature flags: Yes/No
- Configuration changes: Yes/No
- Dependencies added: [list]

## Screenshots / Examples

[Optional - for UI changes]

## Rollback Plan

[Optional - for infrastructure/deployment changes]
```

## Step 5: Create the PR

Use gh CLI to create the PR:

```bash
# Push branch if not already pushed
git push -u origin $(git branch --show-current)

# Create PR with the description
gh pr create --title "[TITLE]" --body "[BODY]" --fill
```

For title, use a brief description of the changes, then more details in the body. If we are closing
a Github or Linear issue, preface the title with the issue number so it can be linked back to the
correct issue tracker (e.g. `#123`).

The `--fill` flag will use commit messages and branch info to enhance the PR.

## Step 6: Report Success

After PR creation, show:

```
✅ Pull Request created successfully!

PR #[number]: [title]
URL: [PR URL]

Summary:
- Files changed: X
- Commits: Y
- Tests: [status]

Next steps:
- Review the PR on GitHub
- Address any CI/CD checks
- Wait for reviews (if applicable)
- Merge when ready
```

## Error Handling

### If not on a feature branch
```
❌ Cannot create PR from main/master branch.

Please create a feature branch first:
git checkout -b kcrane/feature-description-slug
```

### If no commits to PR
```
❌ No commits found on this branch compared to main.

Make sure you've committed your changes.
```

### If gh CLI fails
```
❌ Failed to create PR: [error message]

Troubleshooting:
1. Check you're authenticated: gh auth status
2. Verify remote exists: git remote -v
3. Try pushing manually: git push -u origin $(git branch --show-current)
```

## Tips for Good PRs

- **Keep PRs focused** - One feature/fix per PR
- **Write clear titles** - Describe what changed, not how
- **Link to issues** - Use "Closes #123" or "Fixes #456"
- **Include tests** - Show what you tested and how
- **Add context** - Explain why, not just what
- **Screenshots** - For UI changes, include before/after images
- **Consider reviewers** - Make it easy for others to understand

## Remember

- The PR description is for future developers (including yourself)
- Link back to the original task/issue for context
- Include the IMPLEMENTATION_PLAN.md status if applicable
- Make it easy for reviewers to understand what changed and why
- Tests and coverage information help build confidence
