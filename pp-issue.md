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
- `/pp-issue beyond-spreadsheets 15` → Work on issue #15 in beyond-spreadsheets/tenant-beyond-spreadsheets

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

## Step 9: Determine Change Type

**If PLATFORM change (affects shared code):**
- Work in: `portal-platform/` folder
- Change: `lib/`, `components/shared/`, `app/api/`, etc.
- Benefit: All tenants get this improvement!

**If TENANT-SPECIFIC change:**
- Work in: `portal-platform/app/(tenants)/{slug}/` folder
- Change: Only this tenant's pages/components
- Isolated: Won't affect other tenants

**If DATABASE change:**
- Create migration in tenant repo: `tenant-{slug}/migrations/`
- Document in: `tenant-{slug}/docs/decisions.md`

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

## Step 13: Commit Platform Changes

```bash
cd "$WORK_FOLDER/portal-platform"

git add .

git commit -m "$(cat <<'EOF'
feat({slug}): {issue-title}

{Detailed description of changes}

Changes:
- {list of specific changes}

Fixes: beyond-spreadsheets/tenant-{slug}#{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

git push -u origin "$BRANCH_NAME"
```

Update todo list: Mark "Commit to platform" as completed.

## Step 14: Create Pull Requests

**For tenant repo (if migrations/docs):**
```bash
cd "$WORK_FOLDER/tenant-{slug}"

gh pr create --repo beyond-spreadsheets/tenant-{slug} \
  --title "Issue #{issue-number}: {issue-title}" \
  --body "$(cat <<'EOF'
## Summary
Resolves #{issue-number}

{Brief summary of changes}

## Changes
- Migration: `migrations/{timestamp}_issue_{issue-number}.sql`
- Documentation updated in `docs/decisions.md`

## Testing
- [ ] Migration tested locally
- [ ] Documentation reviewed

Fixes #{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**For platform repo:**
```bash
cd "$WORK_FOLDER/portal-platform"

gh pr create --repo beyond-spreadsheets/portal-platform \
  --title "{slug}: {issue-title}" \
  --base develop \
  --body "$(cat <<'EOF'
## Summary
Implementation for {Tenant Name} - issue beyond-spreadsheets/tenant-{slug}#{issue-number}

{Detailed summary}

## Changes
{List changes}

## Type of Change
- [ ] Platform improvement (benefits all tenants)
- [ ] Tenant-specific feature (only {slug})
- [ ] Bug fix
- [ ] Database migration

## Testing
- [x] Build passed
- [x] Tests passed
- [ ] Tested locally

## Related Issues
- Tenant issue: beyond-spreadsheets/tenant-{slug}#{issue-number}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
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
