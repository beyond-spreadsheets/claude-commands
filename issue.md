---
description: Work on GitHub issue with full workflow (e.g., /issue dkc400)
autoApprove:
  - Bash(gh issue view*)
  - Bash(git clone*)
  - Bash(cd*)
  - Bash(git checkout*)
  - Bash(git pull*)
  - Bash(git branch*)
  - Bash(npm install*)
  - Bash(npm run build*)
  - Bash(npm test*)
  - Bash(npm run test:ci*)
  - Bash(git add*)
  - Bash(git commit*)
  - Bash(git push*)
  - Bash(gh pr create*)
  - Bash(mkdir*)
  - Read
  - Glob
  - Grep
  - Edit
  - Write
---

You are working on a GitHub issue. The issue identifier is: {{arg1}}

Parse the identifier to extract:
- Repository shorthand (e.g., "dkc" from "dkc400")
- Issue number (e.g., "400" from "dkc400")

Repository mapping:
- "dkc" → beyond-spreadsheets/dkc_volunteer_portal

Workflow:
1. Use `gh issue view <issue-number> --repo beyond-spreadsheets/dkc_volunteer_portal` to fetch the issue details and understand the problem
2. Clone the repository into a temporary location and explore the codebase to understand relevant code before asking any questions. Specifically:
   - Find existing code related to the issue (search for relevant files, functions, database tables, API routes)
   - Understand the current implementation and architecture
   - Identify what already exists vs what needs to be built
   - Only ask questions about things you genuinely cannot determine from reading the code
3. Ask me up to 5 focused questions — only for information you could NOT find in the code. If you found everything you need, skip this step entirely.
4. Use the TodoWrite tool to create a task list for this issue workflow including all major steps
5. Think carefully about an appropriate folder name and branch name based on the issue title/description
6. Create a NEW folder with a descriptive name related to the issue (e.g., "dkc-fix-login-bug-400" or "dkc-add-email-validation-400")
7. Clone the repository into that new folder using `git clone <repo-url> <folder-name>`
8. Navigate into the new folder
9. Checkout develop branch and pull latest changes
10. Create and checkout a new branch with a descriptive name matching the folder name
11. Implement the fix based on your earlier codebase exploration
12. Run `npm run build` (or appropriate build command) to test the build
13. Run CI/CD tests locally (e.g., `npm test` or `npm run test:ci`)
14. Only if tests pass: commit changes, push to GitHub, and create a PR with a detailed description
15. Update the todo list as you progress through each step, marking tasks as in_progress and completed

IMPORTANT:
- Always create a NEW folder for this issue - never reuse existing repo folders
- Use the actual issue content to create meaningful folder and branch names
- Don't push or create PR if tests fail