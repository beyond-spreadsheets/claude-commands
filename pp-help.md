---
description: Show complete help guide for Portal Platform commands
autoApprove:
  - Bash(cat *)
  - Read
---

You are showing the complete help guide for Portal Platform commands.

## Display Help File

Read and display the central help file:

```bash
if [ -f ~/.claude/commands/PP-HELP.md ]; then
  cat ~/.claude/commands/PP-HELP.md
elif [ -f /Users/George/claude-commands/PP-HELP.md ]; then
  cat /Users/George/claude-commands/PP-HELP.md
else
  echo "❌ PP-HELP.md not found"
  echo ""
  echo "This file should be in ~/.claude/commands/ or /Users/George/claude-commands/"
  exit 1
fi
```

After displaying the help file, add a footer:

```bash
echo ""
echo "═══════════════════════════════════════════"
echo "   QUICK START"
echo "═══════════════════════════════════════════"
echo ""
echo "New to Portal Platform? Start here:"
echo ""
echo "1. Setup session:           /pp-setup"
echo "2. Create your first tenant: /pp-tenant \"Client Name\""
echo "3. Create pages:            /pp-page <slug> <page-name>"
echo "4. View context anytime:    /pp"
echo "5. Diagnose problems:       /pp-diag"
echo ""
echo "Need help choosing a command? Ask and I'll suggest the right one!"
echo ""
