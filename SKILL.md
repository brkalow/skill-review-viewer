---
name: review
description: Generate a visual code review with inline comments. Use when user invokes "/review", asks to "review my changes", "generate code review", or wants a diff with review comments. Creates a self-contained HTML file using @pierre/diffs.
---

# Code Review Skill

Generate a self-contained HTML file displaying code diffs with inline review comments using the `@pierre/diffs` library.

## Workflow

### Step 1: Determine Change Source

Ask the user or infer from context which changes to review:

1. **Branch diff** (default): Changes in current branch vs main/master
2. **Uncommitted**: All uncommitted changes (staged + unstaged)

### Step 2: Gather Diff Data

Run git commands to get the diff:

```bash
# Detect default branch
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo 'main')

# For branch diff
git diff "${DEFAULT_BRANCH}...HEAD"

# For uncommitted changes
git diff HEAD

# List changed files
git diff --name-only "${DEFAULT_BRANCH}...HEAD"  # or HEAD for uncommitted
```

### Step 3: Analyze Code and Generate Comments

For each changed file:

1. Read the full diff output
2. Analyze the changes looking for:
   - Potential bugs or logic errors
   - Security vulnerabilities
   - Performance concerns
   - Code style and readability issues
   - Missing error handling
   - Type safety issues
   - Incomplete implementations
   - Code that could be simplified

3. Generate review comments with:
   - `side`: `"additions"` for new code, `"deletions"` for removed code
   - `lineNumber`: The line number the comment applies to (from the diff hunk header)
   - `metadata.author`: `"Claude"`
   - `metadata.message`: The review comment text

### Step 4: Generate the HTML File

Create a self-contained HTML file using the template in [references/html-template.md](references/html-template.md).

Key points:
- Load `@pierre/diffs` from jsdelivr CDN
- Embed the diff patches as strings in a `reviewData` object
- Include annotations array for each file
- Use dark theme (`pierre-dark`) to match diffs.com aesthetic

### Step 5: Write and Report

1. Write the HTML to `./code-review.html` (or user-specified path)
2. Report the file location and summary (X files, Y comments)
3. Suggest opening with: `open ./code-review.html`

## Review Comment Guidelines

### What to Comment On

- **Bugs**: Logic errors, off-by-one errors, null/undefined issues
- **Security**: Injection vulnerabilities, exposed secrets, unsafe operations
- **Performance**: N+1 queries, unnecessary loops, memory leaks
- **Clarity**: Confusing names, complex expressions, missing context
- **Robustness**: Missing error handling, edge cases, validation

### Comment Format

Keep comments:
- Concise but actionable
- Focused on one issue per comment
- Constructive with suggestions when possible

Example:
```
"Consider using optional chaining here to handle the case where user is undefined"
```

## Line Number Mapping

Git diff hunk headers show line ranges:
```
@@ -10,6 +12,8 @@
```
- `-10,6` = old file starting at line 10, 6 lines of context
- `+12,8` = new file starting at line 12, 8 lines

For annotations:
- Lines starting with `+` are additions - use `side: "additions"` with the new file line number
- Lines starting with `-` are deletions - use `side: "deletions"` with the old file line number
- Context lines (no prefix) exist in both

Count lines within each hunk to determine the exact line number.

## Example Output Structure

The `reviewData` object embedded in HTML:

```javascript
const reviewData = {
  generatedAt: "2024-01-08T12:00:00Z",
  source: "branch", // or "uncommitted"
  baseBranch: "main",
  files: [
    {
      path: "src/utils/auth.ts",
      patch: `--- a/src/utils/auth.ts
+++ b/src/utils/auth.ts
@@ -15,6 +15,10 @@ export function validateToken(token) {
   if (!token) return false;
+  const decoded = jwt.decode(token);
+  if (decoded.exp < Date.now()) {
+    return false;
+  }
   return true;
 }`,
      annotations: [
        {
          side: "additions",
          lineNumber: 17,
          metadata: {
            author: "Claude",
            message: "Date.now() returns milliseconds, but JWT exp is typically in seconds. Consider: decoded.exp * 1000 < Date.now()"
          }
        }
      ]
    }
  ]
};
```
