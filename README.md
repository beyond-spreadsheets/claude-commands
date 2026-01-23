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

### GitHub Workflow Commands

#### `/issue` - GitHub Issue Workflow
Work on GitHub issue with full workflow (e.g., `/issue dkc400`)

This command:
- Fetches issue details from GitHub
- Asks 5 clarifying questions to help define the issue
- Creates a todo list for tracking progress
- Creates a new folder and branch for the work
- Guides you through investigation, implementation, testing, and PR creation

**Usage:** `/issue dkc400` (where "dkc" maps to beyond-spreadsheets/dkc_volunteer_portal and "400" is the issue number)

#### `/triage` - Triage GitHub Issues
Triage GitHub issues for DKC volunteer portal

This command helps you triage issues in the beyond-spreadsheets/dkc_volunteer_portal repository.

**Usage:** `/triage`

### Portal Platform (Multi-Tenant SaaS) Commands

#### `/pp-help` - Portal Platform Help
Show complete help guide for multi-tenant SaaS commands

**Usage:** `/pp-help`

#### `/pp-tenant "<name>"` - Create New Tenant
Create a new tenant/client with database setup and folder structure

**Usage:** `/pp-tenant "FitZone Gym"`

**What it does:**
- Creates tenant in database (portal.tenants)
- Sets up tenant config with branding
- Creates folder: app/(tenants)/<slug>/
- Enables feature flags
- Creates admin role

#### `/pp-page <slug> <page-name>` - Create Tenant Page
Create a new page for a specific tenant

**Usage:** `/pp-page fitzone home`

**What it does:**
- Creates page at app/(tenants)/<slug>/<page>/page.tsx
- Includes tenant isolation (tenant_id filtering)
- Uses shared components
- Follows tenant's design system

#### `/pp-component <name>` - Create Shared Component
Create a reusable component that all tenants can use

**Usage:** `/pp-component HeroSection`

**What it does:**
- Creates component at components/shared/<name>.tsx
- Tenant-agnostic but tenant-aware (via props)
- Reusable across all tenants

#### `/pp-list` - List All Tenants & Pages
Show complete overview of all tenants, pages, and components

**Usage:** `/pp-list`

**What it shows:**
- All tenants in database
- Each tenant's custom pages
- Shared components
- Feature flags per tenant
- Platform statistics

**See:** [PP-COMMANDS-README.md](PP-COMMANDS-README.md) for complete documentation

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
