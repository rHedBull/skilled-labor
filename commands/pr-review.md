---
name: pr-review
description: Review a GitHub pull request for bugs, security issues, and code quality
---

Review GitHub pull requests with multi-agent analysis and confidence-based filtering.

## Prerequisites

- `gh` CLI installed and authenticated (`gh auth status`)
- Inside a git repository with a GitHub remote

## Usage

The user provides a PR number or URL. If none given, review the most recent PR.

## Review Process

### Step 1: Fetch PR Info

```bash
# Get PR details
gh pr view <NUMBER> --json number,title,author,state,isDraft,body,baseRefName,headRefName,url,additions,deletions,changedFiles

# Get the diff
gh pr diff <NUMBER>

# Get list of changed files
gh pr diff <NUMBER> --name-only
```

If the PR is closed, merged, or a draft, inform the user and ask if they still want to review.

### Step 2: Gather Project Context

Check for project guidelines that inform the review:

```bash
find . -maxdepth 3 -name "CLAUDE.md" -o -name ".cursorrules" -o -name "CONVENTIONS.md" -o -name "CODING_STANDARDS.md" 2>/dev/null | head -20
```

Read any found convention files.

### Step 3: Parallel Review with Subagents

Launch 5 parallel review agents. Each gets the PR diff, changed file list, and project conventions.

**Agent 1 — Convention Compliance**: Check changes against project conventions. Flag violations of explicit project rules.

**Agent 2 — Bug Scan (Shallow)**: Read only the diff. Look for obvious bugs: logic errors, null handling, off-by-one, missing error handling, type mismatches.

**Agent 3 — Historical Context**: Run `git log --oneline -10 -- <file>` and `git blame -L <changed_lines> <file>` for modified files.

**Agent 4 — Related PR Context**: Run `gh pr list --state merged --search "<changed_files>" --limit 5` to find recent PRs touching the same files.

**Agent 5 — Code Comment Compliance**: Read full modified files. Check if changes violate guidance in code comments.

Each agent must return:
```
## Issues Found
- **File**: path/to/file.ts
- **Line(s)**: 42-45
- **Description**: What's wrong
- **Reason**: Why flagged
- **Confidence**: 0-100
```

### Step 4: Confidence Scoring

- **0**: False positive.
- **25**: Might be real. Stylistic issues not in conventions.
- **50**: Real but minor. Nitpick.
- **75**: Very likely real. Verified, will impact functionality.
- **100**: Certain. Confirmed real, evidence directly confirms it.

**Filter OUT**: Pre-existing issues, things linters catch, general quality concerns not in conventions, issues on unmodified lines.

### Step 5: Filter and Format

**Only report issues with confidence >= 80.**

Get the full commit SHA and repo name:
```bash
gh pr view <NUMBER> --json headRefOid --jq '.headRefOid'
gh repo view --json nameWithOwner --jq '.nameWithOwner'
```

Format:
```markdown
### Code Review — PR #<number>: <title>

**<N> issue(s) found** (filtered from <total> candidates, confidence >= 80):

1. **<brief description>** (confidence: <score>)
   Reason: <explanation>
   https://github.com/<owner>/<repo>/blob/<full_sha>/<file>#L<start>-L<end>
```

### Step 6: Post Review (Optional)

Ask the user if they want to post:

```bash
gh pr review <NUMBER> --comment --body "<review_markdown>"
```
