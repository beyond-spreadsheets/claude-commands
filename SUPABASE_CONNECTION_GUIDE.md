# Supabase Connection Guide - Step-by-Step

**Last Updated:** January 17, 2026
**Project:** portal-platform
**Database:** Shared with Beyond Spreadsheets website

---

## Quick Reference

**Supabase Project Details:**
- **Project Ref:** `uusjsarpkhonjpglxzgn`
- **URL:** `https://uusjsarpkhonjpglxzgn.supabase.co`
- **Dashboard:** https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn
- **SQL Editor:** https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn/sql/new
- **Schema:** `portal` (isolated from main website's `public` schema)

---

## Method 1: Node.js Scripts (RECOMMENDED) ✅

**Best for:** Creating/updating data, running queries, checking database state

### Step-by-Step Instructions:

#### Step 1: Navigate to Project
```bash
cd ~/Desktop/portal-platform
```

#### Step 2: Verify Environment Variables Exist
```bash
# Check that .env.local has Supabase credentials
grep "SUPABASE" .env.local
```

**You should see:**
```bash
NEXT_PUBLIC_SUPABASE_URL=https://uusjsarpkhonjpglxzgn.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

If missing, add them from the Supabase dashboard.

#### Step 3: Create a Script

**Example: Query Tenants**

```bash
cat > scripts/query-tenants.js << 'EOF'
#!/usr/bin/env node

const { createClient } = require('@supabase/supabase-js')
require('dotenv').config({ path: '.env.local' })

// Create Supabase client with service role key
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY
)

async function queryTenants() {
  console.log('🔍 Querying all tenants...\n')

  // Query the database
  const { data, error } = await supabase
    .from('tenants')
    .select('id, name, slug, custom_domain')
    .order('created_at', { ascending: true })

  if (error) {
    console.error('❌ Error:', error)
    process.exit(1)
  }

  console.log(`Found ${data.length} tenant(s):\n`)
  data.forEach((t, i) => {
    console.log(`${i + 1}. ${t.name}`)
    console.log(`   Domain: ${t.custom_domain || 'Not set'}`)
    console.log(`   Slug: ${t.slug}`)
    console.log('')
  })
}

queryTenants().then(() => process.exit(0))
EOF
```

#### Step 4: Run the Script
```bash
node scripts/query-tenants.js
```

---

### Common Script Templates:

#### A. Query Data
```javascript
const { data, error } = await supabase
  .from('tenants')
  .select('*')
  .eq('slug', 'bs')
  .single()
```

#### B. Insert Data
```javascript
const { data, error } = await supabase
  .from('tenants')
  .insert({
    name: 'New Customer',
    slug: 'new-customer',
    custom_domain: 'portal.newcustomer.com'
  })
  .select()
```

#### C. Update Data
```javascript
const { data, error } = await supabase
  .from('tenants')
  .update({ custom_domain: 'new-domain.com' })
  .eq('id', 'tenant-uuid')
  .select()
```

#### D. Delete Data
```javascript
const { data, error } = await supabase
  .from('tenants')
  .delete()
  .eq('id', 'tenant-uuid')
```

---

### Available Scripts (Already Created):

```bash
# Verify Beyond Spreadsheets tenant exists
node scripts/verify-bs-tenant.js

# Check for duplicate tenants
node scripts/check-duplicate-tenants.js

# Check tenant config
node scripts/check-tenant-config.js

# Create/verify Beyond Spreadsheets tenant
node scripts/create-bs-tenant.js
```

---

## Method 2: Supabase SQL Editor (For Schema Changes)

**Best for:** Creating tables, running migrations, complex SQL queries

### Step-by-Step Instructions:

#### Step 1: Open SQL Editor
Visit: https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn/sql/new

#### Step 2: Write Your SQL
```sql
-- Example: Query tenants
SELECT
  id,
  name,
  slug,
  custom_domain,
  created_at
FROM portal.tenants
ORDER BY created_at DESC;
```

**IMPORTANT:** Use `portal.` schema prefix:
- ✅ `SELECT * FROM portal.tenants`
- ❌ `SELECT * FROM tenants` (searches `public` schema)

#### Step 3: Run Query
Click the **"Run"** button or press `Cmd + Enter`

#### Step 4: View Results
Results appear in the bottom panel

---

### Common SQL Queries:

#### A. View All Tenants
```sql
SELECT
  t.name,
  t.slug,
  t.custom_domain,
  tc.primary_color,
  tc.email_from
FROM portal.tenants t
LEFT JOIN portal.tenant_config tc ON t.id = tc.tenant_id
ORDER BY t.created_at DESC;
```

#### B. Check Tenant by Domain
```sql
SELECT *
FROM portal.tenants
WHERE custom_domain = 'bs.co.uk';
```

#### C. Update Tenant
```sql
UPDATE portal.tenants
SET custom_domain = 'new-domain.com'
WHERE slug = 'tenant-slug'
RETURNING *;
```

#### D. View Feature Flags
```sql
SELECT
  t.name as tenant_name,
  ff.module_name,
  ff.enabled
FROM portal.feature_flags ff
JOIN portal.tenants t ON ff.tenant_id = t.id
WHERE t.slug = 'bs';
```

---

## Method 3: Supabase CLI (For Migrations)

**Best for:** Running migration files, managing schema changes

### Step-by-Step Instructions:

#### Step 1: Check if CLI is Installed
```bash
which supabase
# Should output: /usr/local/bin/supabase
```

If not installed:
```bash
brew install supabase/tap/supabase
```

#### Step 2: Login (One-Time Setup)
```bash
supabase login
```

This opens a browser to authenticate.

#### Step 3: Link to Project (One-Time Setup)
```bash
cd ~/Desktop/portal-platform
supabase link --project-ref uusjsarpkhonjpglxzgn
```

#### Step 4: Run Migrations
```bash
# Push all migrations to production
supabase db push

# Or execute a specific SQL file
supabase db execute < scripts/your-migration.sql
```

---

## Method 4: From Your Next.js App

**Best for:** Testing how the app connects to Supabase

### Server-Side (Server Components, API Routes):

```typescript
// app/api/test/route.ts
import { createClient } from '@/lib/supabase/server'

export async function GET() {
  const supabase = await createClient()

  const { data, error } = await supabase
    .from('tenants')
    .select('*')

  return Response.json({ data, error })
}
```

### Client-Side (Client Components):

```typescript
// components/TenantList.tsx
'use client'
import { createClient } from '@/lib/supabase/client'

export default function TenantList() {
  const supabase = createClient()

  const { data } = await supabase
    .from('tenants')
    .select('*')

  return <div>{/* render data */}</div>
}
```

---

## Understanding Service Role Key vs Anon Key

### Anon Key (Public Access)
- **Used for:** Client-side code, public API calls
- **Access Level:** Restricted by Row Level Security (RLS)
- **Safe to expose:** Yes, can be in client-side code
- **Variable:** `NEXT_PUBLIC_SUPABASE_ANON_KEY`

### Service Role Key (Admin Access)
- **Used for:** Server-side code, admin scripts
- **Access Level:** FULL ACCESS (bypasses RLS)
- **Safe to expose:** NO! Never in client code or version control
- **Variable:** `SUPABASE_SERVICE_ROLE_KEY`

**Our Node.js scripts use the Service Role Key** because:
1. They run server-side only (not in browser)
2. They need to access the `portal` schema
3. They need to bypass RLS for admin operations

---

## Quick Troubleshooting

### "Cannot find module '@supabase/supabase-js'"
```bash
cd ~/Desktop/portal-platform
npm install
```

### "Unauthorized" or "Invalid API key"
Check your `.env.local` file:
```bash
cat .env.local | grep SUPABASE
```

Verify keys match the dashboard:
https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn/settings/api

### "relation 'tenants' does not exist"
You forgot the `portal.` schema prefix:
- ❌ `FROM tenants`
- ✅ `FROM portal.tenants`

### "Cannot read properties of null (reading 'config')"
`.env.local` file not found or not loaded:
```bash
# Make sure you're in the project directory
cd ~/Desktop/portal-platform

# Check file exists
ls -la .env.local

# Run script from project root
node scripts/your-script.js
```

---

## Schema Information

### Portal Schema Tables:
```
portal.tenants
portal.tenant_config
portal.users
portal.roles
portal.user_roles
portal.feature_flags
portal.bookings
portal.services
portal.service_availability
portal.email_templates
portal.cancellation_policies
portal.sms_logs
portal.sms_scheduled_jobs
portal.audit_logs
```

### Why "portal" schema?
The database is **shared** with the Beyond Spreadsheets website. Using a separate `portal` schema prevents conflicts with the website's tables in the `public` schema.

---

## Real-World Examples from This Session

### 1. Check if Beyond Spreadsheets tenant exists:
```bash
node scripts/verify-bs-tenant.js
```

Output:
```
✅ Tenant Found:
   ID: 6663144e-dd12-4fbd-9a79-6a4e365437d7
   Name: Beyond Spreadsheets
   Domain: bs.co.uk
```

### 2. Fix tenant subdomain issue:
```bash
node scripts/fix-bs-tenant.js
```

This updated the tenant to use custom domain only (set subdomain to NULL).

### 3. Check for duplicates:
```bash
node scripts/check-duplicate-tenants.js
```

Found the issue: tenant had both subdomain AND custom_domain set.

---

## DO NOT Use These Methods ❌

### ❌ Supabase MCP Tools
```typescript
mcp__plugin_supabase_supabase__execute_sql() // Returns "Unauthorized"
mcp__plugin_supabase_supabase__apply_migration() // Returns "Unauthorized"
```

**Why:** Requires additional authentication setup that we don't have configured.

### ❌ Direct REST API SQL Execution
```bash
curl -X POST https://uusjsarpkhonjpglxzgn.supabase.co/rest/v1/...
```

**Why:** Supabase doesn't support raw SQL execution via REST API for security.

### ❌ Database Connection Strings
```bash
psql postgresql://postgres:password@...pooler.supabase.com:6543/postgres
```

**Why:** Requires database password (which is separate from API keys) and is more complex than needed.

---

## Security Best Practices

### ✅ DO:
- Keep `.env.local` in `.gitignore`
- Use service role key only in server-side code
- Use anon key for client-side code
- Run admin scripts locally, not in browser
- Commit scripts to git, but never commit `.env.local`

### ❌ DON'T:
- Never commit service role key to git
- Never use service role key in client-side code
- Never share service role key in screenshots/logs
- Never hardcode keys in scripts (always use `.env.local`)

---

## Getting Help

### Official Documentation:
- **Supabase JS Client:** https://supabase.com/docs/reference/javascript/introduction
- **Supabase CLI:** https://supabase.com/docs/guides/cli
- **SQL Reference:** https://www.postgresql.org/docs/

### Project Documentation:
- **Technical Docs:** `~/Desktop/CLAUDE.md`
- **Debugging Guide:** `~/Desktop/TENANT_DEBUGGING_SUMMARY.md`
- **This Guide:** `~/Desktop/SUPABASE_CONNECTION_GUIDE.md`

### Dashboard Links:
- **Project Dashboard:** https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn
- **SQL Editor:** https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn/sql/new
- **Table Editor:** https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn/editor
- **API Settings:** https://supabase.com/dashboard/project/uusjsarpkhonjpglxzgn/settings/api

---

## Summary Checklist

When you need to connect to Supabase:

1. ✅ **Am I querying/updating data?** → Use Node.js script (Method 1)
2. ✅ **Am I running a migration?** → Use SQL Editor (Method 2) or CLI (Method 3)
3. ✅ **Am I testing app integration?** → Use Next.js API route (Method 4)
4. ✅ **Always use `portal.` schema prefix** for table names
5. ✅ **Always run scripts from project root** (`~/Desktop/portal-platform`)
6. ✅ **Check `.env.local` exists** before running Node.js scripts

---

**End of Guide**
