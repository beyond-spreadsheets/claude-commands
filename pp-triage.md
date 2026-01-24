---
description: Triage issues across all tenant repositories
autoApprove:
  - Bash(gh issue list*)
  - Bash(gh issue view*)
  - Bash(find*)
  - Bash(ls*)
  - Read
---

You are triaging GitHub issues across ALL tenant repositories in the Portal Platform.

## Command Format

`/pp-triage [optional: slug]`

Examples:
- `/pp-triage` - Triage all tenant repos
- `/pp-triage drc` - Triage only Derby Rugby Club repo
- `/pp-triage --priority` - Show only high priority issues

## What This Does

1. Discovers all tenant repositories
2. Lists open issues from each tenant's GitHub repo
3. Categorizes issues by type and priority
4. Creates a prioritized work queue
5. Suggests which issues to work on first with `/pp-issue`

## Step 1: Discover Tenant Repositories

```bash
echo "═══════════════════════════════════════════"
echo "   DISCOVERING TENANT REPOSITORIES"
echo "═══════════════════════════════════════════"
echo ""

# Check if we have a session
if [ -f ~/.claude-portal-session ]; then
  source ~/.claude-portal-session
  TENANTS_DIR="$HOME/portal-work/tenants"
else
  TENANTS_DIR="$HOME/portal-work/tenants"
fi

# Find all tenant repos
if [ -d "$TENANTS_DIR" ]; then
  TENANT_REPOS=$(find "$TENANTS_DIR" -mindepth 1 -maxdepth 1 -type d)
  TENANT_COUNT=$(echo "$TENANT_REPOS" | wc -l | tr -d ' ')

  echo "Found $TENANT_COUNT tenant repositor(ies):"
  echo "$TENANT_REPOS" | while read repo; do
    SLUG=$(basename "$repo")
    echo "  - $SLUG"
  done
  echo ""
else
  echo "❌ No tenant repositories found at: $TENANTS_DIR"
  echo "   Run: /pp-setup to create tenant repos"
  exit 1
fi
```

## Step 2: Fetch Issues from GitHub

```bash
echo "═══════════════════════════════════════════"
echo "   FETCHING ISSUES FROM GITHUB"
echo "═══════════════════════════════════════════"
echo ""

# Create temporary file to store all issues
ISSUES_FILE=$(mktemp)
echo "Tenant,Number,Title,State,Labels,Priority,URL" > "$ISSUES_FILE"

echo "$TENANT_REPOS" | while read repo; do
  SLUG=$(basename "$repo")
  GITHUB_REPO="beyond-spreadsheets/tenant-$SLUG"

  echo "Checking: $GITHUB_REPO"

  # Check if repo exists on GitHub
  if gh repo view "$GITHUB_REPO" >/dev/null 2>&1; then
    # Fetch open issues
    gh issue list --repo "$GITHUB_REPO" --state open --json number,title,labels,url --limit 100 | \
      jq -r --arg slug "$SLUG" '.[] |
        "\($slug),\(.number),\(.title),open,\([.labels[].name] | join(";")),\(
          if ([.labels[].name] | any(. == "priority:high" or . == "urgent" or . == "critical")) then "HIGH"
          elif ([.labels[].name] | any(. == "priority:medium")) then "MEDIUM"
          elif ([.labels[].name] | any(. == "priority:low")) then "LOW"
          else "NORMAL"
          end
        ),\(.url)"' >> "$ISSUES_FILE"
  else
    echo "  ⚠️  Repo not found on GitHub (may be local only)"
  fi
done

echo ""
```

## Step 3: Categorize and Prioritize Issues

```bash
echo "═══════════════════════════════════════════"
echo "   ISSUE CATEGORIZATION"
echo "═══════════════════════════════════════════"
echo ""

# Count issues
TOTAL_ISSUES=$(($(wc -l < "$ISSUES_FILE") - 1))  # Subtract header row

if [ "$TOTAL_ISSUES" -eq 0 ]; then
  echo "✅ No open issues found across all tenant repositories!"
  echo ""
  echo "All tenants are in good shape. Great work! 🎉"
  exit 0
fi

echo "📊 Total Open Issues: $TOTAL_ISSUES"
echo ""

# Categorize by priority
HIGH_PRIORITY=$(grep ",HIGH," "$ISSUES_FILE" | wc -l | tr -d ' ')
MEDIUM_PRIORITY=$(grep ",MEDIUM," "$ISSUES_FILE" | wc -l | tr -d ' ')
LOW_PRIORITY=$(grep ",LOW," "$ISSUES_FILE" | wc -l | tr -d ' ')
NORMAL_PRIORITY=$(grep ",NORMAL," "$ISSUES_FILE" | wc -l | tr -d ' ')

echo "Priority Breakdown:"
echo "  🔴 HIGH:   $HIGH_PRIORITY"
echo "  🟡 MEDIUM: $MEDIUM_PRIORITY"
echo "  🔵 LOW:    $LOW_PRIORITY"
echo "  ⚪ NORMAL: $NORMAL_PRIORITY"
echo ""

# Categorize by type (based on labels)
echo "Issue Types:"
grep "bug" "$ISSUES_FILE" | wc -l | tr -d ' ' | xargs echo "  🐛 Bugs:          "
grep "feature\|enhancement" "$ISSUES_FILE" | wc -l | tr -d ' ' | xargs echo "  ✨ Features:      "
grep "documentation" "$ISSUES_FILE" | wc -l | tr -d ' ' | xargs echo "  📝 Documentation: "
grep "security" "$ISSUES_FILE" | wc -l | tr -d ' ' | xargs echo "  🔒 Security:      "
echo ""
```

## Step 4: Show Prioritized Work Queue

```bash
echo "═══════════════════════════════════════════"
echo "   PRIORITIZED WORK QUEUE"
echo "═══════════════════════════════════════════"
echo ""

# Function to display issues
display_issues() {
  local priority=$1
  local emoji=$2
  local color=$3

  COUNT=$(grep ",$priority," "$ISSUES_FILE" | wc -l | tr -d ' ')

  if [ "$COUNT" -gt 0 ]; then
    echo "$emoji $color PRIORITY ($COUNT issues):"
    echo ""

    grep ",$priority," "$ISSUES_FILE" | while IFS=, read -r tenant number title state labels priority url; do
      # Clean up title
      title=$(echo "$title" | sed 's/"//g')

      # Extract issue type from labels
      TYPE=""
      if echo "$labels" | grep -q "bug"; then TYPE="🐛 Bug"; fi
      if echo "$labels" | grep -q "feature"; then TYPE="✨ Feature"; fi
      if echo "$labels" | grep -q "enhancement"; then TYPE="✨ Enhancement"; fi
      if echo "$labels" | grep -q "security"; then TYPE="🔒 Security"; fi
      if echo "$labels" | grep -q "documentation"; then TYPE="📝 Docs"; fi
      if [ -z "$TYPE" ]; then TYPE="📋 Task"; fi

      echo "  $TYPE | $tenant #$number"
      echo "  Title: $title"
      echo "  Command: /pp-issue $tenant $number"
      echo "  URL: $url"
      echo ""
    done
  fi
}

# Display in priority order
display_issues "HIGH" "🔴" "HIGH"
display_issues "MEDIUM" "🟡" "MEDIUM"
display_issues "NORMAL" "⚪" "NORMAL"
display_issues "LOW" "🔵" "LOW"
```

## Step 5: Security and Critical Issues Alert

```bash
echo "═══════════════════════════════════════════"
echo "   SECURITY & CRITICAL ALERTS"
echo "═══════════════════════════════════════════"
echo ""

# Check for security issues
SECURITY_ISSUES=$(grep -i "security\|vulnerability\|exploit" "$ISSUES_FILE" | wc -l | tr -d ' ')

if [ "$SECURITY_ISSUES" -gt 0 ]; then
  echo "🚨 SECURITY ISSUES FOUND: $SECURITY_ISSUES"
  echo ""
  echo "These should be addressed IMMEDIATELY:"
  echo ""

  grep -i "security\|vulnerability\|exploit" "$ISSUES_FILE" | while IFS=, read -r tenant number title state labels priority url; do
    title=$(echo "$title" | sed 's/"//g')
    echo "  🔒 $tenant #$number: $title"
    echo "     → /pp-issue $tenant $number"
  done
  echo ""
else
  echo "✅ No security issues found"
  echo ""
fi

# Check for platform-wide issues
PLATFORM_ISSUES=$(grep -i "platform\|all-tenants\|global" "$ISSUES_FILE" | wc -l | tr -d ' ')

if [ "$PLATFORM_ISSUES" -gt 0 ]; then
  echo "⚠️  PLATFORM-WIDE ISSUES: $PLATFORM_ISSUES"
  echo ""
  echo "These may affect ALL tenants:"
  echo ""

  grep -i "platform\|all-tenants\|global" "$ISSUES_FILE" | while IFS=, read -r tenant number title state labels priority url; do
    title=$(echo "$title" | sed 's/"//g')
    echo "  ⚠️  $tenant #$number: $title"
    echo "     → /pp-issue $tenant $number"
  done
  echo ""
fi
```

## Step 6: Tenant Health Summary

```bash
echo "═══════════════════════════════════════════"
echo "   TENANT HEALTH SUMMARY"
echo "═══════════════════════════════════════════"
echo ""

echo "Issues per tenant:"
echo ""

echo "$TENANT_REPOS" | while read repo; do
  SLUG=$(basename "$repo")
  COUNT=$(grep "^$SLUG," "$ISSUES_FILE" | wc -l | tr -d ' ')

  if [ "$COUNT" -eq 0 ]; then
    STATUS="✅ Healthy"
  elif [ "$COUNT" -le 3 ]; then
    STATUS="🟢 Good ($COUNT issues)"
  elif [ "$COUNT" -le 10 ]; then
    STATUS="🟡 Needs Attention ($COUNT issues)"
  else
    STATUS="🔴 Critical ($COUNT issues)"
  fi

  echo "  $STATUS - $SLUG"

  # Show breakdown if issues exist
  if [ "$COUNT" -gt 0 ]; then
    HIGH=$(grep "^$SLUG,.*,HIGH," "$ISSUES_FILE" | wc -l | tr -d ' ')
    [ "$HIGH" -gt 0 ] && echo "       🔴 High: $HIGH"
  fi
done

echo ""
```

## Step 7: Suggested Workflow

```bash
echo "═══════════════════════════════════════════"
echo "   SUGGESTED WORKFLOW"
echo "═══════════════════════════════════════════"
echo ""

# Suggest next actions based on issues found
if [ "$SECURITY_ISSUES" -gt 0 ]; then
  echo "1️⃣  ADDRESS SECURITY ISSUES FIRST"
  echo "   These affect tenant safety and should be fixed immediately"
  echo ""
  grep -i "security\|vulnerability\|exploit" "$ISSUES_FILE" | head -1 | while IFS=, read -r tenant number title state labels priority url; do
    echo "   Start with: /pp-issue $tenant $number"
  done
  echo ""
fi

if [ "$HIGH_PRIORITY" -gt 0 ]; then
  echo "2️⃣  WORK ON HIGH PRIORITY ISSUES"
  echo "   Critical bugs and urgent features"
  echo ""
  grep ",HIGH," "$ISSUES_FILE" | head -3 | while IFS=, read -r tenant number title state labels priority url; do
    title=$(echo "$title" | sed 's/"//g' | cut -c1-60)
    echo "   • /pp-issue $tenant $number"
    echo "     $title"
  done
  echo ""
fi

if [ "$MEDIUM_PRIORITY" -gt 0 ] || [ "$NORMAL_PRIORITY" -gt 0 ]; then
  echo "3️⃣  ADDRESS MEDIUM/NORMAL PRIORITY"
  echo "   Regular features and improvements"
  echo ""
  grep -E ",MEDIUM,|,NORMAL," "$ISSUES_FILE" | head -3 | while IFS=, read -r tenant number title state labels priority url; do
    title=$(echo "$title" | sed 's/"//g' | cut -c1-60)
    echo "   • /pp-issue $tenant $number"
  done
  echo ""
fi

if [ "$LOW_PRIORITY" -gt 0 ]; then
  echo "4️⃣  BACKLOG ITEMS (Low Priority)"
  echo "   Work on these when time permits"
  echo ""
fi

echo "💡 Pro Tip: Use /pp-issue <slug> <number> to start working on any issue"
echo "   The command will guide you through investigation and implementation"
echo ""
```

## Step 8: Export Work Queue (Optional)

```bash
echo "═══════════════════════════════════════════"
echo "   EXPORT OPTIONS"
echo "═══════════════════════════════════════════"
echo ""

# Save to file for reference
EXPORT_FILE="$HOME/portal-work/triage-$(date +%Y%m%d-%H%M%S).csv"
cp "$ISSUES_FILE" "$EXPORT_FILE"

echo "📊 Full issue list exported to:"
echo "   $EXPORT_FILE"
echo ""

# Create markdown report
REPORT_FILE="$HOME/portal-work/triage-report-$(date +%Y%m%d-%H%M%S).md"

cat > "$REPORT_FILE" <<EOF
# Portal Platform - Issue Triage Report

**Generated:** $(date)
**Total Issues:** $TOTAL_ISSUES

## Priority Summary

- 🔴 HIGH: $HIGH_PRIORITY
- 🟡 MEDIUM: $MEDIUM_PRIORITY
- ⚪ NORMAL: $NORMAL_PRIORITY
- 🔵 LOW: $LOW_PRIORITY

## High Priority Issues

EOF

grep ",HIGH," "$ISSUES_FILE" | while IFS=, read -r tenant number title state labels priority url; do
  title=$(echo "$title" | sed 's/"//g')
  echo "- **$tenant #$number**: $title" >> "$REPORT_FILE"
  echo "  - Command: \`/pp-issue $tenant $number\`" >> "$REPORT_FILE"
  echo "  - URL: $url" >> "$REPORT_FILE"
  echo "" >> "$REPORT_FILE"
done

echo "📝 Markdown report saved to:"
echo "   $REPORT_FILE"
echo ""

# Cleanup temp file
rm "$ISSUES_FILE"
```

## Summary Output

```bash
echo "═══════════════════════════════════════════"
echo "   TRIAGE COMPLETE"
echo "═══════════════════════════════════════════"
echo ""

if [ "$TOTAL_ISSUES" -eq 0 ]; then
  echo "✅ All tenants are healthy - no open issues!"
  echo ""
  echo "Next steps:"
  echo "  - Monitor for new issues"
  echo "  - Create new features with /pp-page"
  echo "  - Review tenant repositories with /pp-list"
else
  echo "📋 Found $TOTAL_ISSUES issues across $TENANT_COUNT tenant(s)"
  echo ""
  echo "Recommended next action:"

  if [ "$SECURITY_ISSUES" -gt 0 ]; then
    grep -i "security\|vulnerability\|exploit" "$ISSUES_FILE" | head -1 | while IFS=, read -r tenant number title state labels priority url; do
      echo "  🔒 Security: /pp-issue $tenant $number"
    done
  elif [ "$HIGH_PRIORITY" -gt 0 ]; then
    grep ",HIGH," "$ISSUES_FILE" | head -1 | while IFS=, read -r tenant number title state labels priority url; do
      echo "  🔴 High Priority: /pp-issue $tenant $number"
    done
  else
    head -2 "$ISSUES_FILE" | tail -1 | while IFS=, read -r tenant number title state labels priority url; do
      echo "  📋 Next Issue: /pp-issue $tenant $number"
    done
  fi

  echo ""
  echo "View full report: $REPORT_FILE"
fi

echo ""
echo "═══════════════════════════════════════════"
```

## Features

**Automatic Discovery:**
- Finds all tenant repos in ~/portal-work/tenants/
- Checks GitHub for each tenant-{slug} repository
- Handles missing repos gracefully

**Smart Prioritization:**
- Security issues flagged 🚨
- Priority levels (HIGH, MEDIUM, LOW, NORMAL)
- Issue types (Bug, Feature, Security, Docs)
- Platform-wide issues highlighted

**Tenant Health:**
- Shows which tenants need attention
- Healthy (0 issues) vs Critical (10+ issues)
- Per-tenant issue breakdown

**Actionable Output:**
- Direct `/pp-issue` commands ready to run
- Suggested workflow (security → high → medium → low)
- Exported reports (CSV + Markdown)

**Use Cases:**
- Monday morning: Triage all issues across tenants
- Daily standup: Check what needs attention
- Sprint planning: Prioritize work queue
- Client review: See tenant-specific issues

This creates a complete issue management workflow integrated with the multi-tenant architecture!
