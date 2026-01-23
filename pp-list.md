---
description: List all tenants, pages, and components in the SaaS platform
autoApprove:
  - Read
  - Glob
  - Bash(find *)
  - Bash(ls *)
  - Bash(node scripts/*)
---

You are listing the complete overview of the multi-tenant SaaS platform.

## What This Shows

1. ✅ All tenants in the database
2. ✅ Each tenant's custom pages
3. ✅ Shared components
4. ✅ Feature flags per tenant
5. ✅ Platform statistics

## Step 1: List Tenants from Database

Create script to fetch all tenants:

```javascript
// scripts/list-all-tenants.mjs
#!/usr/bin/env node
import { createClient } from '@supabase/supabase-js'
import * as dotenv from 'dotenv'

dotenv.config({ path: '.env.local' })

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY
)

async function listTenants() {
  const { data: tenants, error } = await supabase
    .from('portal.tenants')
    .select(`
      id,
      name,
      slug,
      subdomain,
      custom_domain,
      created_at,
      config:tenant_config(
        logo_url,
        primary_color,
        secondary_color,
        is_public
      )
    `)
    .order('created_at', { ascending: false })

  if (error) {
    console.error('❌ Error fetching tenants:', error.message)
    process.exit(1)
  }

  return tenants
}

async function getTenantFeatures(tenantId) {
  const { data: features } = await supabase
    .from('portal.feature_flags')
    .select('module_name, enabled')
    .eq('tenant_id', tenantId)
    .eq('enabled', true)

  return features?.map(f => f.module_name) || []
}

async function main() {
  const tenants = await listTenants()

  console.log('\n═══════════════════════════════════════════')
  console.log('   PORTAL PLATFORM - TENANT OVERVIEW')
  console.log('═══════════════════════════════════════════\n')

  console.log(`📋 Found ${tenants.length} tenant(s)\n`)

  for (const [index, tenant] of tenants.entries()) {
    console.log(`${index + 1}. ${tenant.name}`)
    console.log(`   Slug: ${tenant.slug}`)
    console.log(`   Subdomain: https://${tenant.subdomain}.yourdomain.com`)

    if (tenant.custom_domain) {
      console.log(`   Custom Domain: https://${tenant.custom_domain}`)
    }

    console.log(`   Tenant ID: ${tenant.id}`)
    console.log(`   Public: ${tenant.config?.is_public ? 'Yes' : 'No'}`)
    console.log(`   Logo: ${tenant.config?.logo_url ? '✅' : '⏳ Pending'}`)
    console.log(`   Colors: ${tenant.config?.primary_color} / ${tenant.config?.secondary_color}`)
    console.log(`   Created: ${new Date(tenant.created_at).toLocaleDateString()}`)

    // Get features
    const features = await getTenantFeatures(tenant.id)
    if (features.length > 0) {
      console.log(`   Features: ${features.join(', ')}`)
    }

    console.log('')
  }

  return tenants
}

main()
  .then(() => process.exit(0))
  .catch(err => {
    console.error('Fatal error:', err)
    process.exit(1)
  })
```

Run it: `node scripts/list-all-tenants.mjs`

## Step 2: List Tenant Pages

Find all tenant-specific pages:

```bash
# List all tenant folders
find /Users/George/Desktop/portal-platform/app/\(tenants\) -mindepth 1 -maxdepth 1 -type d 2>/dev/null | sort
```

For each tenant folder, list pages:

```bash
for tenant_dir in /Users/George/Desktop/portal-platform/app/\(tenants\)/*/; do
  tenant_name=$(basename "$tenant_dir")
  echo "\n📂 Tenant: $tenant_name"

  # Find all page.tsx files
  find "$tenant_dir" -name "page.tsx" -type f | while read page_file; do
    # Get relative path from tenant folder
    rel_path=$(echo "$page_file" | sed "s|$tenant_dir||")
    page_name=$(dirname "$rel_path")

    if [ "$page_name" = "." ]; then
      echo "   • / (home)"
    else
      echo "   • /$page_name"
    fi
  done
done
```

## Step 3: List Shared Components

```bash
# List all shared components
echo "\n🧩 SHARED COMPONENTS\n"

find /Users/George/Desktop/portal-platform/components/shared -name "*.tsx" -type f ! -name "*.example.tsx" ! -name "*.test.tsx" | sort | while read component; do
  component_name=$(basename "$component" .tsx)
  echo "   • $component_name"
done
```

## Step 4: Show Statistics

```javascript
// Add to list-all-tenants.mjs

async function getStatistics(tenants) {
  const stats = {
    totalTenants: tenants.length,
    withCustomDomain: tenants.filter(t => t.custom_domain).length,
    withLogo: tenants.filter(t => t.config?.logo_url).length,
    publicTenants: tenants.filter(t => t.config?.is_public).length
  }

  console.log('═══════════════════════════════════════════')
  console.log('   STATISTICS')
  console.log('═══════════════════════════════════════════\n')

  console.log(`📊 Tenants: ${stats.totalTenants}`)
  console.log(`🌐 With Custom Domain: ${stats.withCustomDomain}`)
  console.log(`🎨 With Logo: ${stats.withLogo}`)
  console.log(`👁️  Public Access: ${stats.publicTenants}`)

  return stats
}

// Call in main():
await getStatistics(tenants)
```

## Step 5: Complete Output Format

```
═══════════════════════════════════════════════════════
      PORTAL PLATFORM - FULL OVERVIEW
═══════════════════════════════════════════════════════

📋 TENANTS (3 found)

1. Beyond Spreadsheets
   Slug: beyond-spreadsheets
   Subdomain: https://beyond-spreadsheets.yourdomain.com
   Custom Domain: https://beyond-spreadsheets.co.uk
   Tenant ID: abc-123-def
   Public: Yes | Logo: ✅ | Colors: #0891b2 / #06b6d4
   Created: Jan 15, 2026
   Features: public_booking, customer_portal, payments

   📄 Pages:
      • / (home)
      • /about
      • /pricing
      • /contact
      • /blog

2. Derby Kids Camp
   Slug: dkc
   Subdomain: https://dkc.yourdomain.com
   Tenant ID: xyz-789-ghi
   Public: Yes | Logo: ✅ | Colors: #ff6600 / #ffcc00
   Created: Jan 18, 2026
   Features: public_booking, documents, references, weeks

   📄 Pages:
      • / (home)
      • /volunteer-application
      • /weeks
      • /references
      • /training

3. Ayup Kids Activities
   Slug: ayup
   Subdomain: https://ayup.yourdomain.com
   Tenant ID: lmn-456-opq
   Public: Yes | Logo: ⏳ | Colors: #10b981 / #34d399
   Created: Jan 20, 2026
   Features: public_booking, customer_portal

   📄 Pages:
      • / (home)
      • /activities
      • /book
      • /about

─────────────────────────────────────────────────────

🧩 SHARED COMPONENTS (8 found)

   • HeroSection
   • Navigation
   • Footer
   • BookingWidget
   • PricingCard
   • Testimonial
   • DashboardWidget
   • CTAButton

─────────────────────────────────────────────────────

🏗️  INFRASTRUCTURE (Shared by all tenants)

   ✅ Authentication (Supabase)
   ✅ Email Service (Resend)
   ✅ SMS Service (Twilio)
   ✅ Payment Processing (Stripe)
   ✅ File Storage (Google Drive)
   ✅ Database (PostgreSQL/Supabase)
   ✅ Hosting (Vercel)

─────────────────────────────────────────────────────

📊 STATISTICS

Tenants: 3
With Custom Domain: 1
With Logo: 2
Public Access: 3
Shared Components: 8
Total Pages: 13

Avg Pages per Tenant: 4.3
Infrastructure Updates: Push to all tenants instantly ✅

─────────────────────────────────────────────────────

💡 QUICK ACTIONS

Create tenant:         /pp-tenant "Business Name"
Create page:           /pp-page <slug> <page-name>
Create component:      /pp-component <ComponentName>
Show this overview:    /pp-list
Get help:              /pp-help

─────────────────────────────────────────────────────

🔗 RESOURCES

Supabase: https://supabase.com/dashboard/project/[ID]
Vercel:   https://vercel.com/dashboard
Docs:     /Users/George/Desktop/portal-platform/howToGuide.md

═══════════════════════════════════════════════════════
```

## Step 6: Show Health Status

Optional: Add health checks:

```javascript
async function checkHealth() {
  console.log('\n🏥 HEALTH CHECK\n')

  // Check database connection
  try {
    const { error } = await supabase.from('portal.tenants').select('count').single()
    console.log('   Database: ✅ Connected')
  } catch (err) {
    console.log('   Database: ❌ Error')
  }

  // Check for tenants without pages
  const tenantsWithoutPages = [] // implement logic
  if (tenantsWithoutPages.length > 0) {
    console.log(`   ⚠️  ${tenantsWithoutPages.length} tenant(s) have no pages yet`)
  }

  // Check for tenants without logos
  const tenantsWithoutLogos = tenants.filter(t => !t.config?.logo_url)
  if (tenantsWithoutLogos.length > 0) {
    console.log(`   ⚠️  ${tenantsWithoutLogos.length} tenant(s) missing logo`)
  }
}
```

## Step 7: Filter Options

If user wants to filter:

```
Would you like to filter the results?

1. Show only tenants with pages
2. Show only tenants with custom domains
3. Show only public tenants
4. Show specific tenant details
5. Show everything (default)
```

Apply filters based on choice.

## Error Handling

**If no tenants found:**
```
📋 No tenants found in the database.

Create your first tenant:
  /pp-tenant "Your Business Name"

Then create pages:
  /pp-page <slug> <page-name>
```

**If database connection fails:**
```
❌ Cannot connect to database

Check:
1. .env.local has correct SUPABASE credentials
2. Supabase project is active
3. Network connection

Run: cat .env.local | grep SUPABASE
```

## Summary Output

Always end with actionable next steps:

```
✅ Platform Overview Complete

📊 Summary:
   • X tenants configured
   • Y pages built
   • Z shared components
   • Infrastructure: Shared ✅

🚀 Next Steps:
   1. Missing pages? Use /pp-page <slug> <page>
   2. Need components? Use /pp-component <Name>
   3. New tenant? Use /pp-tenant "Name"

📚 Help: /pp-help
```

## Validation Checklist

- [ ] Database connection successful
- [ ] All tenants listed
- [ ] Pages counted per tenant
- [ ] Shared components listed
- [ ] Statistics calculated
- [ ] Health status checked
- [ ] Actionable next steps provided

## Quick Filters

**Show specific tenant:**
```bash
node scripts/list-all-tenants.mjs | grep -A 10 "dkc"
```

**Count pages per tenant:**
```bash
find app/\(tenants\)/*/  -name "page.tsx" | wc -l
```

**List tenants alphabetically:**
```bash
ls app/\(tenants\)/ | sort
```

This gives a complete bird's-eye view of your entire multi-tenant SaaS platform!
