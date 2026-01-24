---
description: Create tenant-specific page (isolated, safe)
autoApprove:
  - Read
  - Glob
  - Write
  - Bash(mkdir *)
---

You are creating a TENANT-SPECIFIC PAGE in the multi-tenant SaaS portal-platform.

```
═══════════════════════════════════════════
   TENANT-SPECIFIC PAGE
═══════════════════════════════════════════

✅ This page will ONLY affect: {Tenant Name}
✅ Other tenants: Will NOT be affected
✅ Platform code: Will NOT be modified
✅ Safe to create without impacting other clients

Changes go to: app/(tenants)/{slug}/{page-name}/
```

## Command Format

`/pp-page <tenant-slug> <page-name>`

Examples:
- `/pp-page bs home`
- `/pp-page dkc volunteer-application`
- `/pp-page fitzone-gym class-schedule`

## What This Creates

Creates a new page in: `app/(tenants)/<slug>/<page-name>/page.tsx`

**Safety:**
- ✅ Only affects that tenant's folder
- ✅ Cannot break other tenants
- ✅ Isolated in `app/(tenants)/{slug}/` directory
- ✅ No platform code changes

## Step 1: Verify Tenant Exists

Check that the tenant folder exists:

```bash
if [ ! -d "/Users/George/Desktop/portal-platform/app/(tenants)/SLUG" ]; then
  echo "❌ Tenant folder not found: SLUG"
  echo "Create tenant first with: /pp-tenant \"Tenant Name\""
  exit 1
fi
```

## Step 2: Ask Questions

Ask the user about the page requirements:

1. **What should this page do?**
   - Example: "Display class schedule", "Show volunteer application form", "Homepage with hero and features"

2. **Does this page need data from the database?**
   - [ ] Yes - which tables? (portal.bookings, portal.services, etc.)
   - [ ] No - static content only

3. **Should this be a public page or require login?**
   - [ ] Public (anyone can view)
   - [ ] Requires authentication

4. **Any special features?**
   - [ ] Form submission
   - [ ] Real-time data
   - [ ] Charts/graphs
   - [ ] File uploads
   - [ ] Search/filters

## Step 3: Create Page Directory

```bash
mkdir -p "/Users/George/Desktop/portal-platform/app/(tenants)/SLUG/PAGE_NAME"
echo "✅ Created directory: app/(tenants)/SLUG/PAGE_NAME/"
```

## Step 4: Create Page Component

Based on requirements, create the page file:

### Template A: Static Page (No Database)

```typescript
// app/(tenants)/SLUG/PAGE_NAME/page.tsx

export default function PageName() {
  return (
    <div className="min-h-screen">
      <main className="container mx-auto px-4 py-16">
        <h1 className="text-4xl font-bold mb-8">
          PAGE_TITLE
        </h1>

        {/* Add content here */}
        <p>Content for this page...</p>
      </main>
    </div>
  )
}
```

### Template B: Dynamic Page with Data

```typescript
// app/(tenants)/SLUG/PAGE_NAME/page.tsx
import { headers } from 'next/headers'
import { getTenantFromHostname } from '@/lib/tenant/context'
import { createClient } from '@/lib/supabase/server'

export default async function PageName() {
  const headersList = await headers()
  const hostname = headersList.get('host') || 'localhost:3000'
  const tenant = await getTenantFromHostname(hostname)

  // Security check
  if (!tenant || tenant.slug !== 'SLUG') {
    return <div>Not found</div>
  }

  // Fetch data for THIS tenant only
  const supabase = await createClient()
  const { data, error } = await supabase
    .from('portal.TABLE_NAME')
    .select('*')
    .eq('tenant_id', tenant.id) // CRITICAL: Tenant isolation
    .order('created_at', { ascending: false })

  if (error) {
    console.error('Error fetching data:', error)
  }

  return (
    <div className="min-h-screen">
      <main className="container mx-auto px-4 py-16">
        <h1 className="text-4xl font-bold mb-8">
          PAGE_TITLE
        </h1>

        {/* Display data */}
        {data && data.length > 0 ? (
          <div className="grid gap-6">
            {data.map((item) => (
              <div key={item.id} className="border p-6 rounded-lg">
                {/* Render item */}
                <h3>{item.name || item.title}</h3>
              </div>
            ))}
          </div>
        ) : (
          <p>No data found.</p>
        )}
      </main>
    </div>
  )
}
```

### Template C: Client Component with Form

```typescript
// app/(tenants)/SLUG/PAGE_NAME/page.tsx
"use client";

import { useState } from 'react'
import { createClient } from '@/lib/supabase/client'

export default function PageName() {
  const [formData, setFormData] = useState({
    // form fields
  })
  const [submitting, setSubmitting] = useState(false)

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setSubmitting(true)

    try {
      const supabase = createClient()

      // Get current user
      const { data: { user } } = await supabase.auth.getUser()
      if (!user) {
        alert('Please log in')
        return
      }

      // Get user's tenant
      const { data: userData } = await supabase
        .from('portal.users')
        .select('tenant_id')
        .eq('id', user.id)
        .single()

      // Submit with tenant_id
      const { error } = await supabase
        .from('portal.TABLE_NAME')
        .insert({
          tenant_id: userData.tenant_id, // Automatic isolation
          ...formData
        })

      if (error) {
        alert('Error: ' + error.message)
      } else {
        alert('Success!')
        setFormData({}) // Reset
      }
    } catch (err: any) {
      alert('Error: ' + err.message)
    } finally {
      setSubmitting(false)
    }
  }

  return (
    <div className="min-h-screen">
      <main className="container mx-auto px-4 py-16">
        <h1 className="text-4xl font-bold mb-8">
          PAGE_TITLE
        </h1>

        <form onSubmit={handleSubmit} className="max-w-lg space-y-6">
          {/* Form fields */}
          <div>
            <label className="block text-sm font-medium mb-2">
              Field Name
            </label>
            <input
              type="text"
              value={formData.field}
              onChange={(e) => setFormData({...formData, field: e.target.value})}
              className="w-full px-3 py-2 border rounded-md"
              required
            />
          </div>

          <button
            type="submit"
            disabled={submitting}
            className="w-full bg-primary text-white py-3 rounded-md hover:opacity-90 disabled:opacity-50"
          >
            {submitting ? 'Submitting...' : 'Submit'}
          </button>
        </form>
      </main>
    </div>
  )
}
```

### Template D: Page with API Route

If the page needs complex data logic, create an API route too:

```typescript
// app/api/SLUG/PAGE_NAME/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'

export async function GET(request: NextRequest) {
  const supabase = await createClient()
  const { searchParams } = new URL(request.url)
  const tenantId = searchParams.get('tenant_id')

  if (!tenantId) {
    return NextResponse.json(
      { error: 'tenant_id required' },
      { status: 400 }
    )
  }

  // Fetch data for THIS tenant
  const { data, error } = await supabase
    .from('portal.TABLE_NAME')
    .select('*')
    .eq('tenant_id', tenantId)

  if (error) {
    return NextResponse.json({ error: error.message }, { status: 400 })
  }

  return NextResponse.json({ data })
}
```

And call it from the page:

```typescript
// In the page component
const response = await fetch(
  `/api/SLUG/PAGE_NAME?tenant_id=${tenant.id}`
)
const result = await response.json()
```

## Step 5: Add to Navigation (Optional)

If the tenant has a navigation component, update it:

```typescript
// Ask user if they want to add to navigation
// Update app/(tenants)/SLUG/components/Navigation.tsx

<nav>
  <Link href="/">Home</Link>
  <Link href="/PAGE_NAME">PAGE_TITLE</Link>
</nav>
```

## Step 6: Test the Page

```bash
# Start dev server
npm run dev

# Visit the page
# If tenant is local: http://localhost:3000/PAGE_NAME
# If using subdomain: http://SLUG.localhost:3000/PAGE_NAME
```

## Step 7: Create README

```markdown
// app/(tenants)/SLUG/PAGE_NAME/README.md

# PAGE_TITLE

## Purpose
[What this page does]

## Features
- [Feature 1]
- [Feature 2]

## Data Sources
- Tables: portal.TABLE_NAME
- Tenant-scoped: Yes ✅

## Access
- Public: [Yes/No]
- Requires login: [Yes/No]

## Testing
Visit: http://SLUG.localhost:3000/PAGE_NAME
```

## Safety Checklist

Before completing:

- [ ] Page created in correct tenant folder
- [ ] Tenant slug check included (if using server component)
- [ ] All database queries include `.eq('tenant_id', tenant.id)`
- [ ] No hardcoded tenant references
- [ ] Uses `portal.` schema prefix
- [ ] Cannot access other tenants' data
- [ ] Tested locally

## Summary

Tell the user:

```
✅ Page Created: PAGE_TITLE

📂 Location:
  app/(tenants)/SLUG/PAGE_NAME/page.tsx

🔒 Security:
  - Only accessible to SLUG tenant
  - Data filtered by tenant_id
  - Cannot access other tenants

🧪 Test:
  1. Update /etc/hosts:
     127.0.0.1 SLUG.localhost

  2. Visit:
     http://SLUG.localhost:3000/PAGE_NAME

  3. Verify:
     - Page loads
     - Data is correct
     - Only THIS tenant's data shows

📝 Next:
  - Create more pages: /pp-page SLUG <name>
  - Use shared components: /pp-component <name>
  - List all pages: /pp-list
```

## Common Page Types

**Homepage:**
```
/pp-page bs home
```

**About Page:**
```
/pp-page dkc about
```

**Contact/Booking Form:**
```
/pp-page fitzone-gym book-class
```

**Dashboard (requires login):**
```
/pp-page ayup dashboard
```

**Custom Feature:**
```
/pp-page dkc volunteer-application
```

Each page is **isolated** - working on one tenant's pages cannot break another!
