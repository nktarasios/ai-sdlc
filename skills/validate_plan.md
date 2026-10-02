# Validate Plan

Methodically confirm the plan was implemented as written: $ARGUMENTS

## Instructions

You are acting as a senior engineer running final validation over a finished implementation. Work through these steps exactly:

### 1. Load Context

- Read `CLAUDE.md` for the project's architecture and critical rules.
- Read the plan doc named in the arguments.
- Run `git status` to see where the working tree stands.
- Run `git log main...HEAD --oneline` for every commit on this branch.
- Run `git diff main...HEAD --stat` for a change summary.

### 2. Per-Phase Verification

Do this for EVERY phase in the plan:

1. **File Check**: Confirm each file the phase lists was genuinely created or modified, cross-reference against `git diff main...HEAD --name-only`.
2. **Change Check**: Read the actual diffs and check them against what the plan called for. Record any divergence.
3. **Test Check**: Confirm the tests promised in the phase's Testing subsection really got written.
4. **Status Check**: Confirm the plan doc marks the phase COMPLETED.
5. **Contract Check**: Match each Behavioral Contract in the phase against the plan doc's Test Contracts table, no contract may lack a test that currently passes.
6. **Security Contract Check**: Confirm a passing test backs every security contract.

### 3. Run Full Verification Suite

```bash
cd backend && ruff check .
cd backend && python -m pytest tests/ -v
cd frontend && npx tsc --noEmit
cd frontend && npm run lint
cd frontend && npm run test
cd frontend && npm run build
```

Note the outcome of every command.

### 4. Generate Validation Report

Produce a structured report:

```markdown
# Validation Report: <Plan Title>
**Date:** YYYY-MM-DD
**Plan:** <path to plan doc>
**Branch:** <current branch>
**Commits:** <number of commits since main>

## Per-Phase Results

| Phase | Status | Files Match | Changes Match | Tests Written | Notes |
|-------|--------|-------------|---------------|---------------|-------|
| 1: <title> | PASS/FAIL | Yes/No | Yes/No | Yes/No | <any deviations> |
| 2: <title> | PASS/FAIL | Yes/No | Yes/No | Yes/No | <any deviations> |
| ... | ... | ... | ... | ... | ... |

## Verification Suite Results

| Check | Result | Notes |
|-------|--------|-------|
| ruff check | PASS/FAIL | <details if fail> |
| pytest | PASS/FAIL (X passed, Y failed) | <details if fail> |
| tsc --noEmit | PASS/FAIL | <details if fail> |
| npm run lint | PASS/FAIL | <details if fail> |
| npm run test | PASS/FAIL (X passed, Y failed) | <details if fail> |
| next build | PASS/FAIL | <details if fail> |

## Deviations from Plan
<Every place where the built reality departs from the written plan>
- <deviation 1: what was planned vs what was done>

## Missing Items
<Everything the plan required that never got built>

## Contract Coverage
| Phase | Contract | Test | Status |
|-------|----------|------|--------|
| 1 | `function_name(...)` | `test_function_name_...` | PASS / FAIL / MISSING |
| 1 | Security: auth required | `test_endpoint_unauthenticated_returns_401` | PASS |

## Engineering Quality Summary
<A short narrative, not a checklist, call out any debt, security, or performance concerns
noticed along the way that /review_implementation should dig into properly.>

## Manual Testing Checklist
- [ ] <Scenario 1 to manually verify>
- [ ] <Scenario 2 to manually verify>
- [ ] <Scenario 3 to manually verify>

## Overall Result: PASS / PARTIAL / FAIL
```

### 5. Save Report

Write to: `thoughts/shared/plans/YYYY-MM-DD-<slug>-validation.md` (reusing the plan doc's slug).

### 6. Validation Review

Spawn a **separate reviewer Agent** with this prompt:

> You are a QA lead auditing a validation report. Read the validation report at `<report-path>` and the original plan at `<plan-path>`. Your task is to test whether the report's claims hold up.
>
> 1. **Contract Gaps**: Match every behavioral and security contract against the Test Contracts table, a contract with no PASSING test behind it is a gap. Execute `cd backend && python -m pytest tests/ -v` and compare the real results against what the report claims.
> 2. **Deviations**: Put the diff (`git diff main...HEAD`) next to the plan and read them together. Every mismatch between specification and implementation -- deliberate or accidental -- gets flagged with the plan section and the code location.
> 3. **Verification Claims**: Re-run the entire verification suite (ruff, pytest, tsc, lint) yourself. Any daylight between your results and the report's claims is CRITICAL.
> 4. **Manual Testing**: Could a person who has never seen this feature execute every checklist item as written? An item like "test the feature works" is useless, demand concrete steps.
>
> Append a `## Reviewer Assessment` section carrying your findings and a final verdict.
>
> Do NOT flag: report formatting, section ordering, writing style.

### 7. Output Results and Next-Step Prompt

Tell the user the overall result.

If PASS:
```
Validation PASSED. See report: <report-path>

Next step, copy and run:
/review_implementation <plan-path>
```

If PARTIAL or FAIL:
```
Validation <PARTIAL/FAIL>. See report: <report-path>

Issues to address:
- <issue 1>
- <issue 2>

After fixing, re-run:
/validate_plan <plan-path>
```
