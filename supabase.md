---
description: Connect to Supabase using proven method
autoApprove:
  - Read
  - Glob
  - Grep
  - Edit
  - Write
  - Bash(node scripts/*)
  - Bash(npm install*)
---

You are helping connect to Supabase. Use the **Node.js script method** - this is the 100% reliable approach.

## THE PROVEN METHOD: Node.js Scripts

**Step 1: Verify Environment**
```bash
# Check .env.local has Supabase credentials
grep "SUPABASE" .env.local
```

Must have:
- `NEXT_PUBLIC_SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY`

**Step 2: Create a Script**
```javascript
#!/usr/bin/env node
const { createClient } = require('@supabase/supabase-js')
require('dotenv').config({ path: '.env.local' })

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY
)

async function main() {
  const { data, error } = await supabase
    .from('table_name')
    .select('*')

  if (error) {
    console.error('❌ Error:', error)
    process.exit(1)
  }

  console.log('✅ Data:', data)
}

main().then(() => process.exit(0))
```

**Step 3: Run It**
```bash
node scripts/your-script.js
```

## Key Points

1. **Use Service Role Key** - Full access, bypasses RLS
2. **Never use MCP tools** - They return "Unauthorized"
3. **Never use REST API** - Not supported for SQL
4. **Always use `portal.` schema prefix** (if using portal schema)
5. **Run from project root** - So `.env.local` loads correctly

## Common Script Patterns

**Query:**
```javascript
const { data, error } = await supabase
  .from('table_name')
  .select('*')
  .eq('column', 'value')
```

**Insert:**
```javascript
const { data, error } = await supabase
  .from('table_name')
  .insert({ column: 'value' })
  .select()
```

**Update:**
```javascript
const { data, error } = await supabase
  .from('table_name')
  .update({ column: 'new_value' })
  .eq('id', id)
```

**Delete:**
```javascript
const { data, error } = await supabase
  .from('table_name')
  .delete()
  .eq('id', id)
```

## Troubleshooting

**"Cannot find module '@supabase/supabase-js'"**
```bash
npm install
```

**"Unauthorized" or "Invalid API key"**
```bash
cat .env.local | grep SUPABASE
# Verify keys match Supabase dashboard
```

**"relation 'table' does not exist"**
- Add schema prefix: `portal.table_name` (if using custom schema)

Always create scripts in `scripts/` directory and run with `node scripts/name.js`
