# Portal Platform (PP) Commands - Complete Guide

**Your Multi-Tenant SaaS Command Suite**

These commands help you build and manage your full SaaS platform where:
- ✅ Infrastructure is shared (auth, email, payments)
- ✅ Each tenant has unique pages and design
- ✅ Security updates push to all tenants instantly
- ✅ You can scale to 100+ clients

## 📦 Installation

Commands are installed in: `~/.claude/commands/pp*.md`

## 🎯 Commands Overview

| Command | Purpose | Example |
|---------|---------|---------|
| `/pp-help` | Show help guide | `/pp-help` |
| `/pp-tenant` | Create new tenant | `/pp-tenant "FitZone Gym"` |
| `/pp-page` | Create tenant page | `/pp-page fitzone home` |
| `/pp-component` | Create shared component | `/pp-component HeroSection` |
| `/pp-list` | List everything | `/pp-list` |

## 🚀 Quick Start: Build Your Website

**Step 1: Create your tenant**
```bash
/pp-tenant "Beyond Spreadsheets"
```

**Step 2: Create your pages**
```bash
/pp-page beyond-spreadsheets home
/pp-page beyond-spreadsheets about
/pp-page beyond-spreadsheets pricing
/pp-page beyond-spreadsheets contact
/pp-page beyond-spreadsheets blog
```

**Step 3: Create shared components**
```bash
/pp-component HeroSection
/pp-component PricingCard
/pp-component CTAButton
```

**Step 4: Test locally**
```bash
# Edit /etc/hosts:
127.0.0.1 beyond-spreadsheets.localhost

# Run dev server:
npm run dev

# Visit:
http://beyond-spreadsheets.localhost:3000
```

**Step 5: Deploy**
```bash
npm run build
vercel deploy --prod
```

**Result:** Your website is live with full SaaS infrastructure ready!

## 🎯 Quick Start: Onboard a Client

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

**Step 3: They inherit infrastructure**
- ✅ Auth system (your code)
- ✅ Email system (your code)
- ✅ Payments (your code)
- ✅ Admin dashboard (your code)

**Time to onboard:** ~30 minutes (vs weeks building from scratch!)

## 📖 Architecture

```
portal-platform/
├── lib/                           # SHARED (all tenants)
│   ├── supabase/                 # Database client
│   ├── email/                    # Email service
│   ├── sms/                      # SMS service
│   └── payments/                 # Stripe integration
│
├── middleware.ts                  # SHARED security
│
├── components/
│   └── shared/                    # SHARED components
│       ├── HeroSection.tsx       # Reusable across tenants
│       ├── BookingWidget.tsx     # Tenant-aware styling
│       └── Navigation.tsx        # Customizable per tenant
│
└── app/
    ├── (tenants)/                 # UNIQUE per tenant
    │   ├── beyond-spreadsheets/  # Your website
    │   │   ├── layout.tsx        # Your design system
    │   │   ├── page.tsx          # Home
    │   │   ├── about/page.tsx
    │   │   └── pricing/page.tsx
    │   │
    │   ├── dkc/                   # DKC volunteer portal
    │   │   ├── layout.tsx        # DKC branding
    │   │   ├── page.tsx
    │   │   └── volunteer-application/page.tsx
    │   │
    │   └── fitzone-gym/           # FitZone client
    │       ├── layout.tsx        # FitZone branding
    │       ├── page.tsx
    │       └── classes/page.tsx
    │
    └── admin/                     # SHARED dashboards
        ├── bookings/             # All tenants use this
        ├── customers/            # Data filtered by tenant_id
        └── settings/             # Tenant-specific settings
```

## 🔒 Security Model

**Each command enforces:**

1. **Tenant Isolation**
   - Every database query includes `.eq('tenant_id', tenant.id)`
   - RLS policies at database level
   - No cross-tenant data access possible

2. **File Isolation**
   - Each tenant has own folder: `app/(tenants)/<slug>/`
   - Working on Tenant A cannot break Tenant B
   - Layout files enforce tenant check

3. **Shared Infrastructure**
   - Auth, email, SMS, payments shared
   - Bug fix in lib/ helps ALL tenants
   - Security update deploys to everyone

## 💡 Use Cases

### Use Case 1: Your Own Website
```bash
/pp-tenant "Beyond Spreadsheets"
/pp-page beyond-spreadsheets home
/pp-page beyond-spreadsheets about
# ... 20 pages total
```

### Use Case 2: Simple Client (4 pages)
```bash
/pp-tenant "Ayup Kids"
/pp-page ayup home
/pp-page ayup activities
/pp-page ayup book
/pp-page ayup about
```

### Use Case 3: Complex Client (30+ pages)
```bash
/pp-tenant "Derby Kids Camp"
/pp-page dkc home
/pp-page dkc volunteer-application
/pp-page dkc weeks
/pp-page dkc references
/pp-page dkc training
# ... 25 more pages
```

### Use Case 4: Rapid Onboarding
```bash
# Client 100 - takes 1 hour, not 1 month!
/pp-tenant "New Gym 100"
/pp-page new-gym-100 home
/pp-page new-gym-100 classes
/pp-page new-gym-100 booking

# They instantly get:
# ✅ Auth system
# ✅ Email system
# ✅ Payment processing
# ✅ Admin dashboard
# ✅ Customer management
# ✅ All your latest features!
```

## 🎨 Customization

**Each tenant can have:**
- Unique logo
- Unique colors (primary/secondary)
- Unique pages
- Unique navigation
- Unique features enabled/disabled

**But they share:**
- Authentication system
- Email infrastructure
- Payment processing
- Admin dashboard logic
- Security updates

## 📊 Scalability

**With this architecture:**

| Metric | Traditional | Multi-Tenant SaaS |
|--------|------------|-------------------|
| Time to onboard Client 1 | 4 weeks | 4 weeks |
| Time to onboard Client 2 | 4 weeks | 1 hour ⚡ |
| Time to onboard Client 100 | 4 weeks | 1 hour ⚡ |
| Bug fix deployment | 100 repos | 1 repo ✅ |
| Security update | Manual × 100 | Deploy once ✅ |
| Code maintenance | 100 codebases | 1 codebase ✅ |

## 🔧 Common Workflows

**Morning: Build your site**
```bash
/pp-tenant "Beyond Spreadsheets"
/pp-page beyond-spreadsheets home
/pp-page beyond-spreadsheets pricing
/pp-component PricingCard
```

**Afternoon: Onboard Client A**
```bash
/pp-tenant "FitZone Gym"
/pp-page fitzone-gym home
/pp-page fitzone-gym booking
# Uses your PricingCard component!
```

**Evening: Onboard Client B**
```bash
/pp-tenant "Salon Glam"
/pp-page salon-glam home
/pp-page salon-glam services
# Also uses PricingCard - no duplication!
```

**Next Day: Fix bug in PricingCard**
```bash
# Edit components/shared/PricingCard.tsx
# Deploy once
# ✅ All 3 sites fixed instantly!
```

## 📚 Documentation

Each command has comprehensive help:
- `/pp-help` - Main help guide
- `/pp-tenant` - Tenant creation guide
- `/pp-page` - Page creation guide
- `/pp-component` - Component creation guide
- `/pp-list` - Platform overview

## 🎓 Learning Path

**Week 1: Learn the basics**
1. Read `/pp-help`
2. Create your own tenant
3. Create 5 pages for yourself
4. Deploy and test

**Week 2: Build shared components**
1. Identify common patterns
2. Extract to shared components
3. Use across your pages

**Week 3: Onboard first client**
1. Create their tenant
2. Build their pages
3. Use your shared components
4. Deploy to production

**Week 4: Scale**
1. Onboard 2-3 more clients
2. Refine shared components
3. Add features to lib/
4. See how fast it is!

## ⚠️ Important Rules

**DO:**
- ✅ Create tenant-specific pages in `app/(tenants)/<slug>/`
- ✅ Use shared components from `components/shared/`
- ✅ Always include `tenant_id` filter in queries
- ✅ Use `/pp-tenant` before `/pp-page`
- ✅ Test locally before deploying

**DON'T:**
- ❌ Hardcode tenant data in components
- ❌ Create pages outside tenant folders
- ❌ Skip tenant_id filter in database queries
- ❌ Mix tenant-specific code in lib/
- ❌ Forget to test tenant isolation

## 🆘 Troubleshooting

**Command not found**
```bash
ls ~/.claude/commands/pp*.md
# Should show 5 files
# Restart Claude Code if needed
```

**Tenant not found**
```bash
/pp-list
# Shows all tenants
# Create with /pp-tenant if missing
```

**Page not working**
```bash
# Check file exists:
ls app/\(tenants\)/<slug>/<page>/page.tsx

# Check tenant detection:
# Add console.log in page to debug
```

**Database errors**
```bash
# Check credentials:
cat .env.local | grep SUPABASE

# Test connection:
node scripts/list-all-tenants.mjs
```

## 🎯 Success Metrics

**You'll know it's working when:**
- ✅ You can onboard a client in < 1 hour
- ✅ Bug fix deploys to all tenants instantly
- ✅ New feature available to everyone immediately
- ✅ Each tenant looks completely different
- ✅ Zero cross-tenant data leakage
- ✅ You can manage 50+ tenants easily

## 🚀 Next Steps

1. **Build your site first**
   - Use `/pp-tenant` and `/pp-page`
   - Get comfortable with the workflow

2. **Onboard a test client**
   - Create demo tenant
   - Build a few pages
   - Verify isolation works

3. **Go live**
   - Deploy to Vercel
   - Configure domains
   - Onboard real clients

4. **Scale**
   - Onboard more clients quickly
   - Build library of shared components
   - Push features to everyone at once

## 📖 Additional Resources

- **howToGuide.md** - Tenant setup details
- **CLAUDE.md** - Technical architecture
- **MULTI_TENANT_ROADMAP.md** - Implementation plan
- **dkc_portal_analysis.md** - Complex tenant example

---

**Version:** 1.0
**Created:** January 23, 2026
**Commands:** 5
**Total Size:** ~48 KB of documentation

**You're ready to build a scalable SaaS business!** 🚀

Start with: `/pp-help`
