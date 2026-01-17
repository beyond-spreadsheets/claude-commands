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
2. Ask me 5 questions to help define the issue
3. Use the TodoWrite tool to create a task list for this issue workflow including all major steps
4. Think carefully about an appropriate folder name and branch name based on the issue title/description
5. Create a NEW folder with a descriptive name related to the issue (e.g., "dkc-fix-login-bug-400" or "dkc-add-email-validation-400")
6. Clone the repository into that new folder using `git clone <repo-url> <folder-name>`
7. Navigate into the new folder
8. Checkout develop branch and pull latest changes
9. Create and checkout a new branch with a descriptive name matching the folder name
10. Investigate the issue thoroughly by exploring the codebase
11. If a fix is required:
   - Implement the fix
   - Run `npm run build` (or appropriate build command) to test the build
   - Run CI/CD tests locally (e.g., `npm test` or `npm run test:ci`)
   - Only if tests pass: commit changes, push to GitHub, and create a PR with a detailed description
12. Update the todo list as you progress through each step, marking tasks as in_progress and completed

IMPORTANT:
- Always create a NEW folder for this issue - never reuse existing repo folders
- Use the actual issue content to create meaningful folder and branch names
- Don't push or create PR if tests fail