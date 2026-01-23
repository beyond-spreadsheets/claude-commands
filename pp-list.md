---
description: List all tenants in the database
autoApprove:
  - Bash(node scripts/list-all-tenants.mjs)
---

List all tenants in your multi-tenant SaaS platform.

## What This Does

Simply runs the existing script to show all tenants from the database.

## Instructions

**IF** the script `scripts/list-all-tenants.mjs` exists:
- Run it: `node scripts/list-all-tenants.mjs`
- Show the output to the user
- Done!

**IF** the script doesn't exist:
- Tell the user: "The list-all-tenants.mjs script doesn't exist yet. Create it with the Portal Platform setup first."
- Suggest: "Or create tenants with: /pp-tenant \"Business Name\""

That's it! No complex logic, no file searching, just run the script.

## Expected Output

```
═══════════════════════════════════════════
   PORTAL PLATFORM - TENANT OVERVIEW
═══════════════════════════════════════════

📋 Found 4 tenant(s)

1. Beyond Spreadsheets
   Slug: beyond-spreadsheets
   Subdomain: https://beyond-spreadsheets.yourdomain.com
   Custom Domain: https://beyond-spreadsheets.co.uk
   Tenant ID: b2ea9ee0-eedd-4eb6-8fa3-18b64ab79edf

2. AYUP
   Slug: ayup
   Subdomain: https://ayup.yourdomain.com
   ...
```

Done. Simple. Fast.
