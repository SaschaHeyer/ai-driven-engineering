---
description: Autonomously executes the implementation plan (code, test, commit).
---

You are running the `/implement` custom command. Follow this protocol exactly. This workflow is fully autonomous—execute continuously without pausing for human confirmation unless a blocker prevents progress. Stay inside this command loop until every task is complete or you encounter a blocker you explicitly surface.

## 0. Searches docs/solutions/ for relevant past solutions by.
Use before implementing features or fixing problems to surface knowledge and prevent repeated mistakes.

The `docs/learnings/` directory contains documented solutions.
There might be hundret of files use effective strategy that minimizes tool calls such as

- Extract keywords from feature and description
- Use grep with those keywords to find files.
- Read the conent of those files

## 1. Establish context
   - Ask for the Linear issue key if it is not already known.
   - Before touching git, confirm you are inside the repo or worktree that owns the ticket. Use `git rev-parse --show-toplevel` to identify the current root. If you are not inside a git repository, scan upward and across known sibling repos until you locate the correct root, then `cd` there before continuing.
   - Fetch the current issue description via the Linear MCP.
   - Locate the `## PLAN` section. If it is missing or empty, look for common fallbacks such as `## Tasks`, `## Implementation Plan`, or a `## Bug Brief` with embedded checklists. If a fallback exists, treat it as the plan source without renaming it; otherwise stop and instruct the user to generate it with `/tasks` first.
   - Parse the plan into Relevant Files, Notes, and the checkbox task list.

## 2. Determine the next sub-task
   - Find the first unchecked sub-task in order. If a parent task has no sub-tasks, treat the parent itself as the actionable item.

## 3. Execute the sub-task
   - Carry out the implementation instructions for sub-task right away.
   - If critical context is missing, document the assumption you will proceed with, surface it in your status message, and keep moving.
   - Track any files you touch so you can update the Relevant Files list later.
   - Adhere to project conventions from AGENTS.md / GEMINI.MD files in scope.

## 4. Update Linear immediately after each sub-task
- Mark the completed sub-task checkbox to `[x]` inside the same section you are using for the plan (do not rename existing headers).
- If this completion introduces new work, add fresh sub-tasks beneath the appropriate parent.
- Update the Relevant Files list with one-line explanations for each affected file (create the entry if missing).
- If you created or switched worktrees, add or update a short "Worktrees" note in the plan with the relative paths you are using so the ticket records where the branch lives.
- Use the Linear MCP to write the revised `## PLAN` back to the issue.
- Optionally leave a brief Linear comment summarizing the progress.

## 5. Parent-task completion protocol
   - After marking a sub-task `[x]`, check whether all sibling sub-tasks under the same parent are now `[x]`.
   - If they are, follow this sequence before checking the parent task:
     1. Run the full test suite appropriate for the repo (e.g., `npm test`, `pytest`, `bin/rails test`).
     2. Stage changes (`git add` for all relevant files).
     3. Remove temporary files or debugging artifacts.
     4. Commit using a multi-`-m` command in conventional commit format, summarizing the parent task and referencing the Linear task/PRD. Example:
        ```bash
        git commit -m "feat: add payment validation logic" -m "- Validates card type and expiry" -m "- Adds unit tests for edge cases" -m "Related to T123 in PRD"
        ```
   - After the commit, update the plan to mark the parent task `[x]` (since all its subtasks are now complete) and push the revised plan to Linear.

## 6. Repeat until completion
   - Continue the loop (steps 2–6) without exiting the command until every task and sub-task in the plan reads `[x]` or a blocker prevents further progress.
   - When everything is complete, confirm that the Linear issue status should move forward.
   - **IMPORTANT**: Recommend the user to run `/workflow:walkthrough` to generate a visual summary of the work before moving to `/workflow:finalize`.

## 7. Learnings:
Documented learnings that might apply using
skill: document-learnings

IMPORTANT:
Use a worktree**
   ```bash
   skill: git-worktree
   # The skill will create a new branch from the default branch in an isolated worktree.
   # Evaluate the codebase for local files that might be needed (like .env) and ensure 
   # they are copied from the original repo into the new worktree.
   ```