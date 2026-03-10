---
name: git-worktree
description: Manages Git worktrees for isolated parallel development. Creates branches (linking ticket IDs if available), copies local environment files, and pushes changes to the remote.
---

# GitHub Worktree Workflow

You manage Git worktrees to allow isolated, parallel development. This skill ensures your development environment remains clean while maintaining ticket/issue trackers as the source of truth when applicable.

## Core Mandates

1. **Repository Root First**: Always operate from the repository root that contains `.git`. Many directories may be standalone repos, so `cd` into the correct one first.
2. **Issue Tracker is Truth**: Keep your issue tracker (like Jira, Linear, etc.) as the source of truth. Do NOT save separate local task files.
3. **Context Accuracy**: Always keep the "Relevant Files" section accurate and up to date in your context.
4. **Base Branch**: Every feature or bug branch MUST be based on the `development` branch. All PRs must target `development`.
5. **No Script Overkill**: Use standard Git commands to manage the worktree lifecycle.

## Step-by-Step Workflow

### 1. Initialize the Worktree Environment
Before starting new work, ensure the root directory is prepared for worktrees:

```bash
# From the repository root containing .git
mkdir -p worktrees
# Ensure worktrees/ is ignored in the git repository
grep -q "^worktrees/" .gitignore || echo "worktrees/" >> .gitignore
```

### 2. Create the Worktree
Use the `git worktree` command to create both the branch and its worktree. It is good practice to include the ticket ID in the branch name if one is available.

```bash
git fetch origin development
git worktree add worktrees/feature/<ticket-id> -b feature/<ticket-id> origin/development
```
*(Use `bug/<ticket-id>` if it is a bug fix instead of a feature).*

### 3. Copy Local Environment Files (CRITICAL)
Immediately after creating the worktree, evaluate the main repository root for files that are required for local development and testing but are excluded from version control (e.g., `.env` files, local configurations, or credentials). Copy these files from the original repository into the new worktree to ensure consistency and a functional test environment.

```bash
# 1. Identify local files that are ignored by git but exist in the root
# Ignore obvious build artifacts like node_modules or dist
git clean -ndX | grep -vE "node_modules|dist|build|\.next|\.cache"

# 2. Copy identified files (e.g., .env files) into the worktree
# Note: Ensure you are in the repository root when running this
for file in .env*; do
  if [ -f "$file" ]; then
    cp "$file" "worktrees/feature/<ticket-id>/$file"
  fi
done
# Repeat this pattern for other necessary local files identified in step 1
```

### 4. Work and Iterate
Navigate into the newly created worktree and perform all development and testing there.

```bash
cd worktrees/feature/<ticket-id>
# Perform your file modifications, tests, builds, etc.
```

### 5. Commit and Push
Once the work is complete and tested, push the changes from inside the worktree.

```bash
git add .
git commit -m "feat: <description> (Resolves #<ticket-id>)"
git push -u origin feature/<ticket-id>
```

### 6. Finding Existing Worktrees
If you need to return to a worktree created previously (e.g., for a walkthrough or finalization):

```bash
# List all active worktrees
git worktree list
# Look for the path containing the ticket-id
cd worktrees/feature/<ticket-id>
```


