---
description: Create shared component (reusable across all tenants)
autoApprove:
  - Read
  - Glob
  - Write
  - Bash(mkdir *)
---

You are creating a SHARED COMPONENT that can be used by ALL tenants in the multi-tenant SaaS.

## Command Format

`/pp-component <ComponentName>`

Examples:
- `/pp-component HeroSection`
- `/pp-component BookingWidget`
- `/pp-component PricingCard`
- `/pp-component DashboardMetric`

## What This Creates

Creates a reusable component in: `components/shared/<ComponentName>.tsx`

**Purpose:** Share code across all tenants, avoid duplication

**Safety:** Component is tenant-aware via props, won't mix tenant data

## Step 1: Ask Questions

Ask the user about the component:

1. **What does this component do?**
   - Example: "Display a hero section", "Render a booking form", "Show pricing cards"

2. **What props should it accept?**
   - Content (title, subtitle, etc.)?
   - Styling (colors, sizes)?
   - Data (items to display)?
   - Tenant info (for branding)?

3. **Does it need state/interactivity?**
   - [ ] Yes - needs useState, event handlers
   - [ ] No - pure display component

4. **Does it fetch data?**
   - [ ] Yes - needs to fetch from database
   - [ ] No - receives data as props

## Step 2: Create Component Directory

```bash
mkdir -p "/Users/George/Desktop/portal-platform/components/shared"
echo "✅ Directory ready: components/shared/"
```

## Step 3: Create Component File

Based on requirements, create the component:

### Template A: Simple Display Component

```typescript
// components/shared/ComponentName.tsx

export interface ComponentNameProps {
  title: string
  subtitle?: string
  tenant?: {
    name: string
    config?: {
      primary_color: string
      secondary_color: string
    }
  }
}

export function ComponentName({
  title,
  subtitle,
  tenant
}: ComponentNameProps) {
  return (
    <div
      className="p-8 rounded-lg"
      style={{
        backgroundColor: tenant?.config?.primary_color || '#0891b2'
      }}
    >
      <h2 className="text-3xl font-bold text-white mb-2">
        {title}
      </h2>
      {subtitle && (
        <p className="text-white/90">
          {subtitle}
        </p>
      )}
    </div>
  )
}
```

**Usage example:**
```typescript
// In any tenant page:
<ComponentName
  title="Welcome to FitZone!"
  subtitle="Your fitness journey starts here"
  tenant={tenant}
/>
```

### Template B: Interactive Component

```typescript
// components/shared/ComponentName.tsx
"use client";

import { useState } from 'react'

export interface ComponentNameProps {
  onSubmit: (data: any) => void
  tenant?: any
}

export function ComponentName({
  onSubmit,
  tenant
}: ComponentNameProps) {
  const [value, setValue] = useState('')
  const [loading, setLoading] = useState(false)

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setLoading(true)

    try {
      await onSubmit({ value })
      setValue('') // Reset
    } catch (error) {
      console.error('Submit error:', error)
    } finally {
      setLoading(false)
    }
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Enter value..."
        className="w-full px-4 py-2 border rounded-md"
        style={{
          borderColor: tenant?.config?.primary_color
        }}
      />

      <button
        type="submit"
        disabled={loading}
        className="w-full py-3 rounded-md text-white font-medium disabled:opacity-50"
        style={{
          backgroundColor: tenant?.config?.primary_color || '#0891b2'
        }}
      >
        {loading ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  )
}
```

**Usage example:**
```typescript
// In any tenant page:
<ComponentName
  tenant={tenant}
  onSubmit={async (data) => {
    const supabase = createClient()
    await supabase.from('portal.table').insert({
      tenant_id: tenant.id,
      ...data
    })
  }}
/>
```

### Template C: Data Display Component

```typescript
// components/shared/ComponentName.tsx

export interface ComponentNameProps {
  items: any[]
  tenant?: any
  onItemClick?: (item: any) => void
}

export function ComponentName({
  items,
  tenant,
  onItemClick
}: ComponentNameProps) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {items.map((item) => (
        <div
          key={item.id}
          className="border rounded-lg p-6 hover:shadow-lg transition-shadow cursor-pointer"
          onClick={() => onItemClick?.(item)}
          style={{
            borderColor: tenant?.config?.primary_color || '#e5e7eb'
          }}
        >
          <h3 className="text-xl font-bold mb-2">{item.title}</h3>
          <p className="text-gray-600">{item.description}</p>

          {item.price && (
            <p
              className="text-lg font-bold mt-4"
              style={{
                color: tenant?.config?.primary_color
              }}
            >
              £{item.price}
            </p>
          )}
        </div>
      ))}
    </div>
  )
}
```

**Usage example:**
```typescript
// In any tenant page:
const { data: services } = await supabase
  .from('portal.services')
  .select('*')
  .eq('tenant_id', tenant.id)

<ComponentName
  items={services}
  tenant={tenant}
  onItemClick={(service) => {
    router.push(`/book/${service.id}`)
  }}
/>
```

### Template D: Layout Component

```typescript
// components/shared/ComponentName.tsx

export interface ComponentNameProps {
  children: React.ReactNode
  tenant?: any
  showNav?: boolean
}

export function ComponentName({
  children,
  tenant,
  showNav = true
}: ComponentNameProps) {
  return (
    <div className="min-h-screen flex flex-col">
      {showNav && (
        <nav
          className="bg-white shadow-sm py-4 px-6"
          style={{
            borderBottomColor: tenant?.config?.primary_color,
            borderBottomWidth: '3px'
          }}
        >
          <div className="container mx-auto flex items-center justify-between">
            {tenant?.config?.logo_url && (
              <img
                src={tenant.config.logo_url}
                alt={tenant.name}
                className="h-10"
              />
            )}
            <div className="text-2xl font-bold">
              {tenant?.name}
            </div>
          </div>
        </nav>
      )}

      <main className="flex-1">
        {children}
      </main>

      <footer className="bg-gray-100 py-8 px-6 text-center text-gray-600">
        <p>© {new Date().getFullYear()} {tenant?.name}. All rights reserved.</p>
      </footer>
    </div>
  )
}
```

**Usage example:**
```typescript
// In any tenant layout:
<ComponentName tenant={tenant}>
  <YourPageContent />
</ComponentName>
```

## Step 4: Create TypeScript Types File (Optional)

If the component has complex types:

```typescript
// components/shared/types.ts

export interface Tenant {
  id: string
  name: string
  slug: string
  config?: {
    logo_url?: string
    primary_color: string
    secondary_color: string
  }
}

export interface ComponentNameItem {
  id: string
  title: string
  description: string
  // ... other fields
}
```

Then import in component:
```typescript
import { Tenant, ComponentNameItem } from './types'
```

## Step 5: Create Example Usage File

```typescript
// components/shared/ComponentName.example.tsx

import { ComponentName } from './ComponentName'

// Example 1: Basic usage
export function Example1() {
  return (
    <ComponentName
      title="Welcome"
      subtitle="This is an example"
    />
  )
}

// Example 2: With tenant branding
export function Example2() {
  const tenant = {
    name: 'FitZone Gym',
    config: {
      primary_color: '#ff6b35',
      secondary_color: '#f7931e'
    }
  }

  return (
    <ComponentName
      title="Welcome to FitZone"
      tenant={tenant}
    />
  )
}

// Example 3: With data
export function Example3() {
  const items = [
    { id: 1, title: 'Item 1', description: 'Description 1' },
    { id: 2, title: 'Item 2', description: 'Description 2' },
  ]

  return (
    <ComponentName items={items} />
  )
}
```

## Step 6: Create README

```markdown
// components/shared/ComponentName.README.md

# ComponentName

## Purpose
[What this component does]

## Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| title | string | Yes | The title to display |
| subtitle | string | No | Optional subtitle |
| tenant | Tenant | No | Tenant info for branding |

## Usage

\`\`\`tsx
import { ComponentName } from '@/components/shared/ComponentName'

<ComponentName
  title="My Title"
  subtitle="My Subtitle"
  tenant={tenant}
/>
\`\`\`

## Examples

See `ComponentName.example.tsx` for usage examples.

## Tenant Awareness

This component adapts to tenant branding:
- Uses `tenant.config.primary_color` for styling
- Uses `tenant.config.logo_url` if provided
- Respects tenant customization

## Notes

- [Any special considerations]
- [Performance notes]
- [Accessibility features]
```

## Step 7: Export from Index (Optional)

Add to `components/shared/index.ts`:

```typescript
export { ComponentName } from './ComponentName'
export type { ComponentNameProps } from './ComponentName'
```

## Validation Checklist

- [ ] Component name is PascalCase
- [ ] TypeScript interfaces defined
- [ ] Props include `tenant?` for branding
- [ ] Component is tenant-agnostic (no hardcoded tenant data)
- [ ] Styling respects tenant colors
- [ ] README created with usage examples
- [ ] Example file created (optional but helpful)

## Safety Guarantees

✅ **Tenant Isolation:**
- Component doesn't fetch data directly
- Receives tenant info as props
- Cannot access other tenant's data
- Each tenant can use same component differently

✅ **Reusability:**
- One component, all tenants
- Customizable via props
- Tenant branding applied dynamically

## Summary

Tell the user:

```
✅ Shared Component Created: ComponentName

📂 Location:
  components/shared/ComponentName.tsx
  components/shared/ComponentName.README.md
  components/shared/ComponentName.example.tsx

🎨 Features:
  - Tenant-aware (uses tenant branding)
  - Reusable across all tenants
  - Type-safe with TypeScript
  - Documented with examples

📖 Usage:
  import { ComponentName } from '@/components/shared/ComponentName'

  <ComponentName
    title="My Title"
    tenant={tenant}
  />

🔄 Reuse:
  This component can now be used in ANY tenant page:
  - beyond-spreadsheets
  - dkc
  - ayup
  - fitzone-gym
  - all future tenants!

📝 Next:
  - Use in pages: import and add to any page
  - Create more components: /pp-component <Name>
  - View all components: ls components/shared/
```

## Common Shared Components

**Hero Section:**
```
/pp-component HeroSection
```

**Navigation Bar:**
```
/pp-component Navigation
```

**Footer:**
```
/pp-component Footer
```

**Pricing Card:**
```
/pp-component PricingCard
```

**Testimonial:**
```
/pp-component Testimonial
```

**Booking Form:**
```
/pp-component BookingForm
```

**Dashboard Widget:**
```
/pp-component DashboardWidget
```

Each component is **shared** but **tenant-aware** - same code, different appearance per tenant!
