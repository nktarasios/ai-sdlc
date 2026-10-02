# Catchup

Fast orientation on the project's current state for a fresh session: $ARGUMENTS

## Instructions

Your task is to get a brand-new session oriented on where the project stands. Keep it tight, think 30-second briefing, not status report.

### 1. Active Autopilots

Read every file matching `thoughts/shared/autopilot/*-status.json`. Per entry:
- Show: slug, current step, status, elapsed time
- On failure: include the error and the resume command
- On merge: include when it finished

### 2. Recent Activity

Run `git log --oneline -10` for the ten latest commits, calling out any that came from an autopilot merge.

### 3. Open Plans

Look through `thoughts/shared/plans/*.md` for plans carrying `NOT STARTED` phases, that's in-flight work. Show each plan's name and its next incomplete phase.

### 4. Recent Retrospectives

Look in `thoughts/shared/retrospectives/` for anything written within the last 7 days. Where found, surface just the takeaways (the "Lessons for Future Plans" section only).

### 5. Memory Context

Pull in any recent project or feedback memories with bearing on today's work.

### 6. Output Format

```
SESSION BRIEFING
================

AUTOPILOTS
  <slug>    <step> (<N>/<total>)    <status>    <time>

RECENT COMMITS (last 10)
  <hash> <message>

OPEN PLANS
  <plan>    next: <phase>

RECENT LESSONS
  - <lesson from retrospective>

Ready to work. Suggested next actions:
  - <action based on current state>
```

Hold the whole thing under 30 lines. This is a glance, nothing deeper. When the user wants detail on any piece of it, `/status`, `/prior_art`, or the underlying files are right there.
