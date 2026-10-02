# Abandon

Tear down the leftovers of a failed or unwanted autopilot run: $ARGUMENTS

## Instructions

Your task is to release the resources held by an autopilot run that either failed or is simply no longer wanted.

### 1. Parse Arguments

The argument arrives in one of three forms:
- A plan path (e.g., `thoughts/shared/plans/2026-03-20-example-feature.md`)
- A branch name (e.g., `autopilot/example-feature`)
- A slug (e.g., `example-feature`)

### 2. Find Resources

Starting from the argument, track down:
1. **Status file**: `thoughts/shared/autopilot/<slug>-status.json` -- open it to learn the branch name and worktree path
2. **Worktree**: look for a match in `git worktree list`
3. **Branch**: check `git branch --list autopilot/<slug>`, or use the branch the status file names
4. **Log file**: `thoughts/shared/autopilot/<slug>.log`

### 3. Confirm

Present exactly what is about to be removed:

```
ABANDON: <slug>
  Worktree: <path> (will be removed)
  Branch:   <branch> (will be deleted)
  Status:   <path> (will be updated to "abandoned")
  Log:      <path> (will be preserved)
```

Wait for the user's confirmation before doing anything.

### 4. Clean Up

Proceed in this exact order, the steps depend on one another:
1. Remove the git worktree: `git worktree remove <path> --force`
2. Delete the branch: `git branch -D <branch>`
3. Update the status file: set `status` to `"abandoned"`
4. Leave the log file alone (it's the record of what went wrong)

### 5. Report

```
Abandoned: <slug>
  Worktree removed, branch deleted, status updated.
  Log preserved at: thoughts/shared/autopilot/<slug>.log
```
