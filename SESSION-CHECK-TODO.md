# Session Check Implementation - TODO

## Commands That Need Session Check Added

All `/pp-*` commands (except `/pp-setup` and `/pp-help`) should check for session before running.

### Commands to Update:

1. ✅ `/pp-newtenant` - DONE
2. ⏳ `/pp-page` - Add Step 0 session check
3. ⏳ `/pp-component` - Add Step 0 session check  
4. ⏳ `/pp-list` - Add Step 0 session check
5. ⏳ `/pp-issue` - Add Step 0 session check
6. ⏳ `/pp-diag` - Add Step 0 session check (but make it optional)
7. ⏳ `/pp-triage` - Add Step 0 session check

### Session Check Code Snippet:

Add this as "Step 0" at the beginning of each command:

```markdown
## Step 0: Verify Session Setup

**CRITICAL: Check if /pp-setup has been run first!**

\`\`\`bash
if [ ! -f ~/.claude-portal-session ]; then
  echo "❌ ERROR: Portal Platform session not configured"
  echo ""
  echo "You must run /pp-setup first to configure your work session."
  echo ""
  echo "Run: /pp-setup"
  echo ""
  echo "This will:"
  echo "  - Clone portal-platform repository"
  echo "  - Create tenant repository structure"
  echo "  - Configure session paths"
  echo ""
  exit 1
fi

# Load session variables
source ~/.claude-portal-session

echo "✅ Session configured"
echo "   Portal Platform: $PORTAL_PLATFORM"
echo ""
\`\`\`
```

### Special Cases:

- **`/pp-diag`**: Should allow running WITHOUT session (for diagnosing session problems)
  - Make the session check a warning, not an error
  - Still run diagnostics even if no session

- **`/pp-setup`**: No session check needed (it CREATES the session)

- **`/pp-help`** and `/pp`: No session check needed (informational only)

## Implementation Strategy:

1. Add session check to each command file
2. Test each command after adding check
3. Update all examples in documentation to mention running `/pp-setup` first
4. Create workflow diagram showing `/pp-setup` → other commands

## Documentation Updates Needed:

1. **README.md**: 
   - Change `/pp-tenant` → `/pp-newtenant`
   - Add note: "Run `/pp-setup` first before any `/pp-*` commands"

2. **PP-HELP.md**:
   - Update command list
   - Emphasize `/pp-setup` as first step

3. **PP-COMMANDS-README.md**:
   - Update all workflow examples
   - Show `/pp-setup` as step 1

4. **WORKFLOW.md**:
   - Update tenant creation workflow
   - Use `/pp-newtenant` instead of `/pp-tenant`

## Testing Checklist:

- [ ] Try `/pp-newtenant` without running `/pp-setup` - should fail with clear message
- [ ] Try `/pp-page` without session - should fail  
- [ ] Run `/pp-setup` then `/pp-newtenant` - should work
- [ ] Verify all commands can access `$PORTAL_PLATFORM` variable after loading session

