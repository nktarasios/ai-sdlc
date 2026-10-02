# Spec Conductor

Polling conductor for a live feature pipeline: $ARGUMENTS

## What this does

Loads `pipeline-status.json`, evaluates dependency gates, launches one fresh background
agent for every step that has become ready, and relies on the durable CronJob to bring
itself back. Once every feature is terminal, it deletes the CronJob and exits.

The CronJob created by `/run-spec` invokes this skill every 2 minutes. Manual invocation
is only for forcing a check ahead of schedule.

---

## Step 1: Locate and Read the Tracker

Take the **slug** from `$ARGUMENTS`.

Read: `thoughts/shared/coding/<slug>/pipeline-status.json`

Keep hold of:
- `slug`: the spec slug
- `spec_path`: path to the pm-spec file
- `cron_job_id`: the CronJob to cancel at completion
- Every feature entry

Set `last_conductor_run` to the current ISO timestamp and write the tracker back out.

---

## Step 2: Check for Timed-Out Steps

For each feature showing `step_in_progress: true`:
- Work out how long it has been running from `step_started_at`
- Past 45 minutes, declare the feature failed:
  ```json
  {
    "status": "failed",
    "step_in_progress": false,
    "step_started_at": null,
    "error": "Step timed out after 45 minutes"
  }
  ```

Write the tracker once any timeouts have been applied.

---

## Step 3: Resolve Dependency Gates

For each feature showing `status: "waiting"` and `step_in_progress: false`:

Evaluate every entry in its `gate_conditions` map:
- `"done"` → the referenced feature must carry `status: "done"`
- `"plan_or_later"` → the referenced feature's status must be one of:
  `["plan", "tests", "implement", "validate", "review", "done"]`

When ALL of a feature's gate conditions hold: set `status: "research"` (its pipeline's first step).

Write the tracker once gates have been resolved.

---

## Step 4: Identify Steps to Spawn

Gather every feature where:
- `status` is one of `["research", "plan", "tests", "implement", "validate", "review"]`
- `step_in_progress: false`

These are the **ready steps**: each gets one background agent.

---

## Step 5: Spawn a Fresh Agent for Each Ready Step

Handle each ready step with the following sequence IN ORDER (never jump ahead):

**a) Compute the output path** where this step's artifact will land:

```
Base dir: thoughts/shared/coding/<slug>/<feature-id>/
Research:  <base>/research/<today-YYYY-MM-DD>-<feature-id>-research.md
Plan:      <base>/plan/<today-YYYY-MM-DD>-<feature-id>-plan.md
Tests:     (no single output path, tests are written into the codebase per the plan doc)
Implement: (no output path, changes are in the codebase)
Validate:  <base>/plan/<today-YYYY-MM-DD>-<feature-id>-validation.md
Review:    <base>/review/<today-YYYY-MM-DD>-<feature-id>-review.md
```

For `global-research`:
```
Research:  thoughts/shared/coding/<slug>/global-research/research/<today>-global-research.md
```

**b) Build the agent prompt** from the matching step-type template (Section 6).

**c) Mark the step as in-progress** in the tracker BEFORE the spawn happens:
```json
{
  "step_in_progress": true,
  "step_started_at": "<current ISO timestamp>"
}
```

Write the tracker.

**d) Spawn the agent:**
```
Agent(
  description="<step-type> agent for <feature-id>",
  prompt=<built prompt from step b>,
  run_in_background=true
)
```

Launch every ready step as a background agent. The conductor never waits on any of them.

---

## Step 6: Step Agent Prompt Templates

Resolve every `<placeholder>` at spawn time. Each template is the step agent's entire
instruction set, the agent arrives with zero prior context, so nothing outside the prompt
can be assumed.

---

### RESEARCH PROMPT TEMPLATE

```
You are the RESEARCH agent in an autonomous feature pipeline. You start with zero context.
Do not read any files except those listed below.

FEATURE: <feature-id>, <feature title>
SPEC SLUG: <slug>
TRACKER: thoughts/shared/coding/<slug>/pipeline-status.json
PM SPEC: <spec_path>
OUTPUT DOC PATH: <pre-computed research output path>

STEP 1, READ YOUR INPUTS:
Read these files in order:
  a. CLAUDE.md, note all hard rules (security, scalability, code quality)
  b. <spec_path>, locate the section covering "<feature-id>". Pull out:
     - "Research topics for /research_codebase" (the questions you must answer)
     - "What changes, explicit scope" (the code you will be hunting for)
     - "Acceptance criteria"

STEP 2, READ THE RESEARCH METHODOLOGY:
Read ~/.claude/commands/research_codebase.md, it explains HOW codebase research is done
here (what to examine, how findings are organized, which format to produce).
Apply its methodology, but write your output to the OUTPUT DOC PATH above (not the
default location the skill would normally choose).

STEP 3, INVESTIGATE THE CODEBASE:
Work through every research question in the pm-spec's "Research topics" list.
Be concrete: cite exact line numbers, type signatures, and file paths.
Record any CLAUDE.md constraints bearing on the feature's scope.

STEP 4, WRITE THE RESEARCH DOC:
Write your findings to: <pre-computed research output path>
Create parent directories if they're missing.

STEP 5, UPDATE THE TRACKER:
Read thoughts/shared/coding/<slug>/pipeline-status.json.
Change ONLY these fields for <feature-id>:
  features.<feature-id>.docs.research = "<research doc path>"
  features.<feature-id>.status = "<next step OR 'done' if this feature's pipeline is ['research']>"
  features.<feature-id>.step_in_progress = false
  features.<feature-id>.step_started_at = null

For global-research: next status = "done". For all other features: next status = "plan".

If anything fails: set status = "failed", error = "<brief description>", step_in_progress = false.

Do NOT spawn any other agents. Exit when done.
```

---

### PLAN PROMPT TEMPLATE

```
You are the PLAN agent in an autonomous feature pipeline. You start with zero context.

FEATURE: <feature-id>, <feature title>
SPEC SLUG: <slug>
TRACKER: thoughts/shared/coding/<slug>/pipeline-status.json
PM SPEC: <spec_path>
RESEARCH DOC: <docs.research from tracker>
OUTPUT PLAN PATH: <pre-computed plan output path>

STEP 1, READ YOUR INPUTS:
Read these files in order:
  a. CLAUDE.md
  b. <spec_path>, the feature brief for <feature-id>
  c. <docs.research>, the research findings

STEP 2, READ THE PLAN METHODOLOGY:
Read ~/.claude/commands/create_plan.md for the plan format and process.
Apply its methodology exactly, but write the plan to OUTPUT PLAN PATH (not the skill's default).

STEP 3, WRITE THE PLAN:
Produce the plan document at <pre-computed plan output path>.

STEP 4, PLAN REVIEW:
Spawn a reviewer Agent (NOT a fork) with this prompt:
  "You are a senior engineer reviewing an implementation plan. Read the plan at <plan path>.
   Check: (1) Are all CLAUDE.md hard rules respected? (2) Is every scope item from the
   pm-spec covered? (3) Are behavioral contracts testable? (4) Is the phase order safe?
   Append '## Plan Review' to the plan file with: CRITICAL/WARNING findings + final verdict
   APPROVED or REVISE with numbered list."
Wait for the reviewer to complete.

STEP 5, HANDLE REVIEW:
If APPROVED: proceed to Step 6.
If REVISE: apply every CRITICAL fix to the plan file, then re-spawn the reviewer once more.
  If still REVISE: update tracker with status="failed",
    error="Plan reviewer REVISE after 2 attempts", step_in_progress=false. Exit.

STEP 6, UPDATE TRACKER:
  features.<feature-id>.docs.plan = "<plan path>"
  features.<feature-id>.status = "tests"
  features.<feature-id>.step_in_progress = false
  features.<feature-id>.step_started_at = null

Do NOT spawn any other agents. Exit when done.
```

---

### TESTS PROMPT TEMPLATE

```
You are the TESTS agent in an autonomous feature pipeline. You start with zero context.

FEATURE: <feature-id>, <feature title>
SPEC SLUG: <slug>
TRACKER: thoughts/shared/coding/<slug>/pipeline-status.json
PLAN DOC: <docs.plan from tracker>

STEP 1, READ YOUR INPUTS:
Read CLAUDE.md and <docs.plan>.

STEP 2, READ THE TESTS METHODOLOGY:
Read ~/.claude/commands/test_implementation.md and apply it to this feature.

STEP 3, WRITE RED-PHASE TESTS:
Author failing tests covering every behavioral contract in the plan. Each test must run,
and must fail for the right reason (missing feature, not a syntax or import problem).

STEP 4, TESTS REVIEW:
Spawn a reviewer Agent (NOT a fork):
  "Review the red-phase tests for <feature-id>. Plan: <plan path>.
   Check: (1) Does every behavioral contract have at least one test? (2) Do tests fail for
   the right reason (feature not implemented, not syntax/import error)? (3) Are edge cases
   covered? Append '## Test Review' to the plan file with findings + APPROVED or NEEDS_MORE."
Wait for reviewer.

STEP 5, HANDLE REVIEW:
If APPROVED: proceed to Step 6.
If NEEDS_MORE: write the missing tests, re-run the reviewer once. If still NEEDS_MORE:
  status="failed", error="Test reviewer NEEDS_MORE after 2 attempts". Exit.

STEP 6, UPDATE TRACKER:
  features.<feature-id>.status = "implement"
  features.<feature-id>.step_in_progress = false
  features.<feature-id>.step_started_at = null

Do NOT spawn any other agents. Exit when done.
```

---

### IMPLEMENT PROMPT TEMPLATE

```
You are the IMPLEMENT agent in an autonomous feature pipeline. You start with zero context.

FEATURE: <feature-id>, <feature title>
SPEC SLUG: <slug>
TRACKER: thoughts/shared/coding/<slug>/pipeline-status.json
PLAN DOC: <docs.plan from tracker>

STEP 1, READ YOUR INPUTS:
Read CLAUDE.md and <docs.plan> in full.

STEP 2, READ THE IMPLEMENT METHODOLOGY:
Read ~/.claude/commands/implement_plan.md and apply it.

STEP 3, IMPLEMENT ALL PHASES:
Take the plan's phases in order. For each one:
  a. Implement the phase changes.
  b. Spawn a phase reviewer Agent (NOT a fork):
     "Review Phase <N> of <feature-id> implementation. Plan: <plan path>.
      Check: (1) All scope items for this phase are implemented per the plan. (2) No
      CLAUDE.md rules violated. (3) Tests from the red phase now pass for this phase.
      Append '## Phase <N> Review' to the plan. Verdict: APPROVED or REQUEST_CHANGES."
     Wait for reviewer.
  c. If APPROVED: move on to the next phase.
  d. If REQUEST_CHANGES: fix the issues, re-run the phase reviewer once.
     If still REQUEST_CHANGES:
       status="failed", error="Phase <N> reviewer REQUEST_CHANGES after 2 attempts". Exit.

Do NOT skip phases. Do NOT advance until each phase is APPROVED.

STEP 4, UPDATE TRACKER (only after all phases APPROVED):
  features.<feature-id>.status = "validate"
  features.<feature-id>.step_in_progress = false
  features.<feature-id>.step_started_at = null

Do NOT spawn any other agents. Exit when done.
```

---

### VALIDATE PROMPT TEMPLATE

```
You are the VALIDATE agent in an autonomous feature pipeline. You start with zero context.

FEATURE: <feature-id>, <feature title>
SPEC SLUG: <slug>
TRACKER: thoughts/shared/coding/<slug>/pipeline-status.json
PLAN DOC: <docs.plan from tracker>
OUTPUT VALIDATION PATH: <pre-computed validation output path>

STEP 1, READ YOUR INPUTS:
Read CLAUDE.md and <docs.plan>.

STEP 2, READ THE VALIDATE METHODOLOGY:
Read ~/.claude/commands/validate_plan.md and apply it.

STEP 3, RUN VALIDATION:
Run the full validation suite (type check, lint, tests) exactly as validate_plan.md directs.
Write the results to <pre-computed validation output path>.

STEP 4, HANDLE RESULTS:
If PASS: proceed to Step 5.
If PARTIAL or FAIL: fix the failing items, re-run once. If still failing:
  status="failed", error="Validation FAIL after 1 fix attempt: <summary>". Exit.

STEP 5, UPDATE TRACKER:
  features.<feature-id>.docs.validation = "<validation doc path>"
  features.<feature-id>.status = "review"
  features.<feature-id>.step_in_progress = false
  features.<feature-id>.step_started_at = null

Do NOT spawn any other agents. Exit when done.
```

---

### REVIEW PROMPT TEMPLATE

```
You are the REVIEW agent in an autonomous feature pipeline. You start with zero context.

FEATURE: <feature-id>, <feature title>
SPEC SLUG: <slug>
TRACKER: thoughts/shared/coding/<slug>/pipeline-status.json
PLAN DOC: <docs.plan from tracker>
OUTPUT REVIEW PATH: <pre-computed review output path>

STEP 1, READ YOUR INPUTS:
Read CLAUDE.md and <docs.plan>.

STEP 2, READ THE REVIEW METHODOLOGY:
Read ~/.claude/commands/review_implementation.md and apply it.
Write your review doc to <pre-computed review output path>.

STEP 3, SPAWN THE CODE REVIEWER:
Spawn a reviewer Agent (NOT a fork) to review the implementation independently:
  "You are a senior engineer doing a final code review for <feature-id>.
   Read: CLAUDE.md, <plan path>, and the git diff for this feature's changes.
   Check: (1) All acceptance criteria met. (2) No CLAUDE.md violations. (3) No security
   issues. (4) No N+1 queries or scalability regressions. (5) Configuration lives in the
   project's designated config module, not scattered inline.
   Append '## Code Review' to <review path>. Verdict: APPROVED or CHANGES_REQUESTED with list."
Wait for reviewer.

STEP 4, HANDLE REVIEW:
If APPROVED: proceed to Step 5.
If CHANGES_REQUESTED: fix every CRITICAL item, re-run the reviewer once.
  If still CHANGES_REQUESTED:
    status="failed", error="Code review CHANGES_REQUESTED after 1 fix: <summary>". Exit.

STEP 5, CREATE THE PR:
Open a GitHub PR:
  - Title: "<feature-id>: <feature title>"
  - Branch: feature/<slug>-<feature-id-lowercase>
  - Body: summary of changes, link to plan doc, test plan

STEP 6, UPDATE TRACKER:
  features.<feature-id>.docs.review = "<review path>"
  features.<feature-id>.pr_url = "<PR URL>"
  features.<feature-id>.status = "done"
  features.<feature-id>.step_in_progress = false
  features.<feature-id>.step_started_at = null

Do NOT spawn any other agents. Exit when done.
```

---

## Step 7: Check for Completion

With every ready-step agent spawned, read the tracker once more (carrying the Step 5c updates).

Tally features by terminal status:
- `done` count
- `failed` count
- in-flight count (everything else non-terminal)

**When ALL features read `done` or `failed`:**
1. Cancel the CronJob: `CronDelete(id=<cron_job_id from tracker>)`
2. Report final status:
   ```
   Pipeline complete for <slug>.

   Done (<count>): F1, F2, F4, ...
   Failed (<count>): F3 (plan reviewer), ...

   For failed features: fix the error, set status back to the failed step, step_in_progress=false.
   Then run /spec-conductor <slug> to resume.
   ```
3. Exit (no reschedule).

**When anything is still in-flight:** nothing more to do, the durable CronJob fires again
within 2 minutes on its own. Exit.

---

## Step 8: Write Final Tracker State

Just before exiting, make sure `pipeline-status.json` matches reality exactly, with
`last_conductor_run` set to the current ISO timestamp.
