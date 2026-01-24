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

#### `/pp` - Context Reminder & Quick Reference
Show current session context and critical rules

**Usage:** `/pp`

**Use this when:**
- Starting to work after a break
- About to make a change and unsure of impact
- Need a quick reminder of the rules
- Session context seems lost

#### `/pp-diag` - Diagnose Issues
Run comprehensive diagnostics on portal platform setup

**Usage:** `/pp-diag` or `/pp-diag session` or `/pp-diag tenant drc`

**What it checks:**
- Session configuration
- Portal platform repository
- Tenant repository structure
- Tenant folder in platform
- File extensions (.tsx vs .ts)
- Missing tenant_id filters (security!)
- Files in wrong locations
- Git status and uncommitted changes
- Environment variables

**Use this when:**
- Setup isn't working
- Pages not rendering
- Unclear why something failed
- Want to validate everything is correct
- Checking for security issues (missing filters)

#### `/pp-setup` - Setup Work Session
Setup portal-platform and tenant repo for this work session

**Usage:** `/pp-setup`

**What it does:**
- Asks what you're working on
- Clones portal-platform (shared codebase)
- Creates/updates tenant-specific Git repo
- Initializes folder structure: migrations/, docs/, config/
- Stores session info for other commands

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
- **Saves migration to tenant Git repo**
- **Generates documentation (decisions.md, features.md)**
- **Creates tenant.json config**

#### `/pp-page <slug> <page-name>` - Create Tenant Page
Create a new page for a specific tenant

**Usage:** `/pp-page fitzone home`

**What it does:**
- Creates page at app/(tenants)/<slug>/<page>/page.tsx
- Includes tenant isolation (tenant_id filtering)
- Uses shared components
- Follows tenant's design system

#### `/pp-component <name>` - Create Shared Component
Create a GENERIC framework component (use sparingly!)

**Usage:** `/pp-component LoadingSpinner`

**What it does:**
- Creates component at components/shared/<name>.tsx
- Reusable across all tenants

**⚠️ ONLY USE FOR:**
- Generic utilities (LoadingSpinner, Modal, Button)
- Framework components (FormField, Card)

**❌ DON'T USE FOR:**
- Hero sections (make unique per tenant!)
- Pricing tables (different per tenant!)
- Content components (create in tenant pages instead!)

#### `/pp-list` - List All Tenants
Show all tenants in database

**Usage:** `/pp-list`

**What it does:**
- Runs script from portal-platform
- Shows all tenants with IDs, slugs, domains

#### `/pp-triage` - Triage All Tenant Issues
Scan all tenant repositories and create prioritized work queue

**Usage:** `/pp-triage` or `/pp-triage drc` or `/pp-triage --priority`

**What it does:**
- Discovers all tenant repositories
- Fetches open issues from GitHub for each tenant
- Categorizes by priority (HIGH, MEDIUM, LOW)
- Categorizes by type (Bug, Feature, Security, Docs)
- Flags security issues 🚨
- Shows tenant health summary
- Suggests next actions
- Exports reports (CSV + Markdown)

**Output:**
- Prioritized work queue ready for `/pp-issue`
- Security alerts for immediate attention
- Platform-wide issue detection
- Per-tenant health status

**Use this for:**
- Monday morning planning
- Sprint planning
- Daily standups
- Client reviews

#### `/pp-issue <slug> <issue-number>` - Work on Tenant Issue
Work on GitHub issue for a specific tenant with full workflow

**Usage:** `/pp-issue fitzone 42`

**What it does:**
- Fetches issue from tenant's GitHub repo (beyond-spreadsheets/tenant-{slug})
- Creates isolated work folder
- Clones portal-platform and tenant repo
- Guides through investigation and implementation
- Saves migrations to tenant repo
- Commits code changes to platform
- Creates PRs for both repos
- Updates issue with progress

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
