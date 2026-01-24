---
description: Work on tenant-specific GitHub issue (e.g., /pp-issue fitzone 42)
autoApprove:
  - Bash(gh issue view*)
  - Bash(git clone*)
  - Bash(cd*)
  - Bash(git checkout*)
  - Bash(git pull*)
  - Bash(git branch*)
  - Bash(npm install*)
  - Bash(npm run build*)
  - Bash(npm run dev*)
  - Bash(npm test*)
  - Bash(git add*)
  - Bash(git commit*)
  - Bash(git push*)
  - Bash(gh pr create*)
  - Bash(mkdir*)
  - Read
  - Glob
  - Grep
  - Edit
  - Write
---

You are working on a GitHub issue for a specific tenant in the portal-platform multi-tenant SaaS.

## Command Format

`/pp-issue <slug> <issue-number>`

Examples:
- `/pp-issue fitzone 42` → Work on issue #42 in beyond-spreadsheets/tenant-fitzone
- `/pp-issue bs 15` → Work on issue #15 in beyond-spreadsheets/tenant-bs

## What This Does

1. Fetches issue from tenant-specific GitHub repo
2. Creates isolated work folder
3. Clones portal-platform codebase
4. Creates feature branch
5. Guides you through investigation and implementation
6. Commits changes to tenant repo (migrations/docs)
7. Commits code changes to portal-platform
8. Creates PR

## Step 1: Parse Arguments

Extract:
- `slug` → Tenant slug (e.g., "fitzone")
- `issue-number` → Issue number (e.g., "42")
- `repo` → `beyond-spreadsheets/tenant-{slug}`

## Step 2: Fetch Issue Details

```bash
gh issue view {issue-number} --repo beyond-spreadsheets/tenant-{slug}
```

Show the user:
```
═══════════════════════════════════════════
   WORKING ON TENANT ISSUE
═══════════════════════════════════════════

Tenant: {slug}
Issue: #{issue-number}
Repo: beyond-spreadsheets/tenant-{slug}

Title: {issue-title}
{issue-body}
```

## Step 3: Ask Clarifying Questions

Ask 5 questions to help define the issue and approach:

Example questions:
1. Is this a new feature, bug fix, or enhancement?
2. Does this affect the shared platform code or just tenant-specific pages?
3. Should this change be applied to other tenants too?
4. Will this require database migrations?
5. Are there any specific design/brand requirements for this tenant?

## Step 4: Create Todo List

Use TodoWrite to create task list including:
- [ ] Investigate issue in codebase
- [ ] Determine if platform or tenant-specific change
- [ ] Implement solution
- [ ] Update tenant docs/migrations (if needed)
- [ ] Test locally
- [ ] Run build and tests
- [ ] Commit to tenant repo (if migrations/docs)
- [ ] Commit to platform (if code changes)
- [ ] Create PR
- [ ] Update issue with progress

## Step 5: Create Work Folder

Create descriptive folder based on issue:

```bash
# Example: fitzone-add-booking-widget-42
WORK_FOLDER="/tmp/pp_{slug}_{issue-slug}_{issue-number}"

mkdir -p "$WORK_FOLDER"
cd "$WORK_FOLDER"

echo "📂 Work folder: $WORK_FOLDER"
```

## Step 6: Clone Portal Platform

```bash
# Clone the shared platform codebase
git clone https://github.com/beyond-spreadsheets/portal-platform.git
cd portal-platform

# Checkout develop branch and pull latest
git checkout develop
git pull origin develop

# Create feature branch
BRANCH_NAME="{slug}/issue-{issue-number}-{short-description}"
git checkout -b "$BRANCH_NAME"

echo "✅ Branch created: $BRANCH_NAME"
```

## Step 7: Clone Tenant Repo (for migrations/docs)

```bash
cd "$WORK_FOLDER"

# Clone tenant repo
git clone git@github.com:beyond-spreadsheets/tenant-{slug}.git
cd tenant-{slug}

# Create feature branch
git checkout -b "issue-{issue-number}-{short-description}"

echo "✅ Tenant repo ready for migrations/docs"
```

## Step 8: Investigate Issue

Explore the codebase to understand:
1. Where is the relevant code?
2. Is this in shared platform or tenant-specific folder?
3. What files need to be changed?
4. Are there similar implementations to reference?

Use Glob, Grep, Read tools to explore.

Update todo list: Mark "Investigate issue" as completed.

## Step 9: CRITICAL DECISION - Platform vs Tenant-Specific

**⚠️  STOP AND THINK: This is the most important decision!**

Ask the user explicitly:

```
═══════════════════════════════════════════
   CRITICAL DECISION POINT
═══════════════════════════════════════════

This issue is for: {Tenant Name} ({slug})

Question: Should this change affect ALL tenants or ONLY {Tenant Name}?

Examples:

ALL TENANTS (Platform Code):
✅ Bug fix in auth system
✅ New shared component everyone can use
✅ Security patch
✅ Email template improvement
✅ Payment processing fix
→ Changes go to: lib/, components/shared/, app/api/

ONLY THIS TENANT (Tenant-Specific):
✅ Custom page for this client
✅ Brand-specific styling
✅ Unique business logic for this client
✅ Custom navigation for this tenant
→ Changes go to: app/(tenants)/{slug}/

DATABASE CHANGES:
✅ Tenant-specific data (bookings, users, etc.)
→ Migration goes to: tenant repo only
✅ Platform schema changes (all tenants)
→ Migration goes to: platform repo (requires careful review!)

Please choose:
1. ALL TENANTS - Platform code change
2. ONLY {Tenant Name} - Tenant-specific change
3. DATABASE - Data migration

>
```

**Based on answer, set variables:**

```bash
CHANGE_TYPE="" # "platform" or "tenant" or "database"
AFFECTS_ALL_TENANTS=false # true if platform change
NEEDS_PLATFORM_PR=false
NEEDS_TENANT_PR=false
```

**If PLATFORM change (affects ALL tenants):**
```
⚠️  WARNING: PLATFORM CHANGE DETECTED

This change will affect ALL {TOTAL_TENANT_COUNT} tenants:
- {list first 5 tenant names}
- ... and {remaining} others

Changes to these folders affect everyone:
- lib/ (shared utilities)
- components/shared/ (shared components)
- app/api/ (API routes)
- middleware.ts (auth/routing)
- Any file outside app/(tenants)/

Are you SURE this should affect all tenants?
>
```

Set:
```bash
CHANGE_TYPE="platform"
AFFECTS_ALL_TENANTS=true
NEEDS_PLATFORM_PR=true
WORK_LOCATION="portal-platform/"
```

**If TENANT-SPECIFIC change:**
```
✅ TENANT-SPECIFIC CHANGE

This change will ONLY affect: {Tenant Name}

Work location: app/(tenants)/{slug}/
Other tenants: Not affected
Platform code: Not modified
```

Set:
```bash
CHANGE_TYPE="tenant"
AFFECTS_ALL_TENANTS=false
NEEDS_PLATFORM_PR=true # Still need to commit tenant folder to platform
WORK_LOCATION="portal-platform/app/(tenants)/{slug}/"
```

**If DATABASE change:**
```
📊 DATABASE CHANGE

Migration location: tenant-{slug}/migrations/
Affects: {Tenant Name} data only
Platform schema: Not modified

If this needs to change platform schema (affects all tenants),
please reconsider and choose "ALL TENANTS" instead.
```

Set:
```bash
CHANGE_TYPE="database"
AFFECTS_ALL_TENANTS=false
NEEDS_TENANT_PR=true
WORK_LOCATION="tenant-{slug}/migrations/"
```

## Step 10: Implement Solution

Implement the fix/feature in appropriate location.

Use Edit or Write tools to modify code.

Update todo list: Mark "Implement solution" as completed.

## Step 11: Update Tenant Repo (if migrations/docs)

If you created migrations or need to document decisions:

```bash
cd "$WORK_FOLDER/tenant-{slug}"

# If migration created
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
# Create migration file with actual SQL
cat > migrations/${TIMESTAMP}_issue_{issue-number}_{description}.sql << 'EOF'
-- Migration for issue #{issue-number}
-- {issue-title}

{ACTUAL_SQL_HERE}
EOF

git add migrations/
git commit -m "feat: Add migration for issue #{issue-number}

{issue-title}

Changes:
- {list changes}

Fixes: #{issue-number}"

# Update docs
cat >> docs/decisions.md << 'EOF'

### {DATE} - Issue #{issue-number}: {issue-title}
- Decision: {what was decided}
- Migration: migrations/{timestamp}_issue_{issue-number}.sql
- Affects: {what it affects}

EOF

git add docs/
git commit -m "docs: Document decision for issue #{issue-number}"

git push -u origin issue-{issue-number}-{short-description}
```

## Step 12: Test Solution

```bash
cd "$WORK_FOLDER/portal-platform"

# Install dependencies (if needed)
npm install

# Run build
npm run build

# If build fails, fix errors and try again
# Don't proceed if build fails!

# Run tests (if applicable)
npm test

echo "✅ Build and tests passed"
```

Update todo list: Mark "Test locally" and "Run build and tests" as completed.

## Step 13: Commit Changes (Context-Aware)

**Based on CHANGE_TYPE from Step 9:**

### If PLATFORM change (AFFECTS_ALL_TENANTS=true):

```bash
cd "$WORK_FOLDER/portal-platform"

# Show affected files
echo "⚠️  PLATFORM CHANGE - Will affect ALL tenants"
echo ""
echo "Files changed:"
git diff --name-only

# Confirm with user
echo ""
echo "These changes will be deployed to ALL {TOTAL_TENANT_COUNT} tenants."
echo "Are you sure? (y/n)"
read -r CONFIRM

if [ "$CONFIRM" != "y" ]; then
  echo "Aborting. Please review changes."
  exit 1
fi

git add .

git commit -m "$(cat <<'EOF'
feat(platform): {issue-title}

⚠️  PLATFORM CHANGE - Affects ALL tenants

{Detailed description of changes}

Changes:
- {list of specific changes}

Impact: All {TOTAL_TENANT_COUNT} tenants will receive this change
Origin: Requested by {Tenant Name} - beyond-spreadsheets/tenant-{slug}#{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

git push -u origin "$BRANCH_NAME"

echo "✅ Platform changes committed"
echo "⚠️  Remember: This will affect ALL tenants when merged"
```

### If TENANT-SPECIFIC change (AFFECTS_ALL_TENANTS=false):

```bash
cd "$WORK_FOLDER/portal-platform"

# Show affected files
echo "✅ TENANT-SPECIFIC CHANGE - Only affects {Tenant Name}"
echo ""
echo "Files changed:"
git diff --name-only app/(tenants)/{slug}/

# Verify only tenant folder changed
if git diff --name-only | grep -v "app/(tenants)/{slug}/" ; then
  echo "⚠️  WARNING: Changes detected outside tenant folder!"
  echo "This may affect other tenants. Please review."
  exit 1
fi

git add app/(tenants)/{slug}/

git commit -m "$(cat <<'EOF'
feat({slug}): {issue-title}

✅ TENANT-SPECIFIC - Only affects {Tenant Name}

{Detailed description of changes}

Changes:
- {list of specific changes}

Impact: Only {Tenant Name} affected
Other tenants: No changes

Fixes: beyond-spreadsheets/tenant-{slug}#{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

git push -u origin "$BRANCH_NAME"

echo "✅ Tenant-specific changes committed"
echo "✅ Other tenants will not be affected"
```

### If DATABASE change only:

```bash
# No platform commit needed - only tenant repo migration

cd "$WORK_FOLDER/tenant-{slug}"

git add migrations/

git commit -m "$(cat <<'EOF'
feat: Database migration for issue #{issue-number}

{issue-title}

Migration: migrations/{TIMESTAMP}_issue_{issue-number}.sql

Changes:
- {list of database changes}

Impact: {Tenant Name} data only

Fixes: #{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"

git push -u origin issue-{issue-number}-{short-description}

echo "✅ Migration committed to tenant repo"
echo "✅ No platform changes needed"
```

Update todo list: Mark "Commit to platform" or "Commit to tenant repo" as completed.

## Step 14: Create Pull Requests (Context-Aware)

**Based on CHANGE_TYPE:**

### If PLATFORM change (affects ALL tenants):

**Platform PR only:**
```bash
cd "$WORK_FOLDER/portal-platform"

gh pr create --repo beyond-spreadsheets/portal-platform \
  --title "feat(platform): {issue-title}" \
  --base develop \
  --body "$(cat <<'EOF'
## Summary
⚠️  **PLATFORM CHANGE - Affects ALL tenants**

{Detailed summary}

## Origin
Requested by: {Tenant Name}
Original issue: beyond-spreadsheets/tenant-{slug}#{issue-number}

## Changes
{List changes}

## Impact Analysis
- **Affected tenants**: ALL ({TOTAL_TENANT_COUNT} tenants)
- **Risk level**: [Low/Medium/High]
- **Backwards compatible**: [Yes/No]

## Type of Change
- [x] Platform improvement (benefits all tenants)
- [ ] Bug fix
- [ ] Security patch
- [ ] Breaking change

## Testing
- [x] Build passed
- [x] Tests passed
- [ ] Tested with multiple tenants
- [ ] Reviewed by platform team

## Deployment Notes
- Deploy during low-traffic window
- Monitor all tenants after deployment
- Rollback plan: [describe]

## Related Issues
- Origin: beyond-spreadsheets/tenant-{slug}#{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

⚠️  **REVIEW CAREFULLY - ALL TENANTS WILL BE AFFECTED**
EOF
)"

echo ""
echo "⚠️  PLATFORM PR CREATED"
echo "This PR will affect ALL tenants when merged."
echo "Ensure thorough review before merging!"
```

### If TENANT-SPECIFIC change:

**Tenant repo PR (for docs):**
```bash
cd "$WORK_FOLDER/tenant-{slug}"

gh pr create --repo beyond-spreadsheets/tenant-{slug} \
  --title "Issue #{issue-number}: {issue-title}" \
  --body "$(cat <<'EOF'
## Summary
✅ TENANT-SPECIFIC CHANGE

Resolves #{issue-number}

{Brief summary of changes}

## Changes
- Documentation updated in `docs/decisions.md`
- Configuration: `config/tenant.json` (if changed)

## Impact
- Affects: {Tenant Name} only
- Other tenants: No impact

## Testing
- [ ] Tested locally
- [ ] Documentation reviewed

Fixes #{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**Platform PR (for code):**
```bash
cd "$WORK_FOLDER/portal-platform"

gh pr create --repo beyond-spreadsheets/portal-platform \
  --title "feat({slug}): {issue-title}" \
  --base develop \
  --body "$(cat <<'EOF'
## Summary
✅ **TENANT-SPECIFIC - Only affects {Tenant Name}**

Implementation for issue beyond-spreadsheets/tenant-{slug}#{issue-number}

{Detailed summary}

## Changes
All changes in: `app/(tenants)/{slug}/`

{List changes}

## Impact Analysis
- **Affected tenants**: {Tenant Name} ONLY
- **Other tenants**: No impact
- **Files changed**: Only `app/(tenants)/{slug}/**`

## Type of Change
- [x] Tenant-specific feature
- [ ] Bug fix (tenant-only)
- [ ] New page/component (tenant-only)

## Testing
- [x] Build passed
- [x] Tests passed
- [ ] Tested locally
- [x] Verified no impact on other tenants

## Related Issues
- Tenant issue: beyond-spreadsheets/tenant-{slug}#{issue-number}
- Tenant repo PR: [link after creating above]

🤖 Generated with [Claude Code](https://claude.com/claude-code)

✅ **SAFE TO MERGE - Only affects {Tenant Name}**
EOF
)"

echo ""
echo "✅ TENANT-SPECIFIC PR CREATED"
echo "This PR only affects {Tenant Name}."
echo "Other tenants will not be impacted."
```

### If DATABASE change only:

**Tenant repo PR only:**
```bash
cd "$WORK_FOLDER/tenant-{slug}"

gh pr create --repo beyond-spreadsheets/tenant-{slug} \
  --title "Database migration: {issue-title}" \
  --body "$(cat <<'EOF'
## Summary
📊 DATABASE MIGRATION

Resolves #{issue-number}

{Brief summary of changes}

## Changes
- Migration: `migrations/{TIMESTAMP}_issue_{issue-number}.sql`
- Documentation updated in `docs/decisions.md`

## Migration Details
```sql
{Paste key SQL here}
```

## Rollback Plan
```sql
{Paste rollback SQL here}
```

## Testing
- [ ] Migration tested locally
- [ ] Backup created before migration
- [ ] Rollback tested

## Impact
- Affects: {Tenant Name} data only
- Platform code: No changes
- Other tenants: No impact

Fixes #{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"

echo ""
echo "✅ DATABASE MIGRATION PR CREATED"
echo "No platform changes needed."
echo "This only affects {Tenant Name} data."
```

Update todo list: Mark "Create PR" as completed.

## Step 15: Update Issue

Comment on the original issue with progress:

```bash
gh issue comment {issue-number} --repo beyond-spreadsheets/tenant-{slug} --body "$(cat <<'EOF'
## Update

Work has been completed for this issue.

### Pull Requests Created:
- Tenant repo: {PR_URL_1}
- Platform repo: {PR_URL_2}

### Changes:
{Summary of changes}

### Next Steps:
1. Review PRs
2. Test on staging
3. Merge and deploy

The migration and code are ready for review.
EOF
)"
```

Update todo list: Mark "Update issue with progress" as completed.

## Step 16: Summary

Show the user:

```
✅ Issue #{issue-number} Work Complete!

📂 Work Folder: {WORK_FOLDER}

📝 Pull Requests:
   Tenant repo: {tenant-pr-url}
   Platform: {platform-pr-url}

📋 Changes Summary:
   {summary-of-what-changed}

🎯 Next Steps:
   1. Review PRs
   2. Test on staging environment
   3. Merge to main
   4. Deploy to production
   5. Close issue #{issue-number}

💾 Files to keep:
   - Tenant repo: ~/portal-work/tenants/{slug} (if you need to continue)
   - Work folder: {WORK_FOLDER} (can delete after PR merged)
```

## Important Notes

**Platform vs Tenant-Specific:**
- Platform changes benefit ALL tenants
- Tenant-specific changes only affect one client
- Ask yourself: "Should other tenants get this too?"

**Migrations:**
- Always save to tenant repo: `tenant-{slug}/migrations/`
- Include rollback SQL in comments
- Document in decision log

**Testing:**
- MUST pass build
- MUST pass tests (if applicable)
- Don't create PR if tests fail

**Git Workflow:**
- Tenant repo PR → tenant-specific branch
- Platform PR → develop branch
- Link PRs to each other

**Safety:**
- Work in isolated folder (don't affect other sessions)
- Test thoroughly before pushing
- Document all decisions

This command integrates seamlessly with your multi-tenant architecture while maintaining full audit trails!
