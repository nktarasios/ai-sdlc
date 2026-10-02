# Implement Plan

Carry out the next phase of the plan: $ARGUMENTS

## Instructions

You are acting as a senior engineer executing a plan one phase at a time. Work through these steps exactly:

### 1. Load Context

- Read `CLAUDE.md` in full, with particular attention to Critical Rules.
- Read the plan doc named in the arguments (e.g., `thoughts/shared/plans/YYYY-MM-DD-<slug>.md`).
- When the arguments name a phase (e.g., `phase 2`), work on that one. Otherwise pick the earliest phase still marked `NOT STARTED`.

### 2. Understand the Phase

- Before touching anything, read EVERY file the phase lists under **Files**.
- Know the current state of the code well enough that your edits are surgical rather than approximate.
- Check whether this phase leans on earlier ones, those must already read COMPLETED.
- Execute the pre-written tests for this phase and study the failures: they are your target list. Each failing test's name states a behavior you now owe.

### 3. Implement: Green then Refactor

**Green:**
- Write only as much code as it takes to make this phase's pre-written tests pass.
- The rule: no test asks for it, you don't write it. No gold-plating, no future-proofing, no abstractions serving a single caller.
- Honor every CLAUDE.md critical rule without exception, architecture invariants, data-handling rules, framework constraints all bind every phase. If following the plan would break one of them, stop and raise it rather than implementing.

**Refactor:**
- With every pre-written test green, sweep for duplication, weak names, and inline logic that has grown to ≥2 callers.
- Only pull logic out into a shared spot when the current code calls it from ≥2 places.
- After each refactor, run the tests again, green must survive every change.

### 4. Run Verification

Execute the complete verification suite (substitute your project's CI commands, the block below assumes a Python backend plus Node frontend):

```bash
cd backend && ruff check .
cd backend && python -m pytest tests/ -v
cd frontend && npx tsc --noEmit
cd frontend && npm run lint
cd frontend && npm run test
```

Resolve failures before moving on. A failure with no connection to your changes shouldn't block you, but confirm your changes truly didn't cause it, and write it down.

### 5. Phase Review

With implementation done and verification green, spawn a **separate reviewer Agent** with this prompt:

> You are a senior engineer reviewing the code changes for one implementation phase. The plan lives at `<plan-path>`; you are reviewing Phase N. Your mission is to intercept bugs before they ship.
>
> 1. **Green Check**: Run the project's test suite, everything must pass. Then inspect `git diff` for edits to test files: a modified test hints that the implementation fought the contracts and the tests were bent to fit. Any test change is CRITICAL.
> 2. **Correctness**: For the 2-3 riskiest changes in the diff:
>    - Identify the nastiest input this code could face and follow it through the code path.
>    - Consider a second call with identical arguments, what happens?
>    - Consider an upstream dependency handing back None, an empty string, or an exception, what happens?
>    - When you suspect a bug, write a quick test command and execute it. A reproduced bug is CRITICAL; a suspicion you couldn't reproduce is a WARNING.
> 3. **Minimality**: Does any code exceed what the tests demand? Any abstraction with a single caller? Any feature that no test touches?
> 4. **Pattern Fit**: Does the code sit naturally in the existing conventions (CLAUDE.md critical rules, naming patterns, file placement)?
>
> Output: findings labeled CRITICAL (must fix) / WARNING (should fix) / SUGGESTION (optional), each with file:line. Close with a verdict: **APPROVE** or **REQUEST CHANGES**.
>
> Do NOT flag: style preferences, documentation gaps, pre-existing code outside the diff.

On **REQUEST CHANGES**, resolve every item, re-run verification, and record the fixes in the plan doc.

### 6. Update Plan Status

Edit the plan doc to reflect this phase's completion:
- Flip `**Status:** NOT STARTED` to `**Status:** COMPLETED`
- Where the reviewer demanded fixes, record them: `**Reviewer fixes:** <brief description>`

### 7. Pause for Manual Testing

Say to the user:
> Phase N has been implemented and passed review. This is a good moment for any manual testing you want to do, say the word when you're ready to move on.

Do NOT roll straight into the next phase.

### 8. Output Next-Step Prompt

Look through the plan for phases still marked `NOT STARTED`.

If any remain:
```
Phase N complete. See updated plan: <plan-path>

When ready for the next phase, copy and run:
/implement_plan <plan-path> phase M
```

If every phase is done:
```
All phases complete. See updated plan: <plan-path>

Next step, copy and run:
/validate_plan <plan-path>
```

### 9. Context Check

Should context usage exceed 60%, suggest the user run `/compact` before starting the next phase.
