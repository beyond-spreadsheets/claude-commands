---
description: Create new tenant in multi-tenant SaaS (database + folder structure)
autoApprove:
  - Read
  - Glob
  - Write
  - Bash(node scripts/*)
  - Bash(mkdir *)
---

You are creating a NEW TENANT for the portal-platform multi-tenant SaaS.

## Command Format

`/pp-tenant "<tenant-name>"`

Example: `/pp-tenant "FitZone Gym"`

## What This Command Does

Creates:
1. ✅ Tenant record in database (`portal.tenants`)
2. ✅ Tenant configuration (`portal.tenant_config`)
3. ✅ Feature flags (`portal.feature_flags`)
4. ✅ Admin role (`portal.roles`)
5. ✅ Folder structure (`app/(tenants)/<slug>/`)
6. ✅ Basic layout file with tenant detection

**Does NOT create:** Any pages yet - use `/pp-page` for that

## Step 1: Gather Information

Ask the user for ALL required information:

### Required:
1. **Business Name** - Full name (e.g., "FitZone Gym")
2. **Slug** - URL-friendly (e.g., "fitzone-gym") - suggest based on name
3. **Subdomain** - Alphanumeric only (e.g., "fitzone")

### Optional (can add later):
4. **Custom Domain** - e.g., "fitzone.com"
5. **Logo URL** - Public URL to logo
6. **Primary Color** - Hex code (default: #0891b2)
7. **Secondary Color** - Hex code (default: #06b6d4)
8. **Support Email** - e.g., "support@fitzone.com"

## Step 2: Validate Input

```javascript
// Validate slug format
const slugRegex = /^[a-z0-9]+(?:-[a-z0-9]+)*$/
if (!slugRegex.test(slug)) {
  throw new Error('Slug must be lowercase with hyphens only')
}

// Validate subdomain format
const subdomainRegex = /^[a-z0-9]+$/
if (!subdomainRegex.test(subdomain)) {
  throw new Error('Subdomain must be alphanumeric only')
}
```

## Step 3: Check for Conflicts

Create script to check if tenant already exists:

```javascript
// scripts/check-tenant-<slug>.mjs
#!/usr/bin/env node
import { createClient } from '@supabase/supabase-js'
import * as dotenv from 'dotenv'

dotenv.config({ path: '.env.local' })

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY
)

async function checkConflicts() {
  // Check slug
  const { data: slugExists } = await supabase
    .from('portal.tenants')
    .select('id, name')
    .eq('slug', 'SLUG')
    .single()

  if (slugExists) {
    console.error('❌ Slug already exists:', slugExists.name)
    process.exit(1)
  }

  // Check subdomain
  const { data: subdomainExists } = await supabase
    .from('portal.tenants')
    .select('id, name')
    .eq('subdomain', 'SUBDOMAIN')
    .single()

  if (subdomainExists) {
    console.error('❌ Subdomain already exists:', subdomainExists.name)
    process.exit(1)
  }

  console.log('✅ No conflicts - safe to create')
}

checkConflicts().then(() => process.exit(0))
```

Run it, replacing SLUG and SUBDOMAIN.

## Step 4: Create Tenant in Database

Create the tenant creation script:

```javascript
// scripts/create-tenant-<slug>.mjs
#!/usr/bin/env node
import { createClient } from '@supabase/supabase-js'
import * as dotenv from 'dotenv'

dotenv.config({ path: '.env.local' })

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY
)

const TENANT_DATA = {
  name: 'TENANT_NAME',
  slug: 'SLUG',
  subdomain: 'SUBDOMAIN',
  custom_domain: 'CUSTOM_DOMAIN' || null
}

const CONFIG_DATA = {
  logo_url: 'LOGO_URL' || null,
  primary_color: 'PRIMARY_COLOR' || '#0891b2',
  secondary_color: 'SECONDARY_COLOR' || '#06b6d4',
  email_from: `noreply@SUBDOMAIN.com`,
  support_email: 'SUPPORT_EMAIL' || null,
  require_login_for_booking: false,
  allow_public_registration: true,
  enable_role_based_access: false,
  show_login_on_homepage: false,
  homepage_layout: 'public_booking',
  is_public: true
}

const DEFAULT_FEATURES = [
  'public_booking',
  'customer_portal',
  'documents',
  'payments',
  'reports'
]

async function createTenant() {
  console.log('🚀 Creating tenant:', TENANT_DATA.name)

  // 1. Create tenant
  const { data: tenant, error: tenantError } = await supabase
    .from('portal.tenants')
    .insert(TENANT_DATA)
    .select()
    .single()

  if (tenantError) {
    console.error('❌ Error creating tenant:', tenantError.message)
    process.exit(1)
  }

  console.log('✅ Tenant created')
  console.log('   ID:', tenant.id)
  console.log('   Slug:', tenant.slug)

  // 2. Create config
  const { error: configError } = await supabase
    .from('portal.tenant_config')
    .insert({
      tenant_id: tenant.id,
      ...CONFIG_DATA
    })

  if (configError) {
    console.error('❌ Error creating config:', configError.message)
    // Rollback
    await supabase.from('portal.tenants').delete().eq('id', tenant.id)
    process.exit(1)
  }

  console.log('✅ Config created')

  // 3. Enable features
  const features = DEFAULT_FEATURES.map(name => ({
    tenant_id: tenant.id,
    module_name: name,
    enabled: true,
    activated_at: new Date().toISOString()
  }))

  const { error: featuresError } = await supabase
    .from('portal.feature_flags')
    .insert(features)

  if (featuresError) {
    console.error('⚠️  Warning: Could not enable features:', featuresError.message)
  } else {
    console.log('✅ Features enabled:', DEFAULT_FEATURES.join(', '))
  }

  // 4. Create admin role
  const { data: role, error: roleError } = await supabase
    .from('portal.roles')
    .insert({
      tenant_id: tenant.id,
      name: 'Admin',
      description: 'Full administrative access',
      permissions: [
        'admin.full_access',
        'users.read',
        'users.create',
        'users.update',
        'users.delete',
        'roles.read',
        'bookings.read',
        'bookings.create',
        'bookings.update',
        'bookings.delete'
      ]
    })
    .select()
    .single()

  if (roleError) {
    console.error('⚠️  Warning: Could not create admin role:', roleError.message)
  } else {
    console.log('✅ Admin role created')
  }

  console.log('\n🎉 Tenant created successfully!\n')
  console.log('Details:')
  console.log('  Name:', tenant.name)
  console.log('  Slug:', tenant.slug)
  console.log('  Subdomain:', `https://${tenant.subdomain}.yourdomain.com`)
  if (tenant.custom_domain) {
    console.log('  Custom Domain:', `https://${tenant.custom_domain}`)
  }
  console.log('  Tenant ID:', tenant.id)

  return tenant
}

createTenant()
  .then(() => process.exit(0))
  .catch(err => {
    console.error('Fatal error:', err)
    process.exit(1)
  })
```

Replace placeholders and run: `node scripts/create-tenant-<slug>.mjs`

## Step 4b: Save Migration to Tenant Repo

After successfully creating the tenant, save the migration for version control:

```bash
# Load session info
source ~/.claude-portal-session

# Create migration file with timestamp
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
MIGRATION_FILE="$TENANT_REPO/migrations/${TIMESTAMP}_initial_tenant_setup.sql"

cat > "$MIGRATION_FILE" << 'EOF'
-- Migration: Initial Tenant Setup
-- Date: {DATE}
-- Tenant: {TENANT_NAME}
-- Slug: {SLUG}

-- This migration creates the tenant and initial configuration
-- Generated by /pp-tenant command

-- 1. Tenant Record
INSERT INTO portal.tenants (name, slug, subdomain, custom_domain)
VALUES (
  '{TENANT_NAME}',
  '{SLUG}',
  '{SUBDOMAIN}',
  {CUSTOM_DOMAIN_OR_NULL}
)
RETURNING id;

-- Store tenant_id for subsequent queries
-- (In practice, you'll need to replace {TENANT_ID} with actual UUID after creation)

-- 2. Tenant Configuration
INSERT INTO portal.tenant_config (
  tenant_id,
  logo_url,
  primary_color,
  secondary_color,
  email_from,
  support_email,
  require_login_for_booking,
  allow_public_registration,
  enable_role_based_access,
  show_login_on_homepage,
  homepage_layout,
  is_public
) VALUES (
  '{TENANT_ID}',
  {LOGO_URL_OR_NULL},
  '{PRIMARY_COLOR}',
  '{SECONDARY_COLOR}',
  'noreply@{SUBDOMAIN}.com',
  {SUPPORT_EMAIL_OR_NULL},
  false,
  true,
  false,
  false,
  'public_booking',
  true
);

-- 3. Feature Flags
INSERT INTO portal.feature_flags (tenant_id, module_name, enabled, activated_at)
VALUES
  ('{TENANT_ID}', 'public_booking', true, NOW()),
  ('{TENANT_ID}', 'customer_portal', true, NOW()),
  ('{TENANT_ID}', 'documents', true, NOW()),
  ('{TENANT_ID}', 'payments', true, NOW()),
  ('{TENANT_ID}', 'reports', true, NOW());

-- 4. Admin Role
INSERT INTO portal.roles (tenant_id, name, description, permissions)
VALUES (
  '{TENANT_ID}',
  'Admin',
  'Full administrative access',
  '["admin.full_access","users.read","users.create","users.update","users.delete","roles.read","bookings.read","bookings.create","bookings.update","bookings.delete"]'::jsonb
);

-- Verification Queries
-- SELECT * FROM portal.tenants WHERE slug = '{SLUG}';
-- SELECT * FROM portal.tenant_config WHERE tenant_id = '{TENANT_ID}';
-- SELECT * FROM portal.feature_flags WHERE tenant_id = '{TENANT_ID}';
-- SELECT * FROM portal.roles WHERE tenant_id = '{TENANT_ID}';
EOF

# Commit to tenant repo
cd "$TENANT_REPO"
git add migrations/
git commit -m "feat: Add initial tenant setup migration

Tenant: {TENANT_NAME}
Slug: {SLUG}
Features: public_booking, customer_portal, documents, payments, reports"

echo "✅ Migration saved: $MIGRATION_FILE"
echo "✅ Committed to tenant repo"
```

**Update tenant.json with actual values:**

```bash
cd "$TENANT_REPO"

# Update config/tenant.json with actual tenant_id and details
cat > config/tenant.json << 'EOF'
{
  "name": "{TENANT_NAME}",
  "slug": "{SLUG}",
  "tenant_id": "{TENANT_ID}",
  "created_at": "{TIMESTAMP}",
  "platform_path": "~/portal-work/portal-platform",
  "database": {
    "schema": "portal",
    "tenant_table": "portal.tenants"
  },
  "features": [
    "public_booking",
    "customer_portal",
    "documents",
    "payments",
    "reports"
  ],
  "domains": {
    "subdomain": "{SUBDOMAIN}",
    "custom_domain": {CUSTOM_DOMAIN_OR_NULL}
  },
  "branding": {
    "primary_color": "{PRIMARY_COLOR}",
    "secondary_color": "{SECONDARY_COLOR}",
    "logo_url": {LOGO_URL_OR_NULL}
  }
}
EOF

git add config/tenant.json
git commit -m "feat: Update tenant config with actual values"
```

**Add documentation:**

```bash
# Update decision log
cat >> docs/decisions.md << 'EOF'

### {DATE} - Tenant Created
- Tenant ID: {TENANT_ID}
- Initial features: public_booking, customer_portal, documents, payments, reports
- Admin role created
- Migration: migrations/{TIMESTAMP}_initial_tenant_setup.sql

EOF

# Update features log
cat > docs/features.md << 'EOF'
# Feature Configuration

## Enabled Features

### public_booking
- Enabled: Yes
- Activated: {DATE}
- Config: Default

### customer_portal
- Enabled: Yes
- Activated: {DATE}
- Config: Default

### documents
- Enabled: Yes
- Activated: {DATE}
- Config: Default

### payments
- Enabled: Yes
- Activated: {DATE}
- Config: Default

### reports
- Enabled: Yes
- Activated: {DATE}
- Config: Default

## Feature History

- {DATE}: Initial features enabled

EOF

git add docs/
git commit -m "docs: Document initial features and decisions"

echo "✅ Documentation updated"
```

## Step 5: Create Folder Structure

Create the tenant's folder:

```bash
cd /Users/George/Desktop/portal-platform

# Create tenant folder
mkdir -p app/(tenants)/SLUG

echo "✅ Created folder: app/(tenants)/SLUG/"
```

## Step 6: Create Basic Layout File

Create the layout file with tenant detection:

```typescript
// app/(tenants)/SLUG/layout.tsx
import { headers } from 'next/headers'
import { getTenantFromHostname } from '@/lib/tenant/context'
import { redirect } from 'next/navigation'

export default async function TenantLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const headersList = await headers()
  const hostname = headersList.get('host') || 'localhost:3000'
  const tenant = await getTenantFromHostname(hostname)

  // Security: Ensure this is the correct tenant
  if (!tenant || tenant.slug !== 'SLUG') {
    redirect('/')
  }

  return (
    <html lang="en">
      <head>
        <style dangerouslySetInnerHTML={{
          __html: `
            :root {
              --color-primary: ${tenant.config?.primary_color || '#0891b2'};
              --color-secondary: ${tenant.config?.secondary_color || '#06b6d4'};
            }
          `
        }} />
      </head>
      <body>
        <div className="min-h-screen">
          {children}
        </div>
      </body>
    </html>
  )
}
```

Replace SLUG with actual slug.

## Step 7: Create Placeholder Home Page

```typescript
// app/(tenants)/SLUG/page.tsx
import { getTenantFromHostname } from '@/lib/tenant/context'
import { headers } from 'next/headers'

export default async function TenantHome() {
  const headersList = await headers()
  const hostname = headersList.get('host') || 'localhost:3000'
  const tenant = await getTenantFromHostname(hostname)

  return (
    <div className="flex items-center justify-center min-h-screen bg-gradient-to-br from-primary to-secondary">
      <div className="text-center text-white">
        <h1 className="text-6xl font-bold mb-4">
          {tenant?.name}
        </h1>
        <p className="text-2xl mb-8">
          Welcome to your portal platform
        </p>
        <p className="text-lg opacity-80">
          Create pages with: <code className="bg-white/20 px-2 py-1 rounded">/pp-page {tenant?.slug} &lt;page-name&gt;</code>
        </p>
      </div>
    </div>
  )
}
```

## Step 8: Summary

Tell the user:

```
✅ Tenant Created: TENANT_NAME

📂 Platform Files Created:
  - app/(tenants)/SLUG/
  - app/(tenants)/SLUG/layout.tsx
  - app/(tenants)/SLUG/page.tsx

📦 Tenant Repository:
  - ~/portal-work/tenants/SLUG/
  - migrations/{TIMESTAMP}_initial_tenant_setup.sql
  - config/tenant.json
  - docs/decisions.md
  - docs/features.md

📋 Database:
  - portal.tenants (1 record)
  - portal.tenant_config (1 record)
  - portal.feature_flags (5 records)
  - portal.roles (1 record)

🔗 URLs:
  - Development: http://localhost:3000 (when hostname matches)
  - Subdomain: https://SUBDOMAIN.yourdomain.com
  - Custom: https://CUSTOM_DOMAIN (if configured)

📝 Next Steps:
  1. Push tenant repo to GitHub:
     cd ~/portal-work/tenants/SLUG
     gh repo create beyond-spreadsheets/tenant-SLUG --private
     git remote add origin git@github.com:beyond-spreadsheets/tenant-SLUG.git
     git push -u origin main

  2. Test locally: Update /etc/hosts
     127.0.0.1 SUBDOMAIN.localhost

  3. Create pages:
     /pp-page SLUG home
     /pp-page SLUG about
     /pp-page SLUG contact

  4. Deploy to Vercel:
     - Add domain: SUBDOMAIN.yourdomain.com
     - Configure DNS (if custom domain)

  5. Create admin user (have client sign up)

🎯 Ready to build pages!

💾 All tenant configuration is version controlled in ~/portal-work/tenants/SLUG/
   - Migrations tracked in Git
   - Documentation auto-generated
   - Easy to rebuild or backup
```

## Validation Checklist

- [ ] Tenant name provided
- [ ] Slug validated (lowercase, hyphens)
- [ ] Subdomain validated (alphanumeric)
- [ ] No conflicts in database
- [ ] Tenant created in database
- [ ] Config created
- [ ] Features enabled
- [ ] Admin role created
- [ ] Folder created: `app/(tenants)/<slug>/`
- [ ] layout.tsx created with tenant detection
- [ ] page.tsx created as placeholder

## Safety Guarantees

✅ **Tenant Isolation:**
- Each tenant has own folder
- Layout enforces tenant check
- Cannot access other tenant's pages

✅ **Database Isolation:**
- RLS policies enforce tenant_id filtering
- Each tenant sees only their data

✅ **Safe to Run:**
- Only creates files in that tenant's folder
- Only creates database records for that tenant
- Cannot affect other tenants

Remember: This creates the STRUCTURE. Use `/pp-page` to create actual pages!
