# Status

Display autopilot pipeline state plus recent activity: $ARGUMENTS

## Instructions

Your task is to surface the current state of every autopilot run alongside what's been happening in the project lately.

### 1. Autopilot Status

Read every JSON file matching `thoughts/shared/autopilot/*-status.json` and render each one:

```
AUTOPILOT STATUS
================
  <plan-slug>    <current_step> (<step_index>/<steps_total>)    <status>    <elapsed>
  <plan-slug>    <current_step> (<step_index>/<steps_total>)    <status>    <elapsed>
```

Status values: `running`, `completed`, `merged`, `failed`, `merge_failed`, `revise`

When there are no status files at all, report "No autopilot runs found."

Any entry in `failed` or `revise` state additionally gets:
- Its error message
- Its resume command

### 2. Git Worktrees

Run `git worktree list` and show what's active. Call out any worktree lacking a corresponding status file, those are orphans.

### 3. Recent Git Activity

Run `git log --oneline -10` for the ten latest commits on the current branch.

### 4. Open Plans

Look through `thoughts/shared/plans/` for plan docs still carrying `NOT STARTED` phases, that's unfinished work. List each one alongside its next incomplete phase.

### 5. Format

Lay it all out so it can be scanned at a glance. Brevity matters here, this is a dashboard, not a write-up.
