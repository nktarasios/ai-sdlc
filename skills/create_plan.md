# Create Plan

Produce a phased implementation plan for: $ARGUMENTS

## Instructions

You are acting as a senior software architect drafting an implementation plan. Work through these steps exactly:

### 1. Load Context

- Read `CLAUDE.md` end to end, giving particular weight to Critical Rules, Database Architecture, and File Dependencies.
- When the arguments point at a research doc (e.g., `see thoughts/shared/research/...`), read that doc in full.
- Scan `thoughts/shared/plans/` for existing plans in the same territory.

### 2. Draft the Plan

Build a phased plan in this shape:

```markdown
# Plan: <Feature Title>
**Date:** YYYY-MM-DD
**Research:** <path to research doc, if any>
**Branch:** feature/<slug>

## Goal
<2-3 sentences: what is being built, and the reason it matters>

## Out of Scope
- <Things this plan deliberately will NOT touch>

## Risks
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| ... | ... | ... | ... |

## Phase 1: <Phase Title>
**Status:** NOT STARTED
**Files:**
- `path/to/file.py`: <what changes>
- `path/to/other.py`: <what changes>

**Changes:**
1. <A change described with enough detail to act on>
2. <Another concrete change>

**Behavioral Contracts:**
Specifications exact enough that failing tests can be written from them alone, with no implementation code to consult.
- `function_name(param: Type, other: Type) -> ReturnType`
  - Returns X when given valid inputs Y
  - Raises/returns error Z when param is None or invalid
  - Edge case: when list is empty, returns []
  - Edge case: when value equals boundary N, behavior is X
- `POST /api/endpoint`
  - Request: `{"field": string (required), "count": int (optional, default 0)}`
  - Response 200: `{"id": int, "status": "created", "field": string}`
  - Response 422: when `field` is missing or empty
  - Response 409: when resource with same key already exists
  - Response 404: when parent resource does not exist
- DB invariant: after this phase, table X always has Y relationship to table Z

Security Contracts:
- Which endpoints/functions demand authentication, and what an unauthorized caller gets back (401 vs 403)
- Which fields get validated at input boundaries, what gets rejected, and with which error
- Which fields get sanitized (SQL params, HTML output, shell args)
- No credentials, tokens, or secrets stored in code or plaintext

**Testing:**
- <Test files to create or update>
- <Scenarios needed on top of what the Behavioral Contracts already imply>

**Verification:**
```bash
cd backend && ruff check .
cd backend && python -m pytest tests/ -v
cd frontend && npx tsc --noEmit
cd frontend && npm run lint
cd frontend && npm run test
```

## Phase 2: <Phase Title>
...repeat structure...

## Verification Checklist
- [ ] All phases completed
- [ ] ruff check clean
- [ ] pytest passing
- [ ] tsc --noEmit clean
- [ ] npm run lint clean
- [ ] npm run test passing
- [ ] next build succeeds
- [ ] Manual testing done
- [ ] CLAUDE.md updated (if needed)
```

**Plan requirements:**
- A phase must stand on its own for verification purposes, never bundle unrelated changes into one.
- A **Behavioral Contracts** subsection is mandatory in every phase. The bar for a contract: a developer holding only the contract text can write a failing test from it. "Validates input" fails that bar, the contract has to name which inputs are rejected and exactly what error comes back.
- A **Testing** subsection is mandatory in every phase, naming the test files plus any scenarios beyond the contracts.
- File paths must be real and exact, confirm each with Glob.
- Verification criteria must mirror CI: ruff, pytest, tsc, lint, vitest, next build.
- Order phases so nothing early leans on anything later.
- Size each phase to fit a single working session (roughly 30 minutes of work).
- **Minimize scope**: include only what the goal strictly requires. Never design for imagined future needs, three lines of plain code beat an abstraction built too early.

### 3. Iterate with User

Show the draft and invite feedback, iterating through at most 5 rounds:
- Ask directly: "Is the phase breakdown right? Should anything be added, dropped, or resequenced? Are the behavioral contracts tight enough to test against?"
- Fold in the feedback.
- If a requested change collides with a CLAUDE.md critical rule, say so.

### 4. Save the Plan

Derive a filename slug from the feature (lowercase, hyphens, nothing special).
Write to: `thoughts/shared/plans/YYYY-MM-DD-<slug>.md` (today's date).

### 5. Plan Review

Once saved, spawn a **separate reviewer Agent** with this prompt:

> You are a senior architect vetting an implementation plan. Open `thoughts/shared/plans/YYYY-MM-DD-<slug>.md` together with the project's `CLAUDE.md`. Your task is to surface problems while zero code exists.
>
> 1. **Contract Precision**: Take each behavioral contract and attempt to invent TWO implementations that both honor the contract's wording yet behave differently. Succeeding means the contract is ambiguous, flag it and show both readings.
> 2. **Edge Cases**: For each contract, which inputs or states does it stay silent on? Null, empty, zero, negative, duplicate, concurrent, max-length? Silence on any of these is a gap.
> 3. **Safety**: Audit every default value. When the feature is broken or switched off, does the system fail soft or fall over? What unstated assumptions exist about API availability, data format, OS, file permissions, or Docker state?
> 4. **Security**: Wherever external input can reach a shell command, SQL query, file path, or HTML output, are the auth, validation, and sanitization contracts actually written down?
> 5. **Phase Dependencies**: Could phase N genuinely be built with nothing beyond phase N-1 done? Confirm listed file paths exist (Glob). Confirm functions the contracts mention exist in the codebase (Grep).
> 6. **Scope**: Is anything over-built? An abstraction serving one caller? A phase whose removal would leave the goal intact?
>
> Append a `## Plan Review` section with your findings (CRITICAL / WARNING / SUGGESTION, each tied to a specific location) and a verdict: **APPROVED** or **REVISE**.
>
> Do NOT flag: naming preferences, documentation style, formatting choices.

On a **REVISE** verdict, walk the user through the flagged issues, revise the plan together, and send it back through the reviewer.

### 6. Output Next-Step Prompt

Once the plan carries an approval:

```
Plan approved and saved. See: thoughts/shared/plans/YYYY-MM-DD-<slug>.md

Next step, copy and run:
/test_implementation thoughts/shared/plans/YYYY-MM-DD-<slug>.md
```
