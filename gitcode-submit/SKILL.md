---
name: gitcode-submit
description: Publish local changes to GitCode using the OpenHarmony `oh-gc` CLI. Use when the user asks to create a GitCode issue, create a pull request or merge request, bind/link an issue to a PR, or perform the full issue + PR + binding submission workflow for the current branch.
---

# GitCode Submit

## Overview

Use this skill to publish a local branch to GitCode with `oh-gc`: confirm the
local scope, make sure the branch is pushed, create an issue, create a draft PR,
and explicitly link the issue to the PR.

`gc-oh` and `oh-gc` may refer to the same intended tool. Prefer `oh-gc` when it
is the installed binary. If neither is available, stop and tell the user.

## Preconditions

1. Run `command -v oh-gc || command -v gc-oh`.
2. Run `oh-gc auth status` and stop if the CLI is not authenticated.
3. Run `git status -sb --untracked-files=all` and inspect the diff.
4. Never stage unrelated changes silently.
5. If the worktree is dirty and scope is unclear, ask before staging.

If the intended changes are already committed and the branch is clean, continue
from the existing commit instead of creating a new one.

## Branch And Remote

1. Identify the current branch:

```sh
git branch --show-current
```

2. Identify remotes and tracking:

```sh
git remote -v
git rev-parse --abbrev-ref --symbolic-full-name @{u}
git rev-list --left-right --count @{u}...HEAD
```

3. Push if the branch is ahead:

```sh
git push -u <fork-remote> "$(git branch --show-current)"
```

For fork-based OpenHarmony submissions, the target repository is commonly
`openharmony/<repo>` and the source branch is commonly `<user>:<branch>`.

## Repository Discovery

Use `oh-gc repo view --json --repo OWNER/REPO` to confirm the target repository
and default branch. OpenHarmony repositories commonly use `master`.

Check target branches before choosing a base:

```sh
git ls-remote --heads origin master main dev
```

If only `master` exists upstream, target `master`. If the local branch is based
on a fork-only branch, mention that in the final summary.

## Create The Issue

Use a concise issue title and a body with background, scope, and validation.

```sh
oh-gc issue create --json --repo OWNER/REPO \
  --title "Issue title" \
  --body "$(cat /tmp/issue-body.md)"
```

Capture the returned issue number and URL.

## Create The Draft PR

Create a draft PR by default unless the user explicitly asks for ready review.
Use a real Markdown body with:

- what changed;
- why it changed;
- impact;
- validation;
- linked issue number.

For a fork branch, pass the fork owner in `--head`:

```sh
oh-gc pr create --json --repo OWNER/REPO \
  --title "[codex] concise change summary" \
  --body "$(cat /tmp/pr-body.md)" \
  --base master \
  --head fork-owner:branch-name \
  --draft
```

Capture the returned PR number and URL. GitCode may prefix draft titles with
`[WIP]`; treat that as expected.

## Bind The Issue

Always run the explicit link command, even if the PR body mentions the issue:

```sh
oh-gc pr link PR_NUMBER ISSUE_NUMBER --json --repo OWNER/REPO
```

Verify the binding:

```sh
oh-gc pr linked-issues PR_NUMBER --json --repo OWNER/REPO
```

## Final Summary

Report:

- issue number and URL;
- PR number and URL;
- branch, tracking remote, and commit SHA;
- whether the branch is clean and in sync;
- validation performed;
- any important caveat, such as targeting `master` because no upstream `dev`
  branch exists.
