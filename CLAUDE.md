# Portal Platform - Technical Documentation

Complete technical documentation for the multi-tenant portal platform.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Multi-Tenancy](#multi-tenancy)
3. [Database Schema](#database-schema)
4. [Authentication & Authorization](#authentication--authorization)
5. [Feature Flags](#feature-flags)
6. [White-Labeling](#white-labeling)
7. [API Routes](#api-routes)
8. [Deployment](#deployment)
9. [Development Workflow](#development-workflow)

## Architecture Overview

### Core Technologies

- **Next.js 15**: React framework with App Router
- **TypeScript**: Type-safe development
- **Supabase**: Backend-as-a-Service (Database + Auth)
- **Tailwind CSS**: Utility-first CSS framework
- **Stripe**: Payment processing
- **Resend**: Transactional email
- **Upstash Redis**: Rate limiting and caching

### Design Principles

1. **Multi-tenancy First**: Every feature considers tenant isolation
2. **Security by Default**: Row-level security on all tables
3. **Modular Architecture**: Features can be enabled/disabled per tenant
4. **Type Safety**: Comprehensive TypeScript coverage
5. **Performance**: Server-side rendering, edge functions, caching

## Multi-Tenancy

### Tenant Resolution

Tenants are identified by hostname:

```typescript
// lib/tenant/context.ts
export async function getTenantFromHostname(hostname: string): Promise<Tenant | null> {
  const subdomain = hostname.split('.')[0]

  const { data: tenant } = await supabase
    .from('tenants')
    .select('*, config:tenant_config(*)')
    .or(`subdomain.eq.${subdomain},custom_domain.eq.${hostname}`)
    .single()

  return tenant
}
```

### Hostname Patterns

1. **Subdomain**: `gym-a.portal.com`
   - Subdomain = `gym-a`
   - Matches `tenants.subdomain`

2. **Custom Domain**: `mygym.com`
   - Full hostname match
   - Matches `tenants.custom_domain`

### Tenant Context

Every request includes tenant context:

```typescript
// app/page.tsx
const headersList = await headers()
const hostname = headersList.get('host') || 'localhost:3000'
const tenant = await getTenantFromHostname(hostname)
```

### Middleware

The middleware handles tenant resolution and authentication:

```typescript
// middleware.ts
export async function middleware(request: NextRequest) {
  // 1. Refresh Supabase session
  const { supabaseResponse, user } = await updateSession(request)

  // 2. Check protected routes
  const isProtectedPath = protectedPaths.some(path =>
    request.nextUrl.pathname.startsWith(path)
  )

  // 3. Redirect if unauthorized
  if (isProtectedPath && !user) {
    return NextResponse.redirect(new URL('/auth/login', request.url))
  }

  return supabaseResponse
}
```

## Database Schema

### Core Tables

#### tenants
```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  subdomain TEXT UNIQUE,
  custom_domain TEXT UNIQUE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### tenant_config
```sql
CREATE TABLE tenant_config (
  tenant_id UUID PRIMARY KEY REFERENCES tenants(id),
  logo_url TEXT,
  primary_color TEXT DEFAULT '#0891b2',
  secondary_color TEXT DEFAULT '#06b6d4',
  email_from TEXT,
  support_email TEXT,
  require_login_for_booking BOOLEAN DEFAULT false,
  allow_public_registration BOOLEAN DEFAULT true,
  enable_role_based_access BOOLEAN DEFAULT false,
  show_login_on_homepage BOOLEAN DEFAULT false,
  homepage_layout TEXT DEFAULT 'public_booking'
);
```

#### feature_flags
```sql
CREATE TABLE feature_flags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  module_name TEXT NOT NULL,
  enabled BOOLEAN DEFAULT false,
  config JSONB DEFAULT '{}',
  activated_at TIMESTAMPTZ,
  deactivated_at TIMESTAMPTZ,
  UNIQUE(tenant_id, module_name)
);
```

#### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  email TEXT NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### roles
```sql
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  name TEXT NOT NULL,
  description TEXT,
  permissions JSONB DEFAULT '[]',
  UNIQUE(tenant_id, name)
);
```

#### user_roles
```sql
CREATE TABLE user_roles (
  user_id UUID NOT NULL REFERENCES users(id),
  role_id UUID NOT NULL REFERENCES roles(id),
  granted_at TIMESTAMPTZ DEFAULT NOW(),
  granted_by UUID REFERENCES users(id),
  PRIMARY KEY (user_id, role_id)
);
```

### Row-Level Security (RLS)

All tables have RLS enabled with policies:

```sql
-- Example: Users can only see users in their tenant
CREATE POLICY "Users can view users in their tenant"
  ON users FOR SELECT
  USING (tenant_id = get_user_tenant_id());
```

Helper function:
```sql
CREATE FUNCTION get_user_tenant_id() RETURNS UUID AS $$
BEGIN
  RETURN (SELECT tenant_id FROM users WHERE id = auth.uid());
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

## Authentication & Authorization

### Authentication Flow

1. User signs in via Supabase Auth
2. Session stored in HTTP-only cookies
3. Middleware refreshes session on each request
4. User linked to tenant via `users.tenant_id`

### RBAC System

#### Permission Constants

```typescript
// lib/rbac/permissions.ts
export const PERMISSIONS = {
  'users.read': 'Read users',
  'users.create': 'Create users',
  'users.update': 'Update users',
  'users.delete': 'Delete users',
  'roles.read': 'Read roles',
  'admin.full_access': 'Full admin access',
} as const
```

#### Checking Permissions

```typescript
import { hasPermission, hasAnyPermission } from '@/lib/rbac/permissions'

// Check single permission
const canEdit = await hasPermission(userId, 'users.update')

// Check any of multiple permissions
const canManage = await hasAnyPermission(userId, ['users.update', 'users.delete'])
```

#### Protected Routes

```typescript
// app/admin/page.tsx
export default async function AdminPage() {
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    redirect('/auth/login')
  }

  const hasAccess = await hasPermission(user.id, 'admin.full_access')

  if (!hasAccess) {
    return <AccessDenied />
  }

  // Render admin content...
}
```

## Feature Flags

### Available Modules

```typescript
// lib/feature-flags.ts
export const MODULES = {
  PUBLIC_BOOKING: 'public_booking',
  CUSTOMER_PORTAL: 'customer_portal',
  SMS: 'sms',
  DOCUMENTS: 'documents',
  PAYMENTS: 'payments',
  REPORTS: 'reports',
  API_ACCESS: 'api_access',
} as const
```

### Checking Feature Availability

```typescript
import { isFeatureEnabled, getFeatureConfig } from '@/lib/feature-flags'

// Check if feature is enabled
const enabled = await isFeatureEnabled(tenantId, MODULES.PUBLIC_BOOKING)

// Get feature configuration
const config = await getFeatureConfig(tenantId, MODULES.SMS)
if (config?.provider === 'twilio') {
  // Use Twilio
}
```

### Feature-Gated Components

```typescript
// app/book/page.tsx
export default async function BookPage() {
  const tenant = await getTenantFromHostname(hostname)
  const bookingEnabled = await isFeatureEnabled(tenant.id, MODULES.PUBLIC_BOOKING)

  if (!bookingEnabled) {
    return <FeatureNotAvailable />
  }

  return <BookingInterface />
}
```

### Managing Features

Feature flags are managed via database:

```sql
-- Enable a feature
INSERT INTO feature_flags (tenant_id, module_name, enabled, activated_at)
VALUES ('tenant-uuid', 'public_booking', true, NOW());

-- Disable a feature
UPDATE feature_flags
SET enabled = false, deactivated_at = NOW()
WHERE tenant_id = 'tenant-uuid' AND module_name = 'sms';

-- Update feature config
UPDATE feature_flags
SET config = '{"provider": "twilio", "from_number": "+1234567890"}'::jsonb
WHERE tenant_id = 'tenant-uuid' AND module_name = 'sms';
```

## White-Labeling

### Branding Configuration

Each tenant has customizable branding:

```typescript
interface TenantConfig {
  logo_url: string | null
  primary_color: string      // e.g., '#0891b2'
  secondary_color: string    // e.g., '#06b6d4'
  email_from: string | null
  support_email: string | null
}
```

### Applying Branding

#### CSS Variables

```typescript
// app/layout.tsx
export default async function RootLayout({ children }) {
  const tenant = await getTenantFromHostname(hostname)
  const primaryColor = tenant?.config?.primary_color || '#0891b2'

  return (
    <html>
      <head>
        <style dangerouslySetInnerHTML={{
          __html: `
            :root {
              --color-primary: ${primaryColor};
              --color-secondary: ${secondaryColor};
            }
          `
        }} />
      </head>
      <body>{children}</body>
    </html>
  )
}
```

#### Logo Display

```typescript
{tenant.config?.logo_url && (
  <img src={tenant.config.logo_url} alt={tenant.name} />
)}
```

#### Tailwind Customization

Use CSS variables in Tailwind config:

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: 'var(--color-primary)',
        secondary: 'var(--color-secondary)',
      }
    }
  }
}
```

## API Routes

### Structure

```
app/api/
├── tenants/
│   ├── route.ts           # GET /api/tenants
│   └── [id]/
│       └── route.ts       # GET /api/tenants/:id
├── features/
│   └── route.ts           # GET /api/features
├── users/
│   └── route.ts           # GET /api/users
└── auth/
    ├── login/
    └── logout/
```

### Example API Route

```typescript
// app/api/tenants/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'

export async function GET(request: NextRequest) {
  const supabase = await createClient()

  const { data: tenants, error } = await supabase
    .from('tenants')
    .select('*')

  if (error) {
    return NextResponse.json({ error: error.message }, { status: 500 })
  }

  return NextResponse.json(tenants)
}
```

## Deployment

### Environment Setup

#### Production Environment Variables

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...

# App
NEXTAUTH_SECRET=<strong-random-secret>
NEXTAUTH_URL=https://portal.yourdomain.com

# Integrations
STRIPE_SECRET_KEY=sk_live_...
RESEND_API_KEY=re_...
UPSTASH_REDIS_REST_URL=https://...
UPSTASH_REDIS_REST_TOKEN=...
```

### Vercel Deployment

1. **Connect Repository**
   ```bash
   vercel link
   ```

2. **Set Environment Variables**
   - Go to Vercel Dashboard > Settings > Environment Variables
   - Add all production variables

3. **Deploy**
   ```bash
   vercel deploy --prod
   ```

### Database Migrations

Run migrations in production Supabase:

```bash
# Using Supabase CLI
supabase link --project-ref your-prod-ref
supabase db push
```

Or manually execute SQL in Supabase SQL Editor.

### DNS Configuration

#### Subdomain Setup

Add CNAME record:
```
gym-a.portal.com  CNAME  cname.vercel-dns.com
```

#### Custom Domain

Add A/CNAME records for custom domains:
```
mygym.com  CNAME  cname.vercel-dns.com
```

In Vercel, add all domains in Settings > Domains.

## Development Workflow

### Setup

```bash
# Clone repository
git clone <repo-url>
cd portal-platform

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.local

# Fill in your Supabase credentials in .env.local
```

### Running Locally

```bash
# Development server
npm run dev

# Run tests
npm test

# Lint
npm run lint
```

### Testing Multi-Tenancy Locally

Edit `/etc/hosts`:
```
127.0.0.1  tenant-a.localhost
127.0.0.1  tenant-b.localhost
```

Access at:
- `http://tenant-a.localhost:3000`
- `http://tenant-b.localhost:3000`

### Creating Test Tenants

```sql
-- Insert test tenant
INSERT INTO tenants (id, name, slug, subdomain)
VALUES (
  gen_random_uuid(),
  'Test Gym A',
  'gym-a',
  'gym-a'
);

-- Insert tenant config
INSERT INTO tenant_config (tenant_id, primary_color, secondary_color)
SELECT id, '#ef4444', '#f87171'
FROM tenants WHERE slug = 'gym-a';

-- Enable features
INSERT INTO feature_flags (tenant_id, module_name, enabled, activated_at)
SELECT id, 'public_booking', true, NOW()
FROM tenants WHERE slug = 'gym-a';
```

### Type Generation

Generate types from Supabase schema:

```bash
npx supabase gen types typescript --project-id <project-ref> > types/database.ts
```

## Troubleshooting

### Common Issues

#### 1. "Tenant Not Found"
- Verify tenant exists in database
- Check hostname matches subdomain or custom_domain
- Inspect `tenants` table

#### 2. Authentication Errors
- Verify Supabase credentials in `.env.local`
- Check RLS policies are enabled
- Ensure user exists in `users` table with correct `tenant_id`

#### 3. Permission Denied
- Check user has role assigned in `user_roles`
- Verify role has required permissions
- Check RLS policies

#### 4. Feature Not Available
- Verify feature flag is enabled in `feature_flags`
- Check `enabled = true` for the module

### Debug Helpers

```typescript
// Log tenant info
console.log('Tenant:', tenant)
console.log('Tenant ID:', tenant?.id)
console.log('Config:', tenant?.config)

// Log user permissions
const permissions = await getUserPermissions(user.id)
console.log('User permissions:', permissions)

// Log features
const features = await getTenantFeatures(tenant.id)
console.log('Enabled features:', features)
```

## Next Steps

1. **Implement Auth Pages**: Create login, signup, password reset
2. **Build Admin UI**: Feature flag management, user management
3. **Add Modules**: Implement public booking, customer portal
4. **Integrate Payments**: Stripe checkout flows
5. **Email Templates**: Resend email templates
6. **Rate Limiting**: Implement with Upstash Redis
7. **Analytics**: Add tracking and reporting
8. **Testing**: Unit tests, integration tests, E2E tests

## Resources

- [Next.js Docs](https://nextjs.org/docs)
- [Supabase Docs](https://supabase.com/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
