---
name: commit
description: Stages changed files, commits with an auto-generated ticket-prefixed message, and pushes to the remote branch.
allowed-tools: Bash(git *)
model: haiku
---

# Your Task

1. Get the current branch name: `git rev-parse --abbrev-ref HEAD`
2. Extract the ticket number from the branch name. The ticket follows the pattern `[A-Z]+-[0-9]+` and may appear as:
   - The full branch name (e.g. `MS-860`)
   - A prefix segment (e.g. `MS-860/Part-1`)
   - After a slash (e.g. `feature/MS-860`)
4. Get the staged diff summary: `git diff --stat` and `git status --short`
5. Stage all changed files: `git add -A`
6. Write a commit message:
   - **Title:** `<TICKET> <short imperative summary>` (e.g. `MS-860 Add commit skill to Claude Code`)
   - **Body:** one brief sentence describing what changed
   - Derive both from the diff — do not ask the user
7. Commit using a HEREDOC to avoid shell escaping issues

> If no ticket number is found in the branch name, omit it and use just the short summary as the title.
