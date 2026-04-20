---
name: oh-gc-pr-review
description: Review pull requests in OpenHarmony or other GitCode repositories with the oh-gc CLI, inspect diffs and changed files, validate findings against local code, and leave precise inline comments on the relevant lines. Use when Codex is asked to review a PR, summarize review findings, or post line-specific feedback with oh-gc.
---

# OH-GC PR Review

Use this skill to review a PR with `oh-gc`, confirm findings against the local checkout, and leave clean inline comments.

## Workflow

1. Confirm the CLI and repo context.
   Run `which oh-gc` or `command -v oh-gc`.
   Run `git remote -v` and `git status --short`.
   Use the current repo when it already matches the PR's repository.

2. Pull the PR context with `oh-gc`.
   Run `oh-gc pr view <number> --repo <owner/repo>` for title, author, and description.
   Run `oh-gc pr files <number> --repo <owner/repo>` to see scope.
   Run `oh-gc pr diff <number> --repo <owner/repo>` to inspect the patch.
   Run `oh-gc issue view <issue>` or `oh-gc pr commits <number>` when linked issues or commit history matter.

3. Read surrounding code locally before judging the patch.
   Open the touched files in the local checkout with `sed`, `rg`, or `git show`.
   When the PR branch is needed for exact line numbers or post-change file contents, fetch it with:

```bash
git fetch origin refs/merge-requests/<number>/head:pr-<number>
```

   Then inspect with commands like:

```bash
git show pr-<number>:path/to/file | nl -ba | sed -n '120,180p'
```

4. Review for behavior, not just style.
   Prioritize correctness, stale state, broken invariants, missing resets, bad assumptions, and missing tests.
   Compare the patch against nearby design docs, READMEs, or existing state-machine comments when the feature is subtle.
   Check whether new state is initialized, consumed, and cleared.
   Check whether fallbacks are being mistaken for real results.

5. Draft findings with exact file and line targets.
   Map each finding to a changed file and an added line visible in the PR diff.
   Keep comments short, concrete, and actionable.
   Prefer asking a focused question when intent is unclear.

Use this review summary template when reporting the review result:

```markdown
## Review

**Summary:** [Good / Needs changes]

**Issues:**
- [High] file:line — problem → fix
- [Medium] file:line — problem → fix

**Suggestions:**
-

**Questions:**
-

**Verdict:** [Approve / Request changes]
```

When reviewing PRs:
- Be specific: always include file + line if possible.
- Separate blocking issues from suggestions.
- Keep each comment <= 3 lines.
- Prefer actionable fixes over vague criticism.
- Do not repeat obvious code; focus on insight.
- Default to concise bullet points.

## Commenting

Use `oh-gc pr comment` for inline feedback:

```bash
oh-gc pr comment <number> \
  --repo <owner/repo> \
  --path path/to/file.cpp \
  --line 141 \
  --body 'Explain the concrete risk, why it matters, and what behavior to reconsider.'
```

Use plain single-quoted bodies.
Avoid backticks inside shell-quoted comment text unless they are escaped correctly.
If the body needs an apostrophe, use shell-safe quoting or rewrite the sentence to avoid it.

Good comment pattern:
- Start with the observed behavior in this patch.
- Explain the risk or regression.
- Suggest a safer behavior or ask the key design question.

## Verification

After posting comments, verify them:

```bash
oh-gc pr comments <number> \
  --repo <owner/repo> \
  --comment-type diff_comment \
  --latest \
  --limit 10 \
  --full-body
```

If the shell mangled a comment body, delete and repost it:

```bash
oh-gc pr comments <number> --repo <owner/repo> --delete <comment_id>
```

## Heuristics

- Use `rg` for code search and `sed -n` for targeted reads.
- Prefer local source inspection over relying only on the PR diff.
- Fetch the merge-request ref when exact post-change line context matters.
- Keep inline comments tied to one issue each.
- Mention residual uncertainty when a finding depends on behavior outside the visible patch.
- Note if you did not run tests.
