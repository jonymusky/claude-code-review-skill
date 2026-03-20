# Claude Code Review Skill

A [Claude Code](https://claude.com/claude-code) skill that performs CodeRabbit-style PR reviews with severity grouping, diff suggestions, and a consolidated AI agent prompt block.

## Features

- Reviews PRs for security, code quality, frontend patterns, and project conventions
- Groups findings by severity (Critical, Medium, Nitpick)
- Includes concrete diff suggestions for each finding
- Single "Prompt for AI Agents" block at the end for easy copy-paste fixing
- Adapts to project-specific guidelines (reads CLAUDE.md)
- Posts review as a single formatted GitHub comment

## Installation

### Option 1: Symlink (recommended for personal use)

```bash
# Clone the repo
git clone https://github.com/jonymusky/claude-code-review-skill.git

# Symlink into your Claude Code skills directory
ln -s /path/to/claude-code-review-skill ~/.claude/skills/pr-review
```

### Option 2: Direct clone into skills

```bash
git clone https://github.com/jonymusky/claude-code-review-skill.git ~/.claude/skills/pr-review
```

## Usage

In Claude Code, run:

```
/pr-review 953
```

Or ask naturally:

```
review PR #953
```

## Output Example

The skill posts a single GitHub comment with:

1. **Critical/High findings** — security vulnerabilities, data exposure
2. **Medium findings** — convention violations, missing validation
3. **Nitpick comments** — style, accessibility, minor improvements
4. **Things done well** — positive patterns found
5. **Summary table** — categories x severities
6. **AI Agent prompt** — single consolidated block to fix all issues

## License

MIT
