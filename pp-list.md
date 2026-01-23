---
description: List all tenants in the database
autoApprove:
  - Bash(node scripts/list-all-tenants.mjs)
---

List all tenants in your multi-tenant SaaS platform.

## What This Does

Simply runs the existing script to show all tenants from the database.

## Instructions

**Step 1: Check if Portal Platform is set up**

```bash
if [ -f ~/.claude-portal-path ]; then
  PORTAL_PATH=$(cat ~/.claude-portal-path)
  cd "$PORTAL_PATH"
else
  echo "❌ Portal Platform not set up for this session"
  echo "Run: /pp-setup"
  exit 1
fi
```

**Step 2: Run the script**

```bash
node scripts/list-all-tenants.mjs
```

**Step 3: Done!**

Show the output to the user. That's it!

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
