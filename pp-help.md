---
description: Portal Platform - Multi-tenant SaaS commands help
---

# Portal Platform Commands - Full SaaS Mode

Commands for building and managing your **multi-tenant SaaS platform**.

## 🎯 Architecture Overview

**Your SaaS Platform Structure:**

```
portal-platform/
├── lib/               # SHARED: Auth, email, SMS, payments (all tenants)
├── middleware.ts      # SHARED: Security, routing (all tenants)
├── components/shared/ # SHARED: Reusable UI components
└── app/
    ├── (tenants)/     # UNIQUE: Each tenant's custom pages
    │   ├── beyond-spreadsheets/  # Your website
    │   ├── ayup/                  # Ayup website
    │   └── dkc/                   # DKC volunteer portal
    └── admin/         # SHARED: Admin dashboards (data differs per tenant)
```

**Key Principles:**
- ✅ Infrastructure = Shared (one codebase, push security to all)
- ✅ Data = Isolated (each tenant only sees their data)
- ✅ Design = Unique (each tenant has custom pages/styling)
- ✅ Features = Configurable (enable/disable per tenant)

## 📋 Available Commands

### `/pp-help`
Show this help guide.

---

### `/pp-tenant "<name>"`
**Create a new tenant in the database.**

**Usage:**
```bash
/pp-tenant "FitZone Gym"
/pp-tenant "Derby Kids Camp"
```

**What it does:**
- Creates record in `portal.tenants`
- Creates config in `portal.tenant_config`
- Sets up feature flags
- Creates admin role
- Creates tenant folder structure: `app/(tenants)/<slug>/`
- **Safe:** Only creates database records + empty folder

**Example output:**
```
✅ Tenant created: fitzone-gym
📂 Folder created: app/(tenants)/fitzone-gym/
📋 Database: portal.tenants
🔗 URL: https://fitzone-gym.yourdomain.com
```

---

### `/pp-page <tenant-slug> <page-name>`
**Create a tenant-specific page.**

**Usage:**
```bash
/pp-page beyond-spreadsheets home
/pp-page dkc volunteer-application
/pp-page ayup activities
```

**What it does:**
- Creates page in `app/(tenants)/<slug>/<page>/page.tsx`
- Includes automatic tenant detection
- Includes tenant_id filtering for data
- Uses shared components where possible
- **Safe:** Only affects that tenant's folder

**Example:**
```typescript
// Created: app/(tenants)/dkc/volunteer-application/page.tsx
export default async function VolunteerApplication() {
  const tenant = await getCurrentTenant() // Auto-detects DKC

  // Only DKC data is accessible
  const { data } = await supabase
    .from('portal.applications')
    .select('*')
    .eq('tenant_id', tenant.id) // Isolated!

  return <ApplicationForm tenant={tenant} />
}
```

---

### `/pp-component <name>`
**Create a shared component (used by all tenants).**

**Usage:**
```bash
/pp-component HeroSection
/pp-component BookingWidget
/pp-component DashboardCard
```

**What it does:**
- Creates component in `components/shared/<name>.tsx`
- Fully reusable across all tenants
- Accepts tenant config as props
- **Safe:** Shared code, but tenant-aware

**Example:**
```typescript
// Created: components/shared/HeroSection.tsx
export function HeroSection({
  title,
  subtitle,
  tenant
}: HeroProps) {
  return (
    <div style={{ backgroundColor: tenant.config.primary_color }}>
      <h1>{title}</h1>
      <p>{subtitle}</p>
    </div>
  )
}

// Used differently per tenant:
// DKC: <HeroSection title="Volunteer Today!" tenant={dkc} />
// Ayup: <HeroSection title="Activities for Kids" tenant={ayup} />
```

---

### `/pp-list`
**List all tenants and their pages.**

**Usage:**
```bash
/pp-list
```

**Shows:**
- All tenants in database
- Each tenant's custom pages
- Shared components
- Feature flags per tenant

---

## 🔒 Safety Guarantees

These commands ensure **tenant isolation**:

1. **Database Level:** Every query has `.eq('tenant_id', tenant.id)`
2. **Code Level:** Each tenant's pages in separate folders
3. **RLS Policies:** Database enforces tenant separation
4. **Automatic Detection:** Tenant identified from hostname

**You CANNOT accidentally:**
- ❌ Show Tenant A's data to Tenant B
- ❌ Break Tenant A while working on Tenant B
- ❌ Deploy changes that affect all tenants unintentionally

## 💡 Workflow: Build Your Website

**Step 1: Create your tenant**
```bash
/pp-tenant "Beyond Spreadsheets"
# Creates: beyond-spreadsheets tenant
# Folder: app/(tenants)/beyond-spreadsheets/
```

**Step 2: Create your pages**
```bash
/pp-page beyond-spreadsheets home
/pp-page beyond-spreadsheets about
/pp-page beyond-spreadsheets pricing
/pp-page beyond-spreadsheets contact
# ... create all 20 pages
```

**Step 3: Use shared components**
```bash
/pp-component CTAButton
/pp-component PricingCard
/pp-component TestimonialSlider
```

**Step 4: Deploy**
```bash
npm run build
vercel deploy --prod
# Your site + infrastructure live!
```

**Result:**
- Your website is built
- Infrastructure (auth, email, payments) ready for clients
- When you add Tenant B, they get all the infrastructure too!

## 💡 Workflow: Onboard a Client

**Step 1: Create their tenant**
```bash
/pp-tenant "FitZone Gym"
```

**Step 2: Create their pages**
```bash
/pp-page fitzone-gym home
/pp-page fitzone-gym classes
/pp-page fitzone-gym booking
```

**Step 3: They instantly have:**
- ✅ Auth system (your shared code)
- ✅ Email system (your shared code)
- ✅ Payment system (your shared code)
- ✅ Admin dashboard (your shared code)
- ✅ Custom pages (their unique design)

**Time:** Minutes, not weeks!

## 📚 Database Schema

**Tenant-Scoped Tables (data isolated per tenant):**
```
portal.tenants              - Tenant info
portal.tenant_config        - Branding, settings
portal.users               - Users (belongs to tenant)
portal.bookings            - Bookings (tenant_id filter)
portal.services            - Services (tenant_id filter)
portal.feature_flags       - Features enabled per tenant
```

**Shared Tables (same for all):**
```
auth.users                 - Supabase auth (shared)
```

## 🔧 Feature Flags

Control what each tenant sees:

```sql
-- Enable booking for FitZone
INSERT INTO portal.feature_flags (tenant_id, module_name, enabled)
VALUES ('fitzone-id', 'public_booking', true);

-- Disable booking for Ayup
UPDATE portal.feature_flags
SET enabled = false
WHERE tenant_id = 'ayup-id' AND module_name = 'public_booking';
```

## 🚀 Quick Start

**Build your own site first:**
```bash
1. /pp-tenant "Beyond Spreadsheets"
2. /pp-page beyond-spreadsheets home
3. /pp-page beyond-spreadsheets about
4. /pp-page beyond-spreadsheets pricing
5. npm run dev
6. Visit: http://localhost:3000
```

**Add your first client:**
```bash
1. /pp-tenant "FitZone Gym"
2. /pp-page fitzone-gym home
3. Edit /etc/hosts: 127.0.0.1 fitzone-gym.localhost
4. Visit: http://fitzone-gym.localhost:3000
```

## 📖 Resources

- **howToGuide.md** - Tenant setup workflow
- **CLAUDE.md** - Technical architecture
- **MULTI_TENANT_ROADMAP.md** - Implementation plan

---

**Ready to build your SaaS?** Start with `/pp-tenant "Your Business Name"`
