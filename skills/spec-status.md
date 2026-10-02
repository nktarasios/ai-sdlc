# Spec Status

Report where a running spec's pipeline currently stands: $ARGUMENTS

## Instructions

Strictly read-only: spawn no agents, modify no files.

### Step 1: Read Tracker

The argument is either a pm-spec path or the spec slug itself.
To derive the slug from a path containing `pm-spec/`, take the parent directory's name.

Read: `thoughts/shared/coding/<slug>/pipeline-status.json`

### Step 2: Report

```
Pipeline: <slug>
Started: <started_at>
Last conductor run: <last_conductor_run>
Cron job: <cron_job_id>

Feature                              | Step       | In-Flight | PR
-------------------------------------|------------|-----------|-----
global-research: GRQ-1–N             | done       |           |
Feature 1: <title>                   | review     | yes       |, 
Feature 2: <title>                   | implement  |           |, 
Feature 3: <title>                   | waiting    |           | waiting: global-research=done
Feature 4: <title>                   | waiting    |           | waiting: global-research=done, F1>=plan
Feature 5: <title>                   | waiting    |           | waiting: global-research=done, F1=done, F4=done

Failed:
  None
```

Step values in order: research → plan → tests → implement → validate → review → done
In-Flight = "yes" whenever step_in_progress is true (include the age: "<N> min ago" computed from step_started_at)

### Step 3: Report Failures

Every feature carrying `status: "failed"` gets this block:

```
FAILED, needs human input:
  <feature-id> (<title>)
  Error: <error field>

  To resume:
    1. Fix the issue (the error message tells you which step and why)
    2. Edit pipeline-status.json: set features.<id>.status = "<the failed step>", step_in_progress = false
    3. The conductor will pick it up within 2 minutes automatically
    4. Or force immediately: /spec-conductor <slug>
```
