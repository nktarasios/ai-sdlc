# Test Implementation

Turn the plan's contracts into tests ahead of any implementation (Red Phase): $ARGUMENTS

## Instructions

Your task is to author behavior-specifying tests directly from the plan's contracts, at a moment when **no implementation code exists yet**. This is TDD's Red Phase. Work through these steps exactly:

### 1. Load Context

- Read `CLAUDE.md` and the plan doc named in the arguments.
- Read the existing tests near this area to absorb the project's helpers, fixtures, and conventions.
- Read existing source files where needed for patterns, import paths, and DB schemas, correct test setup depends on them.
- Do NOT attempt to read implementation files that haven't been written, there is nothing there yet.

### 2. Map Contracts to Test Cases

Walk each phase's Behavioral Contracts and Security Contracts and enumerate the cases they demand:

- **Happy path**: every successful outcome the contracts name
- **Error paths**: every specified failure, matched to its exact status code or exception type
- **Boundary values**: empty, None/null, zero, max, single item
- **Security paths**: each security contract (rejecting unauthenticated requests, invalid input, injection strings)
- **DB invariants**: setup → action → assertions on resulting state

### 3. Write Tests

**Unit tests**: every function/method, across happy, error, edge, and boundary cases.

**Integration tests**: API endpoints (every specified status code plus response shape), DB round-trips.

**Security tests**: auth failures, missing required fields, injection payloads wherever a contract promises sanitization.

**Test quality rules:**
- Naming: `test_<function>_<scenario>_<expected_result>`
- Isolation: no test may depend on state another test left behind
- Mock the outside world (network, filesystem, external APIs); never mock the project's own modules
- Every test asserts something real, "it didn't crash" is not an assertion

**Test file locations:**
- Backend tests: `backend/tests/` using pytest (follow existing test patterns)
- Frontend tests: `frontend/tests/` using vitest (follow existing test patterns)

### 4. Verify Test Structure

Execute the whole suite:

```bash
cd backend && python -m pytest tests/ -v
cd frontend && npm run test
```

**How the new tests should behave right now:**
- **ImportError / ModuleNotFoundError**: fine; the module they target doesn't exist yet
- **AssertionError**: best case; the test is checking genuine behavior
- **Passing without implementation**: a red flag to fix; a test that passes against nothing is almost certainly asserting nothing

Tests that existed before must remain green.

### 5. Test Review

Spawn a **separate reviewer Agent** with this prompt:

> You are a senior QA engineer reviewing tests produced in a Red Phase TDD workflow. They were derived from plan contracts -- no implementation exists yet. Read the plan doc at `<plan-path>` (focus on the Behavioral Contracts and Security Contracts sections) plus every new test file. Your mission is to catch weak tests before they let bugs slip past.
>
> 1. **Contract Coverage**: Confirm every behavioral and security contract in the plan is exercised by at least one test. Report each uncovered contract, quoting its exact text.
> 2. **Test Strength**: Choose the 3 most intricate contracts. For each one, sketch a BROKEN implementation the current tests would still PASS. Managing to construct one proves the tests are too weak -- state which assertion or extra case would expose that broken implementation.
> 3. **Assertion Quality**: Verify each test asserts something substantive -- not merely "doesn't crash" or "returns something." A type-only check (isinstance) with no value check is weak. Happy-path-only coverage with no boundary values is incomplete.
> 4. **Security Tests**: Do the tests contain genuine attack inputs? (SQL injection strings, path traversal sequences, XSS payloads, oversized inputs) Asserting a bare 401 without throwing real attack payloads at the code is not a security test.
>
> Verdict: **APPROVE** or **NEEDS MORE TESTS** (naming the missing contracts and the weak tests, each paired with the broken implementation that would slip through).
>
> Do NOT flag: test naming conventions, import ordering, fixture organization, file structure.

On **NEEDS MORE TESTS**, author the additional tests and run the suite again.

### 6. Append Test Contracts to Plan Doc

Add this section to the end of the plan doc:

```markdown
## Test Contracts
**Date:** YYYY-MM-DD  **Tests Written:** N
| Phase | Contract | Test(s) | Expected Failure |
|-------|----------|---------|-----------------|
| 1 | `function_name(...)` | `test_function_name_happy_path` | AssertionError |
| 1 | `POST /api/endpoint` (422 on missing field) | `test_endpoint_missing_field_returns_422` | ImportError (pending) |
```

### 7. Output Next-Step Prompt

```
Red Phase complete, N tests written. See: <plan-path>

Next step, copy and run:
/implement_plan <plan-path>
```
