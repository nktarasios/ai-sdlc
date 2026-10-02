# Autopilot

Drive the full skill pipeline end to end, with a fresh context for every step: $ARGUMENTS

## Instructions

You are starting the autopilot conductor, which executes the whole workflow pipeline (test -> implement all phases -> validate -> review) without further human input. Every step gets a brand-new Claude instance carrying zero prior context, and the work happens inside a git worktree for isolation.

### 0. Prerequisite: the conductor script (not bundled, you write it)

This skill drives `scripts/autopilot.py`, which is **deliberately not shipped with this repo**: it is ~100 lines of plain Python that shells out to `claude -p` once per step, and you should own and shape it for your project. Before first use, write a script that satisfies this contract:

- **Invocation:** `python scripts/autopilot.py <plan-path> [--from <step>] [--to <step>] [--branch <name>] [--dry-run]`
- **Steps:** `test`, `implement_phase_1..N` (one per plan phase), `validate`, `review`: each runs `claude -p "/<matching-skill> <plan-path>"` in a fresh process (fresh context) inside a dedicated git worktree
- **`--dry-run`:** print the resolved step list and exit without running anything
- **State:** append per-step status (started/passed/failed, timestamp) to `thoughts/shared/autopilot/<slug>.json` so a failed run can resume with `--from`
- **Exit behavior:** commit after each passing step; on failure, print the exact resume command and exit non-zero; after the final step passes, merge the worktree branch back

If the script is missing, stop here and tell the user to create it first.

### 1. Parse Arguments

Expect a path to a plan document, optionally followed by flags:
- `--from <step>`: Begin at a chosen step (test, implement_phase_N, validate, review)
- `--to <step>`: Halt once a chosen step finishes
- `--branch <name>`: Branch name (default: autopilot/<slug>)
- `--dry-run`: List the steps without running anything

### 2. Validate

Confirm all of the following before anything launches:
1. A plan file actually exists at the given path
2. The plan carries a completed review (its Plan Review section contains APPROVED or PASS)
3. Absent a `--from` flag, the working tree is clean (`git status --porcelain` shows nothing)

Should any check fail, explain what needs fixing and stop, do not launch.

### 3. Dry Run First

Every launch is preceded by a `--dry-run` pass so the user sees exactly which steps will run:

```bash
python scripts/autopilot.py <plan-path> [flags] --dry-run
```

Present that output and get the user's go-ahead before continuing.

### 4. Launch

Start the conductor script. Expect a long wait, 5-30 minutes depending on how many phases the plan has:

```bash
python scripts/autopilot.py <plan-path> [flags]
```

What the conductor does:
- Sets up an isolated git worktree
- Gives each step its own fresh Claude context
- Commits whenever a step succeeds
- Streams its output live
- Merges back automatically when everything passes
- On failure, prints the command that resumes the run

### 5. Report Results

Once the conductor exits, read the status file at `thoughts/shared/autopilot/<slug>-status.json` and relay:
- The overall status (merged, failed, merge_failed)
- Which steps got through
- Total elapsed time
- On failure: the resume command
- On merge: confirmation that the branch was merged and cleaned up
