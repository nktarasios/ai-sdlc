# Run Spec

Stand up and start an autonomous feature pipeline from a PM spec: $ARGUMENTS

## What this does

Builds a `pipeline-status.json` progress tracker, then registers a durable CronJob that
fires `/spec-conductor <slug>` on a 2-minute cadence. On each firing, the conductor consults
the tracker, launches one fresh background agent for every step that is ready, and tears
itself down once every feature reaches a terminal state. This skill only does the setup, 
it executes once and exits.

---

## Step 1: Parse Arguments

Pull from `$ARGUMENTS`:
- **pm-spec path** (required): full path to the pm-spec file
- `--feature <id>`: optional: restrict the run to this single feature (all others skipped)

Work out the **spec slug** from the path: it is the name of the directory holding the
`pm-spec/` folder (e.g. `thoughts/shared/specs/2026-06-22-example-feature-set/pm-spec/pm-spec.md`
→ slug = `2026-06-22-example-feature-set`).

---

## Step 2: Read the PM Spec

Open the pm-spec file. From its **Feature Implementation Order** table, capture:
- Every feature ID and title
- Each feature's `Depends On` entries

Also read the **Global Research Topics** section so you know which GRQ questions exist.

---

## Step 3: Create the Progress Tracker

Write `thoughts/shared/coding/<slug>/pipeline-status.json`. Should the file already exist,
read it first and leave alone any feature already past `waiting`: in-progress work must
survive a re-run.

```json
{
  "spec_path": "<pm-spec path>",
  "slug": "<slug>",
  "started_at": "<ISO timestamp>",
  "cron_job_id": null,
  "last_conductor_run": null,
  "features": {
    "global-research": {
      "title": "Global Research (all GRQ topics from pm-spec)",
      "pipeline": ["research"],
      "status": "research",
      "step_in_progress": false,
      "step_started_at": null,
      "gate_conditions": {},
      "docs": {},
      "error": null
    },
    "<feature-id>": {
      "title": "<Feature title from pm-spec>",
      "pipeline": ["research", "plan", "tests", "implement", "validate", "review"],
      "status": "research",
      "step_in_progress": false,
      "step_started_at": null,
      "gate_conditions": {},
      "docs": {},
      "pr_url": null,
      "error": null
    }
  }
}
```

**Status values:** `waiting` | `research` | `plan` | `tests` | `implement` | `validate` | `review` | `done` | `failed`

`status` names the step currently ready to run (or the one most recently run). Read together
with `step_in_progress`, it tells the conductor whether an agent should be spawned:
- `step_in_progress: false` + status not `waiting/done/failed` → spawn this step's agent
- `step_in_progress: true` → an agent is already on it, skip
- `waiting` → dependency gate still closed, skip
- `done` / `failed` → terminal, skip

**Gate conditions** take the form of a map `{ "<other-feature-id>": "done" | "plan_or_later" }`.
A feature may not start until every entry in its map is satisfied.

**Gate condition semantics:**
- `"done"` → the referenced feature carries status `"done"`
- `"plan_or_later"` → the referenced feature has moved beyond research (its status is `plan`,
  `tests`, `implement`, `validate`, `review`, or `done`)

**Setting gates from the spec:**

Translate the pm-spec's "Depends On" column into gate entries, choosing the right condition
per dependency. A worked example for a spec containing features F1–F5:
- `global-research`: `gate_conditions: {}` (no deps)
- `F1`: `gate_conditions: {}` (no deps, independent, free to start at once)
- `F2`: `gate_conditions: { "global-research": "done" }`
- `F3`: `gate_conditions: { "global-research": "done", "F1": "plan_or_later" }`
  (F3 leans on F1's design decisions, so F1 needs at minimum an approved plan)
- `F4`: `gate_conditions: {}` (no deps)
- `F5`: `gate_conditions: { "global-research": "done", "F1": "done", "F4": "done" }`
  (F5 stitches F1 and F4 together, so both have to be fully merged before it moves)

Give `status: "waiting"` to every feature whose `gate_conditions` map has entries. Give
`status: "research"` to features whose map is empty (nothing blocks them).

---

## Step 4: Create the Durable CronJob

Register a CronJob via CronCreate:
- `cron`: `"*/2 * * * *"` (every 2 minutes)
- `prompt`: `/spec-conductor <slug>`
- `durable`: `true` (survives Claude restarts)
- `recurring`: `true`

Record the returned job ID in the tracker:
```json
{ "cron_job_id": "<job-id-from-CronCreate>" }
```

---

## Step 5: Report to User

Print:
```
Pipeline launched for: <slug>
Progress tracker: thoughts/shared/coding/<slug>/pipeline-status.json

Features ready to start immediately (parallel):
  global-research, GRQ-1 through GRQ-N
  F1, <feature title>
  F4, <feature title>

Features waiting on gates:
  F2, waiting: global-research=done
  F3, waiting: global-research=done, F1>=plan
  F5, waiting: global-research=done, F1=done, F4=done

Conductor cron: every 2 min (durable, job ID: <id>)
Each step runs as a fresh zero-context agent. No daisy-chaining.

To monitor: /spec-status <slug>
To stop: delete cron job <id> and set step_in_progress: false for any running features
```

Within 2 minutes the conductor fires for the first time and starts spawning research agents.
This skill's work ends here, exit.
