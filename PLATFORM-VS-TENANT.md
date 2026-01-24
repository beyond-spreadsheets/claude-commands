# Platform vs Tenant-Specific Changes

**Critical Guide: Understanding What Affects All Clients vs One Client**

## 🚨 The Most Important Question

**Before making ANY change, ask:**

> "Should this affect ALL {TOTAL_COUNT} tenants or ONLY {THIS_TENANT}?"

## 📊 Decision Flowchart

```
START: You need to make a change
       |
       v
   ┌───────────────────────────────────────┐
   │ What are you changing?                │
   └───────────────┬───────────────────────┘
                   |
        ┌──────────┴──────────┐
        |                     |
        v                     v
   Code Change          Database Change
        |                     |
        |                     v
        |          ┌──────────────────────────┐
        |          │ Schema or Data?           │
        |          └──────────┬────────────────┘
        |                     |
        |          ┌──────────┴──────────┐
        |          v                     v
        |      Schema Change         Data Change
        |      (new table,          (insert/update
        |       new column)          for ONE tenant)
        |          |                     |
        |          v                     v
        |     PLATFORM CHANGE       TENANT-SPECIFIC
        |     (affects ALL)         (affects ONE)
        |
        v
   ┌───────────────────────────────────────┐
   │ Where is the code?                    │
   └───────────────┬───────────────────────┘
                   |
        ┌──────────┴──────────┐
        |                     |
        v                     v
  Outside tenant folder   Inside tenant folder
  (lib/, components/,     (app/(tenants)/{slug}/)
   app/api/, etc.)              |
        |                       v
        v                  TENANT-SPECIFIC
   PLATFORM CHANGE         (affects ONE)
   (affects ALL)
```

## 🎯 Quick Reference

### ⚠️  PLATFORM CHANGES (Affect ALL Tenants)

**File Locations:**
```
✅ lib/                      # Shared utilities
✅ components/shared/        # Shared components
✅ app/api/                  # API routes
✅ middleware.ts             # Auth/routing
✅ app/layout.tsx            # Root layout
✅ Any file outside app/(tenants)/
```

**Examples:**
- Bug fix in authentication system
- New shared component (HeroSection, BookingWidget)
- Security patch
- Email template improvement
- Payment processing fix
- Database schema change (new table for all tenants)
- API route enhancement

**Commands:**
- `/pp-component HeroSection` ⚠️  Affects ALL
- `/pp-issue {slug} 42` → Choose "ALL TENANTS" ⚠️  Affects ALL

**Impact:**
```
⚠️  Changes deploy to ALL {TOTAL_COUNT} tenants:
- bs
- fitzone-gym
- dkc
- warriors-martial-arts
- derby-rfc
- ... and ALL future tenants
```

**Commit Message:**
```
feat(platform): Add booking widget component

⚠️  PLATFORM CHANGE - Affects ALL tenants

Changes:
- Added BookingWidget to components/shared/

Impact: All tenants can now use BookingWidget
```

### ✅ TENANT-SPECIFIC CHANGES (Affect ONE Tenant)

**File Locations:**
```
✅ app/(tenants)/{slug}/           # Tenant folder
✅ app/(tenants)/{slug}/**/*.tsx   # Tenant pages/components
✅ tenant-{slug}/migrations/       # Tenant data migrations
✅ tenant-{slug}/docs/             # Tenant documentation
✅ tenant-{slug}/config/           # Tenant configuration
```

**Examples:**
- Custom page for this client ("FitZone Class Schedule")
- Brand-specific styling
- Unique business logic for this tenant
- Custom navigation for this tenant
- Tenant data migration (add records for this tenant)

**Commands:**
- `/pp-page fitzone-gym home` ✅ Affects ONLY fitzone-gym
- `/pp-issue fitzone-gym 42` → Choose "ONLY THIS TENANT" ✅ Affects ONLY fitzone-gym

**Impact:**
```
✅ Changes affect ONLY: FitZone Gym
✅ Other tenants: NO IMPACT
✅ Files changed: app/(tenants)/fitzone-gym/** only
```

**Commit Message:**
```
feat(fitzone-gym): Add class schedule page

✅ TENANT-SPECIFIC - Only affects FitZone Gym

Changes:
- Added app/(tenants)/fitzone-gym/class-schedule/page.tsx

Impact: Only FitZone Gym affected
Other tenants: No changes
```

## 🔍 How to Decide

### Step 1: Ask These Questions

1. **Who requested this?**
   - One client? → Probably tenant-specific
   - Multiple clients? → Might be platform
   - Internal improvement? → Might be platform

2. **Will other tenants benefit?**
   - Yes → Platform change
   - No → Tenant-specific

3. **Is this a bug fix?**
   - Bug in shared code → Platform change (affects all)
   - Bug in one tenant's page → Tenant-specific

4. **Where will the files be?**
   - Outside `app/(tenants)/` → Platform change
   - Inside `app/(tenants)/{slug}/` → Tenant-specific

### Step 2: Use the `/pp-issue` Decision Point

When you run `/pp-issue fitzone-gym 42`, it will ask:

```
═══════════════════════════════════════════
   CRITICAL DECISION POINT
═══════════════════════════════════════════

This issue is for: FitZone Gym (fitzone-gym)

Question: Should this change affect ALL tenants or ONLY FitZone Gym?

Please choose:
1. ALL TENANTS - Platform code change
2. ONLY FitZone Gym - Tenant-specific change
3. DATABASE - Data migration

>
```

Choose carefully!

## 📋 Examples by Scenario

### Scenario 1: Client Requests New Feature

**Request:** "FitZone wants a class schedule page"

**Analysis:**
- Who: One client (FitZone)
- Benefit others: Maybe, but FitZone-specific design
- Files: `app/(tenants)/fitzone-gym/class-schedule/`

**Decision:** ✅ **TENANT-SPECIFIC**

**Command:**
```bash
/pp-page fitzone-gym class-schedule
```

---

### Scenario 2: Multiple Clients Want Same Thing

**Request:** "FitZone and Warriors both want booking widgets"

**Analysis:**
- Who: Multiple clients
- Benefit others: Yes! All gyms could use it
- Files: `components/shared/BookingWidget.tsx`

**Decision:** ⚠️  **PLATFORM CHANGE**

**Command:**
```bash
/pp-component BookingWidget
```

Then each tenant uses it:
```tsx
// app/(tenants)/fitzone-gym/book/page.tsx
<BookingWidget tenant={tenant} />

// app/(tenants)/warriors/book/page.tsx
<BookingWidget tenant={tenant} />
```

---

### Scenario 3: Bug in Auth System

**Request:** "Login is broken for FitZone"

**Analysis:**
- Who: Reported by one client
- Benefit others: YES! If auth is broken, affects everyone
- Files: `lib/auth/` or `app/api/auth/`

**Decision:** ⚠️  **PLATFORM CHANGE** (Bug fix)

**Process:**
```bash
/pp-issue fitzone-gym 123
> Choose: 1. ALL TENANTS - Platform code change
```

**Why:** Even though FitZone reported it, auth is shared. Fix helps everyone.

---

### Scenario 4: Custom Branding

**Request:** "FitZone wants their logo and colors"

**Analysis:**
- Who: One client
- Benefit others: No, FitZone-specific branding
- Files: Database config for FitZone

**Decision:** ✅ **TENANT-SPECIFIC** (Data only)

**Process:**
- Update via tenant config (database)
- Or migration in `tenant-fitzone-gym/migrations/`

---

### Scenario 5: Security Patch

**Request:** "Fix XSS vulnerability in comment system"

**Analysis:**
- Who: Internal security team
- Benefit others: YES! Security affects everyone
- Files: Wherever comment code is

**Decision:** ⚠️  **PLATFORM CHANGE** (Security)

**Priority:** HIGH - deploy to all tenants immediately

## ⚡ Command Impact Summary

| Command | Affects | Location | Warning |
|---------|---------|----------|---------|
| `/pp-setup` | Session only | N/A | ✅ Safe |
| `/pp-tenant` | One tenant | `app/(tenants)/{slug}/` + DB | ✅ Isolated |
| `/pp-page {slug} <page>` | One tenant | `app/(tenants)/{slug}/{page}/` | ✅ Isolated |
| `/pp-component <Name>` | **ALL tenants** | `components/shared/` | ⚠️  Platform |
| `/pp-list` | N/A (read-only) | N/A | ✅ Safe |
| `/pp-issue` → ALL | **ALL tenants** | `lib/`, `components/`, etc. | ⚠️  Platform |
| `/pp-issue` → ONLY | One tenant | `app/(tenants)/{slug}/` | ✅ Isolated |
| `/pp-issue` → DATABASE | One tenant | `tenant-{slug}/migrations/` | ✅ Isolated |

## 🛡️ Safety Checks

### Before Committing Platform Changes

```bash
# Check what files changed
git diff --name-only

# If you see:
lib/**                    → ⚠️  Platform change
components/shared/**      → ⚠️  Platform change
app/api/**               → ⚠️  Platform change
middleware.ts            → ⚠️  Platform change

# Ask yourself:
1. Did I INTEND to change platform code?
2. Will ALL tenants benefit from this?
3. Did I test with multiple tenants?
4. Is this backwards compatible?

# If NO to any: STOP and reconsider!
```

### Before Committing Tenant Changes

```bash
# Check what files changed
git diff --name-only

# Should ONLY see:
app/(tenants)/fitzone-gym/**  → ✅ Tenant-specific

# If you see anything else:
git diff --name-only | grep -v "app/(tenants)/fitzone-gym/"

# If output is NOT empty:
# ⚠️  WARNING: You're changing more than just the tenant folder!
```

## 📝 Commit Message Templates

### Platform Change
```
feat(platform): <what you changed>

⚠️  PLATFORM CHANGE - Affects ALL tenants

Changes:
- <list changes>

Impact: All {TOTAL_COUNT} tenants will receive this change
Origin: <which tenant requested it>
Risk: [Low/Medium/High]

Testing:
- Tested with [list tenants]
- Backwards compatible: [Yes/No]
```

### Tenant-Specific Change
```
feat(<slug>): <what you changed>

✅ TENANT-SPECIFIC - Only affects <Tenant Name>

Changes:
- <list changes>

Impact: Only <Tenant Name> affected
Other tenants: No changes

Files: app/(tenants)/<slug>/** only
```

## 🎓 Learning Guide

### Week 1: Understand the Architecture
- Read CLAUDE.md and WORKFLOW.md
- Understand `app/(tenants)/` vs `lib/` vs `components/shared/`
- Practice identifying platform vs tenant code

### Week 2: Make Tenant-Specific Changes
- Use `/pp-page` for a few tenants
- See how changes are isolated
- Gain confidence in tenant-specific work

### Week 3: Make Platform Changes Carefully
- Create shared components with `/pp-component`
- Understand impact on all tenants
- Test changes with multiple tenants

### Week 4: Use `/pp-issue` Workflow
- Work on real issues
- Practice the decision point
- See full workflow in action

## 🚨 Common Mistakes

### ❌ Mistake 1: Changing Platform When You Meant Tenant
```bash
# WRONG: Changed lib/auth.ts for FitZone-specific auth
# Impact: Broke auth for ALL tenants!

# RIGHT: Created app/(tenants)/fitzone-gym/custom-auth.tsx
# Impact: Only FitZone affected
```

### ❌ Mistake 2: Duplicating Code Instead of Sharing
```bash
# WRONG: Copied HeroSection into each tenant folder
# Problem: Now have to fix bugs in 10 places!

# RIGHT: Created components/shared/HeroSection.tsx
# Benefit: Fix once, fixes for everyone!
```

### ❌ Mistake 3: Not Verifying File Changes
```bash
# WRONG: Committed without checking
git commit -am "fix for fitzone"

# RIGHT: Checked first
git diff --name-only
# Saw: app/(tenants)/fitzone-gym/** only ✅
git commit -am "feat(fitzone-gym): fixed issue"
```

## 🎯 Key Takeaways

1. **Always ask first:** "ALL tenants or ONLY this tenant?"
2. **Check file locations:** Outside `app/(tenants)/` = platform change
3. **Use the commands correctly:**
   - `/pp-page` = tenant-specific (safe)
   - `/pp-component` = platform (affects all)
   - `/pp-issue` = asks you to decide
4. **Verify before committing:** `git diff --name-only`
5. **Clear commit messages:** Label platform vs tenant-specific

---

**Remember: When in doubt, assume it affects ALL tenants and proceed with caution!**

Better to be overly careful with platform changes than accidentally break 100 clients.
