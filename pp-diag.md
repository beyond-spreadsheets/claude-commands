---
description: Diagnose Portal Platform issues and validate setup
autoApprove:
  - Bash(cat *)
  - Bash(ls *)
  - Bash(test *)
  - Bash(find *)
  - Read
  - Glob
---

You are diagnosing issues with the Portal Platform multi-tenant SaaS setup.

## Command Format

`/pp-diag [optional: specific-check]`

Examples:
- `/pp-diag` - Run all diagnostics
- `/pp-diag session` - Check session only
- `/pp-diag tenant drc` - Check specific tenant
- `/pp-diag files` - Check file structure
- `/pp-diag git` - Check git status

## Diagnostic Steps

### Step 1: Check Session Context

```bash
echo "═══════════════════════════════════════════"
echo "   SESSION DIAGNOSTICS"
echo "═══════════════════════════════════════════"
echo ""

# Check session file exists
if [ -f ~/.claude-portal-session ]; then
  echo "✅ Session file exists"
  echo ""
  cat ~/.claude-portal-session
  echo ""

  # Load session variables
  source ~/.claude-portal-session

  echo "Session Variables:"
  echo "  PORTAL_PLATFORM: $PORTAL_PLATFORM"
  echo "  TENANT_REPO: $TENANT_REPO"
  echo "  TENANT_SLUG: $TENANT_SLUG"
  echo ""
else
  echo "❌ No session file found"
  echo "   Run: /pp-setup to create session"
  echo ""
fi

# Check legacy portal path
if [ -f ~/.claude-portal-path ]; then
  echo "✅ Legacy portal path exists"
  cat ~/.claude-portal-path
  echo ""
else
  echo "⚠️  No legacy portal path (this is okay if session file exists)"
  echo ""
fi
```

### Step 2: Check Portal Platform Repository

```bash
echo "═══════════════════════════════════════════"
echo "   PORTAL PLATFORM DIAGNOSTICS"
echo "═══════════════════════════════════════════"
echo ""

if [ -f ~/.claude-portal-session ]; then
  source ~/.claude-portal-session
  PORTAL_PATH="$PORTAL_PLATFORM"
else
  PORTAL_PATH="$HOME/portal-work/portal-platform"
fi

if [ -d "$PORTAL_PATH" ]; then
  echo "✅ Portal platform directory exists: $PORTAL_PATH"
  echo ""

  # Check if it's a git repo
  if [ -d "$PORTAL_PATH/.git" ]; then
    echo "✅ Git repository initialized"
    cd "$PORTAL_PATH"
    echo "   Branch: $(git branch --show-current)"
    echo "   Remote: $(git remote get-url origin 2>/dev/null || echo 'No remote')"
    echo ""
  else
    echo "❌ Not a git repository"
    echo ""
  fi

  # Check node_modules
  if [ -d "$PORTAL_PATH/node_modules" ]; then
    echo "✅ Dependencies installed (node_modules exists)"
  else
    echo "⚠️  Dependencies not installed"
    echo "   Run: cd $PORTAL_PATH && npm install"
  fi
  echo ""

  # Check .env.local
  if [ -f "$PORTAL_PATH/.env.local" ]; then
    echo "✅ Environment file exists (.env.local)"

    # Check for required variables (without showing values)
    echo "   Checking required variables:"
    grep -q "NEXT_PUBLIC_SUPABASE_URL" "$PORTAL_PATH/.env.local" && echo "   ✅ NEXT_PUBLIC_SUPABASE_URL" || echo "   ❌ Missing NEXT_PUBLIC_SUPABASE_URL"
    grep -q "NEXT_PUBLIC_SUPABASE_ANON_KEY" "$PORTAL_PATH/.env.local" && echo "   ✅ NEXT_PUBLIC_SUPABASE_ANON_KEY" || echo "   ❌ Missing NEXT_PUBLIC_SUPABASE_ANON_KEY"
    grep -q "SUPABASE_SERVICE_ROLE_KEY" "$PORTAL_PATH/.env.local" && echo "   ✅ SUPABASE_SERVICE_ROLE_KEY" || echo "   ❌ Missing SUPABASE_SERVICE_ROLE_KEY"
  else
    echo "❌ No .env.local file"
    echo "   Run: cd $PORTAL_PATH && cp .env.example .env.local"
    echo "   Then add your Supabase credentials"
  fi
  echo ""

  # Check key directories
  echo "Key Directories:"
  [ -d "$PORTAL_PATH/app/(tenants)" ] && echo "   ✅ app/(tenants)/" || echo "   ❌ app/(tenants)/ missing"
  [ -d "$PORTAL_PATH/lib" ] && echo "   ✅ lib/" || echo "   ❌ lib/ missing"
  [ -d "$PORTAL_PATH/components" ] && echo "   ✅ components/" || echo "   ⚠️  components/ missing"
  [ -d "$PORTAL_PATH/components/shared" ] && echo "   ✅ components/shared/" || echo "   ⚠️  components/shared/ missing"
  echo ""

else
  echo "❌ Portal platform not found at: $PORTAL_PATH"
  echo "   Run: /pp-setup to clone repository"
  echo ""
fi
```

### Step 3: Check Tenant Repository

```bash
echo "═══════════════════════════════════════════"
echo "   TENANT REPOSITORY DIAGNOSTICS"
echo "═══════════════════════════════════════════"
echo ""

if [ -f ~/.claude-portal-session ]; then
  source ~/.claude-portal-session

  if [ -d "$TENANT_REPO" ]; then
    echo "✅ Tenant repository exists: $TENANT_REPO"
    echo "   Tenant: $TENANT_SLUG"
    echo ""

    cd "$TENANT_REPO"

    # Check structure
    echo "Repository Structure:"
    [ -d "migrations" ] && echo "   ✅ migrations/" || echo "   ❌ migrations/ missing"
    [ -d "docs" ] && echo "   ✅ docs/" || echo "   ❌ docs/ missing"
    [ -d "config" ] && echo "   ✅ config/" || echo "   ❌ config/ missing"
    [ -d "backups" ] && echo "   ✅ backups/" || echo "   ⚠️  backups/ missing (optional)"
    echo ""

    # Check key files
    echo "Key Files:"
    [ -f "README.md" ] && echo "   ✅ README.md" || echo "   ⚠️  README.md missing"
    [ -f ".gitignore" ] && echo "   ✅ .gitignore" || echo "   ❌ .gitignore missing"
    [ -f "config/tenant.json" ] && echo "   ✅ config/tenant.json" || echo "   ❌ config/tenant.json missing"
    [ -f "docs/decisions.md" ] && echo "   ✅ docs/decisions.md" || echo "   ⚠️  docs/decisions.md missing"
    [ -f "docs/features.md" ] && echo "   ✅ docs/features.md" || echo "   ⚠️  docs/features.md missing"
    echo ""

    # Count migrations
    MIGRATION_COUNT=$(find migrations -name "*.sql" 2>/dev/null | wc -l | tr -d ' ')
    echo "Migrations: $MIGRATION_COUNT SQL file(s)"
    if [ "$MIGRATION_COUNT" -gt 0 ]; then
      echo "   Latest migrations:"
      find migrations -name "*.sql" | sort -r | head -3 | sed 's/^/   - /'
    fi
    echo ""

    # Git status
    echo "Git Status:"
    if [ -d ".git" ]; then
      echo "   ✅ Git initialized"
      echo "   Branch: $(git branch --show-current)"
      echo "   Commits: $(git rev-list --count HEAD 2>/dev/null || echo '0')"

      REMOTE=$(git remote get-url origin 2>/dev/null)
      if [ -n "$REMOTE" ]; then
        echo "   ✅ Remote: $REMOTE"
      else
        echo "   ⚠️  No remote configured"
        echo "      Suggest: gh repo create beyond-spreadsheets/tenant-$TENANT_SLUG --private"
      fi

      # Check for uncommitted changes
      if [ -n "$(git status --porcelain)" ]; then
        echo "   ⚠️  Uncommitted changes:"
        git status --short | sed 's/^/      /'
      else
        echo "   ✅ No uncommitted changes"
      fi
    else
      echo "   ❌ Not a git repository"
    fi
    echo ""

  else
    echo "❌ Tenant repository not found: $TENANT_REPO"
    echo "   Run: /pp-setup to create tenant repository"
    echo ""
  fi
else
  echo "⚠️  No session active - cannot check tenant repository"
  echo "   Run: /pp-setup first"
  echo ""
fi
```

### Step 4: Check Tenant Folder in Platform

```bash
echo "═══════════════════════════════════════════"
echo "   TENANT FOLDER DIAGNOSTICS"
echo "═══════════════════════════════════════════"
echo ""

if [ -f ~/.claude-portal-session ]; then
  source ~/.claude-portal-session

  TENANT_FOLDER="$PORTAL_PLATFORM/app/(tenants)/$TENANT_SLUG"

  if [ -d "$TENANT_FOLDER" ]; then
    echo "✅ Tenant folder exists: $TENANT_FOLDER"
    echo ""

    cd "$TENANT_FOLDER"

    # Check for layout
    if [ -f "layout.tsx" ]; then
      echo "   ✅ layout.tsx exists"

      # Check if layout has tenant detection
      if grep -q "getTenantFromHostname" layout.tsx; then
        echo "      ✅ Has tenant detection"
      else
        echo "      ⚠️  Missing tenant detection"
      fi

      # Check if layout validates slug
      if grep -q "tenant.slug !== '$TENANT_SLUG'" layout.tsx || grep -q "tenant.slug === '$TENANT_SLUG'" layout.tsx; then
        echo "      ✅ Validates tenant slug"
      else
        echo "      ⚠️  Missing slug validation"
      fi
    else
      echo "   ❌ layout.tsx missing"
      echo "      Run: /pp-tenant to create tenant structure"
    fi
    echo ""

    # List all pages
    echo "   Pages:"
    PAGES=$(find . -name "page.tsx" -o -name "page.ts" | sort)
    if [ -n "$PAGES" ]; then
      echo "$PAGES" | while read page; do
        PAGE_NAME=$(dirname "$page" | sed 's/^\.\///')

        # Check file extension
        if [[ "$page" == *.tsx ]]; then
          EXT_STATUS="✅"
        else
          EXT_STATUS="⚠️  (.ts - should be .tsx)"
        fi

        echo "      $EXT_STATUS $PAGE_NAME"

        # Check for tenant_id filtering in page
        if grep -q "tenant_id" "$page"; then
          echo "         ✅ Uses tenant_id filtering"
        elif grep -q "supabase" "$page" && ! grep -q "tenant_id" "$page"; then
          echo "         ⚠️  WARNING: Has database queries but NO tenant_id filter!"
        fi
      done
    else
      echo "      ⚠️  No pages found"
      echo "      Run: /pp-page $TENANT_SLUG <page-name>"
    fi
    echo ""

    # Check for components
    if [ -d "components" ]; then
      echo "   Components:"
      find components -name "*.tsx" -o -name "*.ts" | sed 's/^/      - /'
      echo ""
    fi

  else
    echo "❌ Tenant folder not found: $TENANT_FOLDER"
    echo "   Run: /pp-tenant \"$TENANT_SLUG\" to create tenant"
    echo ""
  fi
else
  echo "⚠️  No session active"
  echo ""
fi
```

### Step 5: Check for Common Issues

```bash
echo "═══════════════════════════════════════════"
echo "   COMMON ISSUES CHECK"
echo "═══════════════════════════════════════════"
echo ""

if [ -f ~/.claude-portal-session ]; then
  source ~/.claude-portal-session

  cd "$PORTAL_PLATFORM"

  # Check for wrong file extensions
  echo "Wrong File Extensions:"
  WRONG_EXT=$(find app/(tenants)/$TENANT_SLUG -name "page.ts" 2>/dev/null)
  if [ -n "$WRONG_EXT" ]; then
    echo "   ⚠️  Found .ts files (should be .tsx):"
    echo "$WRONG_EXT" | sed 's/^/      /'
  else
    echo "   ✅ No .ts files in pages (all .tsx)"
  fi
  echo ""

  # Check for missing tenant_id filters
  echo "Missing tenant_id Filters:"
  MISSING_FILTERS=$(grep -r "supabase.from\|from('portal\." app/(tenants)/$TENANT_SLUG --include="*.tsx" --include="*.ts" 2>/dev/null | grep -v "tenant_id" | grep -v "getTenantFromHostname" || echo "")
  if [ -n "$MISSING_FILTERS" ]; then
    echo "   ⚠️  WARNING: Database queries without tenant_id filter:"
    echo "$MISSING_FILTERS" | sed 's/^/      /'
  else
    echo "   ✅ All queries appear to filter by tenant_id"
  fi
  echo ""

  # Check for files in wrong locations
  echo "Files in Wrong Locations:"
  WRONG_LOCATION=$(find lib -name "*$TENANT_SLUG*" 2>/dev/null)
  if [ -n "$WRONG_LOCATION" ]; then
    echo "   ⚠️  Tenant-specific files in shared lib/:"
    echo "$WRONG_LOCATION" | sed 's/^/      /'
    echo "      These should be in app/(tenants)/$TENANT_SLUG/"
  else
    echo "   ✅ No tenant-specific files in shared locations"
  fi
  echo ""

  # Check git status
  echo "Git Status (Platform):"
  if [ -n "$(git status --porcelain)" ]; then
    echo "   ⚠️  Uncommitted changes:"
    git status --short | sed 's/^/      /'
    echo ""

    # Check if changes are outside tenant folder
    OUTSIDE_CHANGES=$(git status --short | grep -v "app/(tenants)/$TENANT_SLUG" || echo "")
    if [ -n "$OUTSIDE_CHANGES" ]; then
      echo "   ⚠️  WARNING: Changes OUTSIDE tenant folder:"
      echo "$OUTSIDE_CHANGES" | sed 's/^/      /'
      echo "      This may affect other tenants!"
    else
      echo "   ✅ Changes only in tenant folder"
    fi
  else
    echo "   ✅ No uncommitted changes"
  fi
  echo ""

fi
```

### Step 6: Summary and Recommendations

```bash
echo "═══════════════════════════════════════════"
echo "   DIAGNOSTICS SUMMARY"
echo "═══════════════════════════════════════════"
echo ""

# Collect all issues
ISSUES=()

[ ! -f ~/.claude-portal-session ] && ISSUES+=("No session - run /pp-setup")
[ ! -d "$PORTAL_PLATFORM" ] && ISSUES+=("Portal platform not cloned")
[ ! -f "$PORTAL_PLATFORM/.env.local" ] && ISSUES+=(".env.local missing")
[ ! -d "$PORTAL_PLATFORM/node_modules" ] && ISSUES+=("Dependencies not installed")
[ ! -d "$TENANT_REPO" ] && ISSUES+=("Tenant repository not created")
[ ! -d "$PORTAL_PLATFORM/app/(tenants)/$TENANT_SLUG" ] && ISSUES+=("Tenant folder not created - run /pp-tenant")

if [ ${#ISSUES[@]} -eq 0 ]; then
  echo "✅ No critical issues found!"
  echo ""
  echo "Next steps:"
  echo "  - Create pages: /pp-page $TENANT_SLUG <page-name>"
  echo "  - Work on issues: /pp-issue $TENANT_SLUG <issue-number>"
  echo "  - List tenants: /pp-list"
else
  echo "⚠️  Issues found:"
  printf '   %s\n' "${ISSUES[@]}"
  echo ""
  echo "Recommended fixes:"
  [ ! -f ~/.claude-portal-session ] && echo "   1. Run: /pp-setup"
  [ ! -f "$PORTAL_PLATFORM/.env.local" ] && echo "   2. Create .env.local and add Supabase credentials"
  [ ! -d "$PORTAL_PLATFORM/node_modules" ] && echo "   3. Run: npm install in portal-platform"
  [ ! -d "$PORTAL_PLATFORM/app/(tenants)/$TENANT_SLUG" ] && echo "   4. Run: /pp-tenant to create tenant"
fi

echo ""
echo "═══════════════════════════════════════════"
```

## Quick Diagnostic Modes

### Session Only
If user runs `/pp-diag session`, only run Step 1.

### Tenant Only
If user runs `/pp-diag tenant <slug>`, run Steps 3-4 for that tenant.

### Files Only
If user runs `/pp-diag files`, run Step 4 and check file structure.

### Git Only
If user runs `/pp-diag git`, check git status for both platform and tenant repos.

## Output Example

Show user a comprehensive report with:
- ✅ Green checks for what's working
- ⚠️  Yellow warnings for issues that should be fixed
- ❌ Red errors for critical problems
- Clear next steps

This helps identify setup issues, missing files, wrong file extensions, missing tenant_id filters, and more.
