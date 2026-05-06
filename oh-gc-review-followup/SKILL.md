---
name: oh-gc-review-followup
description: Summarize and execute the GitCode PR review follow-up workflow with oh-gc. Use when Codex needs to inspect GitCode pull request review comments, map unresolved comments to local branch changes, implement or verify fixes, and reply to review discussions with oh-gc pr comment-reply.
---

# oh-gc PR Review Follow-up

Use this workflow for review cycles on a GitCode PR that corresponds to the current local branch.

## Required Pairing

Use this together with `$gitcode-cli` when available. Follow its first-step rule before any other `oh-gc` command:

```bash
oh-gc --version
```

If the installed version does not match the local `$gitcode-cli` skill requirement, update `oh-gc` first or tell the user why you cannot.

## Workflow

1. Confirm context:

```bash
oh-gc auth status
git status --short --branch
git remote -v
oh-gc repo get-remote
oh-gc pr view <pr-number> --json
```

2. Collect review comments:

```bash
oh-gc pr comments <pr-number> --comment-type diff_comment --limit 100 --latest --full-body --json
```

Focus on unresolved human review comments. Treat CI messages, build commands, DCO notices, and stale self-comments as background unless the user explicitly asks about them.

3. Map comments to local code:

```bash
oh-gc pr files <pr-number> --json
git diff --stat
git diff -- <relevant-files>
```

For each actionable thread, identify the `id`, `discussion_id`, file, line, reviewer request, and intended response.

4. Implement follow-up fixes when needed.

Respect the repository's AGENTS.md and existing worktree changes. Keep user-preferred choices when the user clarifies them, even if a reviewer suggested a different cleanup. For example, retaining a local null check can be correct when it improves readability and protects against future utility changes.

5. Verify locally before replying.

Prefer narrow checks for touched code:

```bash
git -c core.whitespace=blank-at-eol,blank-at-eof,space-before-tab,cr-at-eol diff --check
```

For OpenHarmony graphic_2d targets, compile narrow generated Ninja objects or relevant `hb build` targets when practical. If a broader build fails in unrelated code before reaching the touched files, report that clearly.

6. Reply to each review discussion:

```bash
oh-gc pr comment-reply <pr-number> --comment-id <comment-id> --body "<reply>"
```

or:

```bash
oh-gc pr comment-reply <pr-number> --discussion-id <discussion-id> --body "<reply>"
```

Write concise replies in the language used by the review thread when possible. State what changed, what was intentionally kept, or why a point is already covered.

7. Confirm replies landed:

```bash
oh-gc pr comments <pr-number> --comment-type diff_comment --limit 100 --latest --full-body --json
```

Check that each target thread now contains your reply.

## Reply Style

Use concrete, review-friendly wording:

- `已调整：...`
- `已处理：...`
- `这里考虑后保留...，原因是...`
- `已验证：...`

Avoid claiming a fix was pushed unless it was actually committed and pushed. If changes are only local, say so in the final user summary.

## Final Summary

Report:

- Which review threads were replied to, preferably by comment or reply id.
- Which files changed locally.
- Which checks passed.
- Any verification caveats, especially unrelated build failures.
