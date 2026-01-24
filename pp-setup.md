---
description: Setup Portal Platform session for specific tenant
autoApprove:
  - Bash(git clone*)
  - Bash(git pull*)
  - Bash(pwd)
  - Bash(ls*)
  - Bash(test*)
  - Bash(cat*)
  - Bash(mkdir*)
  - Bash(git init*)
  - Bash(git add*)
  - Bash(git commit*)
---

Setup Portal Platform work session for a specific tenant.

## Command Format

`/pp-setup <slug>`

Examples:
- `/pp-setup ayup` - Setup for AYUP tenant
- `/pp-setup bs` - Setup for Beyond Spreadsheets
- `/pp-setup drc` - Setup for Derby Rugby Club

**Special case:**
- `/pp-setup platform` - Setup for platform-wide changes (no specific tenant)

## Step 1: Parse Argument

```bash
TENANT_SLUG="$1"

if [ -z "$TENANT_SLUG" ]; then
  echo "❌ ERROR: Tenant slug required"
  echo ""
  echo "Usage: /pp-setup <slug>"
  echo ""
  echo "Examples:"
  echo "  /pp-setup ayup      # Work on AYUP"
  echo "  /pp-setup bs        # Work on Beyond Spreadsheets"
  echo "  /pp-setup platform  # Platform-wide changes"
  echo ""
  exit 1
fi

echo "🚀 Setting up session for: $TENANT_SLUG"
echo ""
```

## Step 2: Auto-detect portal-platform

```bash
# Auto-detect portal-platform location
if [ -f "$(pwd)/package.json" ] && grep -q "portal-platform" "$(pwd)/package.json"; then
  PORTAL_PLATFORM="$(pwd)"
  echo "✅ Using portal-platform at: $PORTAL_PLATFORM"
elif [ -d "$HOME/Desktop/portal-platform" ]; then
  PORTAL_PLATFORM="$HOME/Desktop/portal-platform"
  echo "✅ Found portal-platform at: $PORTAL_PLATFORM"
elif [ -d "$HOME/portal-work/portal-platform" ]; then
  PORTAL_PLATFORM="$HOME/portal-work/portal-platform"
  echo "✅ Found portal-platform at: $PORTAL_PLATFORM"
else
  # Clone to standard location
  PORTAL_PLATFORM="$HOME/portal-work/portal-platform"
  echo "📦 Cloning portal-platform..."
  mkdir -p "$HOME/portal-work"
  git clone https://github.com/beyond-spreadsheets/portal-platform.git "$PORTAL_PLATFORM"
  echo "✅ Cloned to: $PORTAL_PLATFORM"
fi

echo ""
```

## Step 3: Setup tenant repository

```bash
if [ "$TENANT_SLUG" = "platform" ]; then
  # Platform-wide work, no tenant repo needed
  TENANT_REPO=""
  echo "✅ Platform-wide session (no tenant repo)"
else
  # Setup tenant-specific repo
  TENANT_REPO="$HOME/portal-work/tenants/$TENANT_SLUG"
  
  if [ -d "$TENANT_REPO" ]; then
    echo "✅ Tenant repo exists: $TENANT_REPO"
    cd "$TENANT_REPO"
    
    # Try to pull latest
    if [ -d ".git" ]; then
      git pull origin main 2>/dev/null && echo "   Updated from GitHub" || echo "   (Local only)"
    fi
  else
    echo "📦 Creating tenant repo: $TENANT_REPO"
    mkdir -p "$TENANT_REPO"
    cd "$TENANT_REPO"
    
    git init
    mkdir -p migrations docs config backups
    
    # Create README
    cat > README.md << EOFREADME
# Tenant: $TENANT_SLUG

Configuration repository for this tenant.

## Structure
- \`migrations/\` - Database migrations
- \`docs/\` - Decision logs and documentation
- \`config/\` - Tenant configuration (tenant.json)
- \`backups/\` - Database backups (not in git)

## Usage
- Migrations saved here by \`/pp-newtenant\` and \`/pp-issue\`
- Documentation auto-generated
- Config tracked for audit trail
EOFREADME

    # Create .gitignore
    cat > .gitignore << EOFGITIGNORE
.env.local
*.key
*.pem
backups/*.sql
backups/*.dump
*.tmp
.DS_Store
EOFGITIGNORE

    git add .
    git commit -m "feat: Initialize tenant repo for $TENANT_SLUG"
    
    echo "✅ Tenant repo created"
  fi
fi

echo ""
```

## Step 4: Save session

```bash
# Save session configuration
cat > ~/.claude-portal-session << EOFSESSION
PORTAL_PLATFORM=$PORTAL_PLATFORM
TENANT_REPO=$TENANT_REPO
TENANT_SLUG=$TENANT_SLUG
EOFSESSION

# Backwards compatibility
echo "$PORTAL_PLATFORM" > ~/.claude-portal-path

echo "═══════════════════════════════════════════"
echo "   SESSION CONFIGURED"
echo "═══════════════════════════════════════════"
echo ""
echo "✅ Portal Platform: $PORTAL_PLATFORM"
if [ -n "$TENANT_REPO" ]; then
  echo "✅ Tenant: $TENANT_SLUG"
  echo "✅ Tenant Repo: $TENANT_REPO"
else
  echo "✅ Mode: Platform-wide changes"
fi
echo ""
echo "Session saved to: ~/.claude-portal-session"
echo ""
```

## Step 5: Show next steps

```bash
if [ "$TENANT_SLUG" = "platform" ]; then
  # Platform-wide work
  echo "Platform-Wide Session"
  echo "━━━━━━━━━━━━━━━━━━━━"
  echo ""
  echo "Changes will affect ALL tenants."
  echo ""
  echo "Next steps:"
  echo "  /pp-component <name>    # Create shared component"
  echo "  /pp-list                # List all tenants"
  echo ""
else
  # Tenant-specific work
  echo "Next Steps for $TENANT_SLUG"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo ""
  
  # Check if tenant exists in database
  echo "Suggested commands:"
  echo ""
  echo "  /pp-issue $TENANT_SLUG <num>  # Work on GitHub issue"
  echo "  /pp-page $TENANT_SLUG <page>   # Create new page"
  echo ""
  echo "If tenant doesn't exist in database yet:"
  echo "  /pp-newtenant \"Tenant Name\"    # Create in database"
  echo ""
fi

echo "═══════════════════════════════════════════"
```

## Summary

This command:
- ✅ Takes one argument: tenant slug
- ✅ Auto-detects portal-platform location
- ✅ Creates/updates tenant repo automatically
- ✅ Saves session for other commands
- ✅ Zero interactive prompts
- ✅ Supports "platform" for platform-wide work

Usage:
```bash
/pp-setup ayup         # Work on AYUP
/pp-setup bs           # Work on Beyond Spreadsheets  
/pp-setup platform     # Platform-wide changes
```

All other `/pp-*` commands check this session and fail with clear message if not configured.
