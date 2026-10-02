# Retrospective

Study a finished feature and pull out the lessons worth keeping: $ARGUMENTS

## Instructions

Your task is to examine a completed feature implementation and distill from it whatever will make the next piece of work better. This fires automatically after every successful autopilot merge, and can equally be run by hand.

### 1. Load Artifacts

Starting from the plan path in the arguments, find and read:
- **Plan doc**: the plan itself (phases, contracts, review notes)
- **Autopilot log**: `thoughts/shared/autopilot/<slug>.log` (timing, failures, retries)
- **Autopilot status**: `thoughts/shared/autopilot/<slug>-status.json`
- **Review doc**: `thoughts/shared/reviews/YYYY-MM-DD-<slug>-review.md` (if exists)
- **Validation doc**: `thoughts/shared/plans/YYYY-MM-DD-<slug>-validation.md` (if exists)
- **Git log**: `git log --oneline` covering the autopilot commits

A missing artifact isn't fatal, record its absence and proceed with the rest.

### 2. Analyze

Pull out the following metrics and observations:

**Execution**:
- Wall-clock time end to end
- How long each step took (from log timestamps)
- Which steps landed on the first attempt versus needing fixes
- Any step that failed outright and had to be resumed

**Quality Gates**:
- What did the reviewers actually catch? Sort it: correctness bugs, security issues, edge cases, pattern violations, minimality concerns
- Did anything surface LATE (at review) that an EARLIER stage (planning or tests) should have surfaced?
- Did any reviewer finding turn out to be a false positive, flagged, but not actually a problem?

**Plan Quality**:
- Did any contract prove ambiguous enough to confuse the implementation?
- Did phase ordering hide a dependency problem?
- Did scope hold steady, or did it stretch or shrink mid-implementation?
- Were the named files the right ones, or did surprises turn up?

**Process**:
- What ran smoothly enough to deliberately repeat?
- Where was the friction, and what would remove it?

### 3. Write Retrospective

Write to: `thoughts/shared/retrospectives/YYYY-MM-DD-<slug>.md`

```markdown
# Retrospective: <Feature Title>
**Date:** YYYY-MM-DD
**Plan:** <plan-path>
**Total Time:** <elapsed>

## Metrics
| Step | Time | First Try? | Issues Found |
|------|------|------------|-------------|
| test | Xs | Yes/No | <count> |
| implement_phase_1 | Xs | Yes/No | <count> |
| ... | ... | ... | ... |

## What Went Well
- <thing that worked>

## What Should Improve
- <issue>: <what happened> -> <what should change>

## Late Catches
Issues surfaced at review/validation that stronger planning or tests would have surfaced sooner:
- <issue>: caught at <step>, should have been caught at <earlier-step> by <method>

## Lessons for Future Plans
- <actionable lesson>
```

### 4. Save Key Lessons to Memory

Where a lesson is weighty enough to shape future work, commit it to memory:
- Process improvements go in as feedback memories
- Architectural discoveries go in as project memories
- The bar for saving: genuinely non-obvious, and relevant beyond this one feature

### 5. Output

```
Retrospective complete. See: thoughts/shared/retrospectives/YYYY-MM-DD-<slug>.md
Lessons saved to memory: <count> (if any)
```
