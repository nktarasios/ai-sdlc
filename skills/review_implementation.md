# Review Implementation

Full final code review, the gate before a PR, for: $ARGUMENTS

## Instructions

You are a **senior engineer conducting the last code review** before a PR gets opened. Nothing checks this work after you, so leave nothing unexamined. Work through these steps exactly:

### 1. Load Context

- Read `CLAUDE.md` in full, Critical Rules, architecture notes, and file dependencies above all.
- Read the plan doc named in the arguments.
- If the plan cites a research doc, read that too.
- If a validation report exists (the plan's slug with a `-validation` suffix), read it.
- If the plan doc carries a test summary section, go over it.

### 2. Review All Changes

Pull the complete diff with `git diff main...HEAD` and review EVERY file it touches.

Judge each file on:

1. **Architecture Fit**: Does the change belong where it landed? Does it match how the codebase already does things?
2. **Code Quality**: Readable, clean, maintainable? Free of dead code, commented-out blocks, and ticketless TODOs?
3. **Security**:
   - No SQL injection (parameterized queries only)
   - No XSS (proper escaping/sanitization)
   - No command injection
   - No secrets/credentials in code
   - No unsafe deserialization
4. **Performance**:
   - No N+1 queries
   - No unnecessary DB calls
   - Algorithms suited to the actual data size
   - No blocking operations in async paths
5. **Tests**: Enough coverage? Assertions that mean something?
6. **Documentation**: If the architecture moved, did CLAUDE.md move with it? Are the genuinely non-obvious spots commented?
7. **Verification**: Pick the 2-3 riskiest changes across the whole diff and actually run something, a test or command probing an edge case you distrust. A failure is a real bug: report it CRITICAL alongside the reproducing command. A pass still earns a note in the review recording what you exercised. This is the step that converts hunches into evidence.

### 2.5 Code Debt Assessment

Fill in this table in the review report, covering every changed file:

```markdown
## Code Debt Assessment
| Debt Type | Severity | Location | Description |
|-----------|----------|----------|-------------|
| Premature abstraction (< 2 callers) | High/Med/Low | file:line | |
| Gold-plating (code no test exercises) | High/Med/Low | file:line | |
| Dead code | High/Med/Low | file:line | |
| Duplicate logic | High/Med/Low | file:line | |
| SRP violation | High/Med/Low | file:line | |
| Unnecessary indirection | High/Med/Low | file:line | |
```

### 2.6 Security Assessment

Run a full OWASP pass over every changed line. Fill in this table in the review report:

```markdown
## Security Assessment
| Class | Status | Location | Notes |
|-------|--------|----------|-------|
| SQL injection (parameterized queries only) | PASS/FAIL/N/A | | |
| XSS (output escaping, no raw user data in HTML) | PASS/FAIL/N/A | | |
| Command injection (no shell=True with user input) | PASS/FAIL/N/A | | |
| Path traversal (no unsanitized paths from input) | PASS/FAIL/N/A | | |
| Secrets in code (no credentials/tokens hardcoded) | PASS/FAIL/N/A | | |
| Auth/authz (endpoints enforce what contracts specified) | PASS/FAIL/N/A | | |
| Input validation (only at system boundaries) | PASS/FAIL/N/A | | |
| Unsafe deserialization | PASS/FAIL/N/A | | |
| CSRF (no state-changing GETs) | PASS/FAIL/N/A | | |
| Race conditions (shared mutable state) | PASS/FAIL/N/A | | |
```

Every PASS needs a short note on what was checked and why the code is safe. A bare PASS with no reasoning is a rubber stamp, not a review.

### 2.7 Performance Assessment

Fill in this table in the review report:

```markdown
## Performance Assessment
| Issue | Severity | Location | Notes |
|-------|----------|----------|-------|
| N+1 queries | High/Med/Low | | |
| Missing indexes for new query patterns | High/Med/Low | | |
| Blocking I/O in async paths | High/Med/Low | | |
| Unnecessary repeated DB calls in a request | High/Med/Low | | |
| Algorithm inappropriate for data scale | High/Med/Low | | |
```

**Verdict rules:**
- Any **FAIL** in Security → CHANGES REQUESTED [CRITICAL]
- Any **High** severity debt or performance → CHANGES REQUESTED [CRITICAL]
- **Med** severity → CHANGES REQUESTED [WARNING]
- **Low** severity → SUGGESTION

### 3. Project Critical Rules Check

List out every hard rule your project's `CLAUDE.md` declares (architecture invariants, data-handling constraints, framework-specific requirements, known platform limits) and check each one against the diff explicitly:

- [ ] <Rule 1 from CLAUDE.md, verified how>
- [ ] <Rule 2 from CLAUDE.md, verified how>
- [ ] ...one checkbox per hard rule, no rule skipped

A rule the diff doesn't touch still gets a line, an explicit N/A with a one-sentence reason. Silence is not an answer.

### 4. Generate Review

Produce a structured review:

```markdown
# Code Review: <Plan Title>
**Date:** YYYY-MM-DD
**Reviewer:** Claude Code (automated)
**Branch:** <branch name>
**Commits:** <N commits>
**Files Changed:** <N files>

## Summary
<2-3 sentences: what got built, and the overall judgment>

## File-by-File Review

### `path/to/file.py`
- **Purpose of changes:** <what changed and why>
- **Issues:**
  - [CRITICAL] <must fix before merge> (line N)
  - [WARNING] <should fix> (line N)
  - [SUGGESTION] <nice to have> (line N)
- **Verdict:** APPROVE / CHANGES REQUESTED

### `path/to/other.ts`
...repeat for each changed file...

## Project Critical Rules
| Rule | Status | Notes |
|------|--------|-------|
| <CLAUDE.md rule 1> | PASS/FAIL/N/A | <details> |
| <CLAUDE.md rule 2> | PASS/FAIL/N/A | <details> |
| ... | ... | ... |

## Test Assessment
- Backend tests: <assessment>
- Frontend tests: <assessment>
- Coverage gaps: <any gaps>

## Overall Verdict: APPROVED / CHANGES REQUESTED

### If CHANGES REQUESTED:
**Critical items (must fix):**
1. <item>

**Warnings (should fix):**
1. <item>

**Suggestions (optional):**
1. <item>
```

### 5. Save Review

Write to: `thoughts/shared/reviews/YYYY-MM-DD-<slug>-review.md`

### 6. Output Result

If **APPROVED**:
```
Code review APPROVED. See: thoughts/shared/reviews/YYYY-MM-DD-<slug>-review.md

Ready to create PR. Copy and run:

gh pr create --title "<suggested title>" --body "$(cat thoughts/shared/reviews/YYYY-MM-DD-<slug>-review.md)"
```

If **CHANGES REQUESTED**:
```
Code review: CHANGES REQUESTED. See: thoughts/shared/reviews/YYYY-MM-DD-<slug>-review.md

Items to fix:
1. [CRITICAL] <item>
2. [WARNING] <item>

After fixing, re-run:
/review_implementation <plan-path>

Or if changes require re-implementing:
/implement_plan <plan-path> (to address review items)
```
