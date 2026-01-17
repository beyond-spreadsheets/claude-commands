# Claude Code Custom Slash Commands

This repository contains custom slash commands for Claude Code.

## Installation

To use these commands, copy the `.md` files to your `.claude/commands` directory:

```bash
cp *.md ~/.claude/commands/
```

On Windows:
```bash
copy *.md %USERPROFILE%\.claude\commands\
```

## Available Commands

### `/issue` - GitHub Issue Workflow
Work on GitHub issue with full workflow (e.g., `/issue dkc400`)

This command:
- Fetches issue details from GitHub
- Asks 5 clarifying questions to help define the issue
- Creates a todo list for tracking progress
- Creates a new folder and branch for the work
- Guides you through investigation, implementation, testing, and PR creation

**Usage:** `/issue dkc400` (where "dkc" maps to beyond-spreadsheets/dkc_volunteer_portal and "400" is the issue number)

### `/triage` - Triage GitHub Issues
Triage GitHub issues for DKC volunteer portal

This command helps you triage issues in the beyond-spreadsheets/dkc_volunteer_portal repository.

**Usage:** `/triage`

## Repository Mappings

Current repository shortcuts:
- `dkc` → beyond-spreadsheets/dkc_volunteer_portal

## Customization

Feel free to modify these commands or add your own! Each command file should include:

1. YAML frontmatter with a `description` field
2. The prompt that Claude Code will execute

Example:
```markdown
---
description: Your command description here
---

Your prompt text here
```
