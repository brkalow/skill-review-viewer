---
name: review
description: Generate a visual code review with inline comments. Use when user invokes "/review", asks to "review my changes", "generate code review", or wants a diff with review comments.
---

# Code Review Skill

Generate an HTML diff viewer with review comments using `@pierre/diffs`.

## Workflow

1. **Get the diff** - Uncommitted changes by default, or branch diff if clean
2. **Gather git info** - Run git commands below
3. **Run 3 parallel reviews** - Launch subagents with different focus areas (see below)
4. **Aggregate results** - Deduplicate findings, rank by severity
5. **Generate HTML** - Use template from [references/html-template.md](references/html-template.md)
6. **Open in browser** - Write to `/tmp/code-review-{timestamp}.html` and open it

## Git Commands

```bash
# Detect default branch
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo 'main')

# Get diff (uncommitted or branch)
git diff HEAD                         # uncommitted changes
git diff "${DEFAULT_BRANCH}...HEAD"  # branch diff

# Repository info
REPO=$(git remote get-url origin 2>/dev/null | sed 's/.*github.com[:/]\(.*\)\.git/\1/')
gh pr view --json url --jq '.url' 2>/dev/null  # PR URL if available
```

## Parallel Review Strategy

Launch 3 subagents in parallel, each with a different focus:

1. **Defects** - Logic errors, boundary conditions, null/undefined handling, missing validation, error handling gaps, edge cases
2. **Security** - Injection risks, authentication/authorization issues, exposed secrets, unsafe operations
3. **Architecture** - Pattern violations, unnecessary complexity, performance issues (N+1 queries, quadratic algorithms on unbounded data)

Each subagent reviews the diff and returns findings with file path, line number, severity, and message.

Aggregate by: deduplicating similar findings, ranking by severity (high/medium/low), keeping only issues with realistic impact.

## Review Standards

- **Be certain** - Don't speculate about bugs; verify before flagging
- **Be realistic** - Only raise edge cases with plausible scenarios
- **Stay focused** - Only review modified code, not pre-existing issues
- **Skip style** - No nitpicks on formatting or preferences
- **Be direct** - Factual tone, specific file/line references, actionable suggestions

## Embedding Patches

Diff content contains backticks and `${...}` that break JavaScript template literals. Use `JSON.stringify()` to safely embed patches:

```javascript
files: [
  { path: "src/file.ts", patch: ${JSON.stringify(diffString)}, annotations: [] }
]
```

## Line Number Mapping

Diff hunk headers: `@@ -10,6 +12,8 @@`
- `-10,6` = old file line 10, 6 lines
- `+12,8` = new file line 12, 8 lines

For annotations:
- `+` lines: use `side: "additions"` with new file line number
- `-` lines: use `side: "deletions"` with old file line number
