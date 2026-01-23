---
description: Setup Portal Platform for this work session
autoApprove:
  - Bash(git clone:*)
  - Bash(git pull:*)
---

Setup Portal Platform repository for this Claude Code work session.

## What This Does

Pulls the portal-platform repo to a unique folder for this work session, allowing you to:
- Work on multiple clients simultaneously in different Claude sessions
- Keep work isolated per session
- Have all scripts and tools available

## Instructions

**Step 1: Determine work folder**

Ask the user: "What are you working on today?"

Examples:
- "my own website" → folder: `~/portal-work/beyond-spreadsheets`
- "FitZone Gym client" → folder: `~/portal-work/fitzone-gym`
- "DKC volunteer portal" → folder: `~/portal-work/dkc`
- "testing" → folder: `~/portal-work/test-{timestamp}`

**Step 2: Clone or update repo**

```bash
WORK_FOLDER="~/portal-work/{slug-or-project-name}"

# Check if folder exists
if [ -d "$WORK_FOLDER" ]; then
  echo "📂 Found existing portal-platform at $WORK_FOLDER"
  cd "$WORK_FOLDER"
  git pull origin main
  echo "✅ Updated to latest version"
else
  echo "📦 Cloning portal-platform to $WORK_FOLDER"
  git clone https://github.com/beyond-spreadsheets/portal-platform.git "$WORK_FOLDER"
  cd "$WORK_FOLDER"
  echo "✅ Portal Platform cloned"
fi
```

**Step 3: Install dependencies (if needed)**

```bash
if [ ! -d "node_modules" ]; then
  echo "📦 Installing dependencies..."
  npm install
fi
```

**Step 4: Check environment**

```bash
if [ ! -f ".env.local" ]; then
  echo "⚠️  No .env.local found"
  echo "📋 Copy from .env.example:"
  echo "   cp .env.example .env.local"
  echo "   Then add your Supabase credentials"
else
  echo "✅ Environment configured"
fi
```

**Step 5: Store the path**

Create a file to remember this session's portal path:

```bash
echo "$WORK_FOLDER" > ~/.claude-portal-path
echo "✅ Portal Platform ready at: $WORK_FOLDER"
```

**Step 6: Summary**

Show the user:

```
═══════════════════════════════════════════
   PORTAL PLATFORM - SESSION SETUP
═══════════════════════════════════════════

✅ Portal Platform: {WORK_FOLDER}
✅ Scripts available: scripts/list-all-tenants.mjs
✅ Other commands can now find this installation

Next steps:
- List tenants: /pp-list
- Create tenant: /pp-tenant "Business Name"
- Create page: /pp-page {slug} {page-name}

Working directory: {WORK_FOLDER}
```

## How Other Commands Use This

All other PP commands should check for the portal path:

```bash
# In pp-list, pp-tenant, pp-page, etc:
if [ -f ~/.claude-portal-path ]; then
  PORTAL_PATH=$(cat ~/.claude-portal-path)
  cd "$PORTAL_PATH"
  # Now run scripts, create files, etc.
else
  echo "❌ Portal Platform not set up"
  echo "Run: /pp-setup"
  exit 1
fi
```

## Multiple Sessions

Each Claude Code session can have its own portal-platform folder:

**Session 1: Building your website**
```
~/portal-work/beyond-spreadsheets/
```

**Session 2: Working on FitZone client**
```
~/portal-work/fitzone-gym/
```

**Session 3: Debugging DKC portal**
```
~/portal-work/dkc/
```

This prevents conflicts and keeps work organized!

## Expected Output

```
What are you working on today?

User: "my website"

📦 Cloning portal-platform to ~/portal-work/beyond-spreadsheets
Cloning into '~/portal-work/beyond-spreadsheets'...
✅ Portal Platform cloned
📦 Installing dependencies...
added 324 packages in 12s
✅ Environment configured
✅ Portal Platform ready at: ~/portal-work/beyond-spreadsheets

═══════════════════════════════════════════
   PORTAL PLATFORM - SESSION SETUP
═══════════════════════════════════════════

✅ Portal Platform: ~/portal-work/beyond-spreadsheets
✅ Scripts available: scripts/list-all-tenants.mjs
✅ Other commands can now find this installation

Next steps:
- List tenants: /pp-list
- Create tenant: /pp-tenant "Business Name"
- Create page: /pp-page beyond-spreadsheets home

Working directory: ~/portal-work/beyond-spreadsheets
```

## Notes

- The path is stored in `~/.claude-portal-path` (session-specific)
- Each Claude Code instance can have its own path
- Other PP commands read this file to know where to work
- You can manually change it: `echo "/path/to/portal" > ~/.claude-portal-path`
