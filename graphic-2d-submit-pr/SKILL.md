---
name: graphic-2d-submit-pr
description: Submit a Pull Request from a local branch to openharmony/graphic_graphic_2d repository.
---

## Context

This workflow submits a PR from the user's fork (`zhoutianer/graphic_graphic_2d`) to the upstream repository (`openharmony/graphic_graphic_2d`).

## Repository Setup

- **Upstream remote**: `gitcode` points to `openharmony/graphic_graphic_2d`
- **Fork remote**: `z` points to `zhoutianer/graphic_graphic_2d` (user's personal fork)

## Workflow Steps

### 1. Push the branch to fork

Push the local branch to the user's fork:

```bash
git push z <branch-name>:<branch-name>
```

Note: Only push to remote `z`, not `gitcode`. The `gitcode` remote is the upstream that receives PRs.

### 2. Create a tracking issue

Use `mcp__gitcode__create_issue` to create an issue for tracking:

- **owner**: `openharmony`
- **repo**: `graphic_graphic_2d`
- **title**: Descriptive title of the change
- **body**: Description of the issue/change

Example:
```json
{
  "owner": "openharmony",
  "repo": "graphic_graphic_2d",
  "title": "Revert color picker task priority change",
  "body": "This issue tracks the reversion of a color picker task priority change..."
}
```

The response will include an `issue_number` (e.g., `22628`).

### 3. Create the Pull Request

Use `mcp__gitcode__create_pull_request` to create the PR:

- **owner**: `openharmony`
- **repo**: `graphic_graphic_2d`
- **title**: PR title (typically matches the commit message)
- **head**: `<username>:<branch-name>` (e.g., `zhoutianer:priority`)
- **base**: `master` (upstream target branch)
- **body**: PR description with issue reference (e.g., `Fixes #<issue_number>`)

Example:
```json
{
  "owner": "openharmony",
  "repo": "graphic_graphic_2d",
  "title": "Revert color picker task priority",
  "head": "zhoutianer:priority",
  "base": "master",
  "body": "This PR reverts a color picker task priority change.\n\nFixes #22628"
}
```

## Verification

After completing the steps:

1. **PR URL**: The response includes `web_url` (e.g., `https://gitcode.com/openharmony/graphic_graphic_2d/merge_requests/28676`)
2. **Issue link**: The issue should be linked in the PR description
3. **Changes**: The PR diff should show the expected code changes

## Common Patterns

- **PR title**: Use imperative mood (e.g., "Revert color picker task priority" not "Reverting color picker task priority")
- **Issue reference**: Use `Fixes #<number>` to automatically link and close the issue when PR merges
- **Base branch**: Typically `master` for this repository

## Available Tools

- `mcp__gitcode__create_issue` - Create new issues
- `mcp__gitcode__create_pull_request` - Create new PRs
- `mcp__gitcode__get_pull_request` - Get PR details
- `mcp__gitcode__get_issue` - Get issue details
- `mcp__gitcode__list_pull_requests` - List repository PRs
- `mcp__gitcode__list_issues` - List repository issues
