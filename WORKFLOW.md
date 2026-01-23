# Portal Platform - Complete Workflow Guide

This guide shows the complete workflow for managing multi-tenant SaaS with Git-backed configuration.

## 🏗️ Architecture

```
~/portal-work/
├── portal-platform/           # Shared codebase (all tenants)
└── tenants/
    ├── beyond-spreadsheets/   # Your website config
    │   ├── migrations/        # SQL migrations
    │   ├── docs/             # Decision logs
    │   ├── config/           # tenant.json
    │   └── backups/          # Database backups
    ├── fitzone-gym/          # Client A config
    └── dkc/                  # Client B config
```

Each tenant gets:
- **Git repo**: `beyond-spreadsheets/tenant-{slug}` (private)
- **Platform folder**: `app/(tenants)/{slug}/` (in shared codebase)
- **Database records**: Isolated by `tenant_id`

## 📋 Commands Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `/pp-setup` | Setup work session | `/pp-setup` |
| `/pp-tenant` | Create new tenant | `/pp-tenant "FitZone Gym"` |
| `/pp-page` | Create tenant page | `/pp-page fitzone home` |
| `/pp-component` | Create shared component | `/pp-component HeroSection` |
| `/pp-list` | List all tenants | `/pp-list` |
| `/pp-issue` | Work on tenant issue | `/pp-issue fitzone 42` |

## 🚀 Workflow 1: Create New Tenant

**Step 1: Setup session**
```bash
/pp-setup
> "What are you working on today?"
> "FitZone Gym" (new client)
```

Creates:
- `~/portal-work/portal-platform/` (shared codebase)
- `~/portal-work/tenants/fitzone-gym/` (Git repo for this tenant)

**Step 2: Create tenant in database**
```bash
/pp-tenant "FitZone Gym"
```

Prompts for:
- Business name: "FitZone Gym"
- Slug: "fitzone-gym" (suggested)
- Subdomain: "fitzone"
- Colors, logo, etc. (optional)

Creates:
- Database records (tenants, config, features, roles)
- Platform folder: `portal-platform/app/(tenants)/fitzone-gym/`
- Migration: `tenants/fitzone-gym/migrations/20260123_120000_initial_tenant_setup.sql`
- Config: `tenants/fitzone-gym/config/tenant.json`
- Docs: `tenants/fitzone-gym/docs/{decisions,features}.md`

**Step 3: Push tenant repo to GitHub**
```bash
cd ~/portal-work/tenants/fitzone-gym
gh repo create beyond-spreadsheets/tenant-fitzone-gym --private
git push -u origin main
```

**Step 4: Create pages**
```bash
/pp-page fitzone-gym home
/pp-page fitzone-gym about
/pp-page fitzone-gym classes
/pp-page fitzone-gym contact
```

Each page created in: `portal-platform/app/(tenants)/fitzone-gym/{page}/page.tsx`

**Step 5: Deploy**
```bash
cd ~/portal-work/portal-platform
git add app/(tenants)/fitzone-gym/
git commit -m "feat: Add FitZone Gym tenant pages"
git push
```

Vercel auto-deploys. Add domain in Vercel dashboard.

## 🐛 Workflow 2: Work on Existing Tenant Issue

**Step 1: Setup session**
```bash
/pp-setup
> "What are you working on today?"
> "FitZone Gym" (existing client)
```

**Step 2: View issues**
```bash
cd ~/portal-work/tenants/fitzone-gym
gh issue list
```

**Step 3: Work on issue**
```bash
/pp-issue fitzone-gym 42
```

This:
1. Fetches issue #42 from `beyond-spreadsheets/tenant-fitzone-gym`
2. Asks 5 clarifying questions
3. Creates work folder: `/tmp/pp_fitzone-gym_add-booking-widget_42/`
4. Clones portal-platform and tenant repo
5. Creates branches in both repos
6. Guides you through implementation
7. Saves migrations to tenant repo
8. Commits code to platform
9. Creates PRs for both repos
10. Updates issue with progress

**Step 4: Review & merge**
- Review PRs
- Merge tenant repo PR (migrations/docs)
- Merge platform PR (code changes)
- Deploy

## 🔄 Workflow 3: Onboard Multiple Clients

**Morning: Client A**
```bash
/pp-setup
> "Warriors Martial Arts"

/pp-tenant "Warriors Martial Arts"
# Create pages...

cd ~/portal-work/tenants/warriors-martial-arts
gh repo create beyond-spreadsheets/tenant-warriors-martial-arts --private
git push -u origin main
```

**Afternoon: Client B**
```bash
/pp-setup
> "Derby RFC"

/pp-tenant "Derby RFC"
# Create pages...

cd ~/portal-work/tenants/derby-rfc
gh repo create beyond-spreadsheets/tenant-derby-rfc --private
git push -u origin main
```

**Result:**
- 2 clients onboarded
- Each has own Git repo (full audit trail)
- Both share same platform code (auth, email, SMS, payments)
- Security updates push to both instantly

## 📊 Benefits of This Workflow

### Version Control
✅ Every database change tracked in migrations
✅ Git history shows who changed what and when
✅ Can roll back to any point in time

### Disaster Recovery
✅ Lost database? Run migrations to rebuild
✅ Corrupted tenant? Clone from GitHub
✅ Need staging environment? Use migrations

### Audit Trail
✅ Decision logs document why choices were made
✅ Feature tracking shows what's enabled
✅ Migration files show all schema changes

### Client Management
✅ Each client has private GitHub repo
✅ Can give client access to their repo
✅ Full transparency on what was built

### Scalability
✅ Onboard Client 1: 4 weeks
✅ Onboard Client 2: 1 hour
✅ Onboard Client 100: 1 hour
✅ Deploy security fix: Once (all tenants get it)

## 🎯 Best Practices

**1. Always run /pp-setup first**
- Sets up session correctly
- Ensures other commands know where to work

**2. Push tenant repos to GitHub immediately**
- Backup is critical
- Can't lose configuration

**3. Document decisions**
- Use `docs/decisions.md` in tenant repo
- Explain WHY you made choices

**4. Test migrations**
- Always test SQL before committing
- Include rollback SQL in comments

**5. Link PRs**
- Platform PR should reference tenant issue
- Tenant PR should reference platform PR

**6. Use /pp-issue for all work**
- Full workflow automation
- Ensures nothing is forgotten

## 📁 File Locations

**Shared codebase:**
```
~/portal-work/portal-platform/
├── app/(tenants)/
│   ├── beyond-spreadsheets/  # Your website
│   ├── fitzone-gym/          # Client A
│   └── dkc/                  # Client B
├── components/shared/        # Shared components
└── lib/                      # Shared infrastructure
```

**Tenant configs:**
```
~/portal-work/tenants/
├── beyond-spreadsheets/
│   ├── migrations/
│   ├── docs/
│   ├── config/
│   └── backups/
├── fitzone-gym/
└── dkc/
```

**Session info:**
```
~/.claude-portal-session      # Full session config
~/.claude-portal-path          # Platform path (backwards compat)
```

## 🔐 Security

**Tenant Isolation:**
- RLS policies at database level
- Every query filters by `tenant_id`
- Each tenant has own folder
- Layout files enforce tenant check

**Git Repos:**
- Tenant repos are PRIVATE
- Never commit secrets (.env.local in .gitignore)
- Migrations don't contain credentials

**Access Control:**
- Only you have access to tenant repos
- Can grant client read access if needed
- Database credentials stay in portal-platform/.env.local

## 🎓 Learning Resources

- `/pp-help` - Main help guide
- `PP-COMMANDS-README.md` - Complete command documentation
- `CLAUDE.md` - Technical architecture
- This file - Workflow examples

---

**Ready to build your SaaS platform!** 🚀

Start with: `/pp-setup`
