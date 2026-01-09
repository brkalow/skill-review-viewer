# /review - Claude Code Skill

A Claude Code skill that generates visual code reviews with inline comments using [@pierre/diffs](https://diffs.com/).

## Features

- Review git branch diffs or uncommitted changes
- AI-generated review comments on specific lines
- Self-contained HTML output (no build step)
- Dark theme matching diffs.com aesthetic
- Split or unified diff views
- Syntax highlighting via Shiki

## Installation

Copy the skill to your Claude Code skills directory:

```bash
# Clone the repo
git clone https://github.com/brkalow/skill-review-viewer.git

# Copy to skills directory
mkdir -p ~/.claude/skills/review
cp -r skill-review-viewer/SKILL.md skill-review-viewer/references ~/.claude/skills/review/
```

Or create a symlink:

```bash
ln -s /path/to/skill-review-viewer ~/.claude/skills/review
```

## Usage

In any git repository with changes:

```
/review
```

Claude will:
1. Gather diffs from your branch or working directory
2. Analyze the code and generate review comments
3. Create `./code-review.html` with visual diffs and inline comments

Open the generated file:
```bash
open ./code-review.html
```

## Output

The generated HTML file:
- Loads `@pierre/diffs` from jsdelivr CDN
- Displays diffs with syntax highlighting
- Shows review comments inline at the relevant lines
- Works offline after initial load (CDN assets are cached)

## Customization

### Change Source

By default, reviews the current branch diff. You can ask for:
- "review uncommitted changes"
- "review staged changes"
- "review changes since main"

### Output Location

By default writes to `./code-review.html`. Specify a different path:
- "review and save to ~/reviews/my-review.html"

## Dependencies

- Claude Code CLI
- Git
- Modern browser (for viewing output)

No npm install or build step required - the HTML loads dependencies from CDN.

## License

MIT
