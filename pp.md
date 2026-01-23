---
description: Portal Platform context reminder and quick reference
---

You are working with the **Portal Platform** - a multi-tenant SaaS architecture.

## 🎯 Current Session Context

Read the session file to understand what you're working on:

```bash
if [ -f ~/.claude-portal-session ]; then
  cat ~/.claude-portal-session
  echo ""
  echo "Session is active."
else
  echo "❌ No active session"
  echo "Run: /pp-setup to start"
  exit 1
fi
```

Show the user:
```
═══════════════════════════════════════════
   PORTAL PLATFORM - SESSION CONTEXT
═══════════════════════════════════════════

📂 Platform: {PORTAL_PLATFORM path}
📦 Tenant Repo: {TENANT_REPO path}
🏢 Working on: {TENANT_SLUG}

Current session is configured and ready.
```

## 🚨 CRITICAL RULES - READ BEFORE EVERY ACTION

### Rule 1: Code vs Data

**CODE changes affect location:**
- `lib/` = ⚠️  ALL TENANTS
- `components/shared/` = ⚠️  ALL TENANTS
- `app/api/` = ⚠️  ALL TENANTS
- `app/(tenants)/{slug}/` = ✅ ONE TENANT only

**DATA changes affect scope:**
- `portal.tenants` table = Could affect multiple tenants
- `tenant-{slug}/migrations/` = ✅ ONE TENANT only
- Database query with `tenant_id` filter = ✅ ONE TENANT only

### Rule 2: When to Use Shared Components

**✅ ONLY for truly generic, framework-level components:**
- Loading spinners
- Modal dialogs
- Buttons
- Form fields
- Card wrappers
- Toast notifications

**❌ NEVER for content or business logic:**
- Hero sections (each site is unique!)
- Pricing tables (each tenant has different pricing!)
- Navigation (each tenant has different structure!)
- Testimonials (different content per tenant!)
- Feature sections (unique per tenant!)

**Instead:** Create content inside tenant pages directly.

### Rule 3: Database Queries MUST Filter by tenant_id

**WRONG:**
```typescript
const { data } = await supabase
  .from('portal.bookings')
  .select('*')
// ❌ Returns ALL tenants' bookings!
```

**RIGHT:**
```typescript
const { data } = await supabase
  .from('portal.bookings')
  .select('*')
  .eq('tenant_id', tenant.id)
// ✅ Only this tenant's bookings
```

### Rule 4: Always Know Impact Before Making Changes

**Before ANY change, ask:**
> "Does this affect ALL tenants or ONLY {current_tenant}?"

**If ALL tenants:**
- Are you SURE it should?
- Have you tested with multiple tenants?
- Is it backwards compatible?
- Did you warn in commit message?

**If ONE tenant:**
- Are files ONLY in `app/(tenants)/{slug}/`?
- Did you verify with `git diff --name-only`?
- Is commit message labeled `feat({slug}):`?

### Rule 5: Migrations Go to Tenant Repo

**Platform schema changes:**
- Rare! Requires careful review
- Affects ALL tenants
- Goes to platform repo

**Tenant data changes:**
- Common! Safe
- Affects ONE tenant only
- Goes to `tenant-{slug}/migrations/`

## 📋 Quick Command Reference

| Command | Use When | Affects |
|---------|----------|---------|
| `/pp-setup` | Starting new work session | Session only |
| `/pp-tenant "Name"` | Creating new client | One tenant |
| `/pp-page {slug} home` | Creating tenant page | One tenant |
| `/pp-component Button` | Creating GENERIC framework component | All tenants ⚠️ |
| `/pp-list` | Listing all tenants | Read-only |
| `/pp-issue {slug} 42` | Working on issue | Ask: ALL or ONE? |

## 🎯 Common Scenarios

### Scenario: Client wants a custom homepage

**WRONG:**
```bash
/pp-component HeroSection
# ❌ All sites get same hero!
```

**RIGHT:**
```bash
/pp-page fitzone-gym home
# Then create unique hero inside that page
# ✅ Only FitZone gets this design
```

### Scenario: Need a loading spinner

**RIGHT:**
```bash
/pp-component LoadingSpinner
# ✅ Generic utility, all tenants can use
```

### Scenario: Fix a bug in auth

**Process:**
```bash
/pp-issue fitzone-gym 123
# At decision point, choose: ALL TENANTS
# ⚠️ This fixes auth for everyone
```

### Scenario: Add FitZone's pricing page

**RIGHT:**
```bash
/pp-page fitzone-gym pricing
# ✅ Unique pricing for FitZone only
```

## 📖 Full Documentation

For deeper understanding, read:
- **PLATFORM-VS-TENANT.md** - Decision flowchart and scenarios
- **WORKFLOW.md** - Complete workflows
- **CLAUDE.md** - Technical architecture

## 🔄 When to Run This Command

Run `/pp` when:
- Starting to work after a break (context refresh)
- About to make a change and unsure of impact
- Claude seems confused about architecture
- You need a quick reminder of the rules
- Session context seems lost

## ⚡ Quick Checks

**Before creating a component:**
```
Is this TRULY generic (Button, Modal, Spinner)?
→ YES: Use /pp-component
→ NO: Create inside tenant page
```

**Before making a code change:**
```
Where is the file?
→ Outside app/(tenants)/: ⚠️ Affects ALL tenants
→ Inside app/(tenants)/{slug}/: ✅ Affects ONE tenant
```

**Before database query:**
```
Did I include .eq('tenant_id', tenant.id)?
→ YES: ✅ Safe
→ NO: ❌ Will leak data across tenants!
```

---

**Remember: When in doubt, assume it affects ALL tenants and proceed with extreme caution!**

Better to be overly cautious than accidentally break 100 clients.
