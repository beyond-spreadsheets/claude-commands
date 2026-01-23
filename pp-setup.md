---
description: Setup Portal Platform for this work session
autoApprove:
  - Bash(git clone:*)
  - Bash(git pull:*)
---

Setup Portal Platform repository for this Claude Code work session.

## What This Does

Pulls the portal-platform repo to a unique folder for this work session, allowing you to:
- Work on multiple clients simultaneously in different Claude sessions
- Keep work isolated per session
- Have all scripts and tools available

## Instructions

**Step 1: Determine work folder**

Ask the user: "What are you working on today?"

Examples:
- "my own website" → folder: `~/portal-work/beyond-spreadsheets`
- "FitZone Gym client" → folder: `~/portal-work/fitzone-gym`
- "DKC volunteer portal" → folder: `~/portal-work/dkc`
- "testing" → folder: `~/portal-work/test-{timestamp}`

**Step 2: Clone portal-platform (shared codebase)**

```bash
PORTAL_FOLDER="~/portal-work/portal-platform"

# Clone or update the shared platform codebase
if [ -d "$PORTAL_FOLDER" ]; then
  echo "📂 Found existing portal-platform"
  cd "$PORTAL_FOLDER"
  git pull origin main
  echo "✅ Updated to latest version"
else
  echo "📦 Cloning portal-platform"
  git clone https://github.com/beyond-spreadsheets/portal-platform.git "$PORTAL_FOLDER"
  cd "$PORTAL_FOLDER"
  echo "✅ Portal Platform cloned"
fi
```

**Step 3: Create tenant-specific Git repo**

```bash
TENANT_REPO="~/portal-work/tenants/{slug-or-project-name}"

# Create or update tenant repo
if [ -d "$TENANT_REPO" ]; then
  echo "📂 Found existing tenant repo: $TENANT_REPO"
  cd "$TENANT_REPO"
  git pull origin main 2>/dev/null || echo "   (Local repo only)"
else
  echo "📦 Creating tenant repo: $TENANT_REPO"
  mkdir -p "$TENANT_REPO"
  cd "$TENANT_REPO"

  # Initialize Git repo
  git init

  # Create folder structure
  mkdir -p migrations
  mkdir -p docs
  mkdir -p config
  mkdir -p backups

  # Create README
  cat > README.md << 'EOF'
# {Tenant Name} - Configuration Repository

This repository contains all tenant-specific configuration, migrations, and documentation.

## Structure

- `migrations/` - Database migration files
- `docs/` - Decision logs and documentation
- `config/` - Tenant-specific configuration
- `backups/` - Database backups and exports

## Files

- `tenant.json` - Core tenant configuration
- `migrations/*.sql` - Database migrations (timestamped)
- `docs/decisions.md` - Architecture decision records
- `docs/features.md` - Enabled features and configuration

## Git History

This repo tracks all changes to tenant configuration over time.
Each migration, config change, and decision is committed for full audit trail.
EOF

  # Create .gitignore
  cat > .gitignore << 'EOF'
# Environment secrets
.env.local
*.key
*.pem

# Backups (too large for git)
backups/*.sql
backups/*.dump

# Temporary files
*.tmp
.DS_Store
EOF

  # Initial commit
  git add .
  git commit -m "feat: Initialize tenant configuration repo"

  echo "✅ Tenant repo created"
fi
```

**Step 4: Create tenant.json configuration file**

```bash
cd "$TENANT_REPO"

if [ ! -f "config/tenant.json" ]; then
  cat > config/tenant.json << 'EOF'
{
  "name": "{Tenant Name}",
  "slug": "{slug}",
  "created_at": "{timestamp}",
  "platform_path": "~/portal-work/portal-platform",
  "database": {
    "schema": "portal",
    "tenant_table": "portal.tenants"
  },
  "features": [],
  "domains": {
    "subdomain": null,
    "custom_domain": null
  },
  "branding": {
    "primary_color": "#0891b2",
    "secondary_color": "#06b6d4",
    "logo_url": null
  }
}
EOF
  git add config/tenant.json
  git commit -m "feat: Add tenant configuration"
fi
```

**Step 5: Create initial documentation**

```bash
if [ ! -f "docs/decisions.md" ]; then
  cat > docs/decisions.md << 'EOF'
# Architecture Decision Records

## Decision Log

### {Date} - Initial Setup
- Created tenant configuration repository
- Chose portal-platform as shared codebase
- Tenant slug: {slug}

EOF
  git add docs/decisions.md
  git commit -m "docs: Add decision log"
fi

if [ ! -f "docs/features.md" ]; then
  cat > docs/features.md << 'EOF'
# Feature Configuration

## Enabled Features

_(Will be populated as features are enabled)_

## Feature History

EOF
  git add docs/features.md
  git commit -m "docs: Add feature tracking"
fi
```

**Step 6: Install dependencies in portal-platform**

```bash
cd "$PORTAL_FOLDER"

if [ ! -d "node_modules" ]; then
  echo "📦 Installing dependencies..."
  npm install
fi
```

**Step 7: Check environment**

```bash
cd "$PORTAL_FOLDER"

if [ ! -f ".env.local" ]; then
  echo "⚠️  No .env.local found in portal-platform"
  echo "📋 Copy from .env.example:"
  echo "   cp .env.example .env.local"
  echo "   Then add your Supabase credentials"
else
  echo "✅ Environment configured"
fi
```

**Step 8: Store the paths**

Create session file with both paths:

```bash
cat > ~/.claude-portal-session << EOF
PORTAL_PLATFORM=$PORTAL_FOLDER
TENANT_REPO=$TENANT_REPO
TENANT_SLUG={slug}
EOF

# For backwards compatibility
echo "$PORTAL_FOLDER" > ~/.claude-portal-path

echo "✅ Session configured"
```

**Step 9: Summary**

Show the user:

```
═══════════════════════════════════════════
   PORTAL PLATFORM - SESSION SETUP
═══════════════════════════════════════════

✅ Shared Platform: ~/portal-work/portal-platform
✅ Tenant Repo: ~/portal-work/tenants/{slug}

📁 Tenant Repository Structure:
   ~/portal-work/tenants/{slug}/
   ├── migrations/        # Database migrations
   ├── docs/             # Decisions & features
   ├── config/           # tenant.json
   └── backups/          # Database backups

📝 Git initialized in tenant repo
   All changes will be tracked for audit trail

Next steps:
- List tenants: /pp-list
- Create tenant in DB: /pp-tenant "{Tenant Name}"
- Create page: /pp-page {slug} {page-name}

Session files:
- ~/.claude-portal-session (full session config)
- ~/.claude-portal-path (platform path)
```

## How Other Commands Use This

All other PP commands should check for the portal path:

```bash
# In pp-list, pp-tenant, pp-page, etc:
if [ -f ~/.claude-portal-path ]; then
  PORTAL_PATH=$(cat ~/.claude-portal-path)
  cd "$PORTAL_PATH"
  # Now run scripts, create files, etc.
else
  echo "❌ Portal Platform not set up"
  echo "Run: /pp-setup"
  exit 1
fi
```

## Multiple Sessions

Each Claude Code session can have its own portal-platform folder:

**Session 1: Building your website**
```
~/portal-work/beyond-spreadsheets/
```

**Session 2: Working on FitZone client**
```
~/portal-work/fitzone-gym/
```

**Session 3: Debugging DKC portal**
```
~/portal-work/dkc/
```

This prevents conflicts and keeps work organized!

## Expected Output

```
What are you working on today?

User: "my website"

📦 Cloning portal-platform to ~/portal-work/beyond-spreadsheets
Cloning into '~/portal-work/beyond-spreadsheets'...
✅ Portal Platform cloned
📦 Installing dependencies...
added 324 packages in 12s
✅ Environment configured
✅ Portal Platform ready at: ~/portal-work/beyond-spreadsheets

═══════════════════════════════════════════
   PORTAL PLATFORM - SESSION SETUP
═══════════════════════════════════════════

✅ Portal Platform: ~/portal-work/beyond-spreadsheets
✅ Scripts available: scripts/list-all-tenants.mjs
✅ Other commands can now find this installation

Next steps:
- List tenants: /pp-list
- Create tenant: /pp-tenant "Business Name"
- Create page: /pp-page beyond-spreadsheets home

Working directory: ~/portal-work/beyond-spreadsheets
```

## Notes

- The path is stored in `~/.claude-portal-path` (session-specific)
- Each Claude Code instance can have its own path
- Other PP commands read this file to know where to work
- You can manually change it: `echo "/path/to/portal" > ~/.claude-portal-path`
