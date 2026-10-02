# Research Codebase

Thorough codebase investigation of: $ARGUMENTS

## Instructions

You are acting as a senior engineer doing deep research into a codebase. Work through these steps exactly:

### 1. Load Context

- Read `CLAUDE.md` in full, it defines the project's architecture, hard rules, and file dependencies.
- Look in `thoughts/shared/research/` for earlier research touching this topic; extend what exists rather than redoing it.
- If APIs, schemas, or components are in play, consult `README.md` for reference material.

### 2. Parallel Exploration

Launch **parallel Explore sub-agents** so the investigation proceeds on several fronts at once:

1. **File Discovery Agent**: Locate every file connected to the topic via Glob patterns and directory listings, and map how those files relate to one another.
2. **Code Analysis Agent**: Read the key sources the discovery pass surfaced. Work out the data flow, the function signatures, the class hierarchies, and the logic paths that matter.
3. **Test Coverage Agent**: Hunt down the tests that already cover this area. Establish what is exercised, what isn't, and which testing patterns the project follows.
4. **DB/API Surface Agent**: Enumerate the database tables, API endpoints, and data models in scope, and trace a request through its full lifecycle.

### 3. Synthesize Findings

Merge everything the sub-agents returned into one structured research document using these sections:

```markdown
# Research: <Topic>
**Date:** YYYY-MM-DD
**Scope:** <one-line description>

## Summary
<3-5 sentence executive summary>

## Files Involved
| File | Role | Lines of Interest |
|------|------|-------------------|
| ... | ... | ... |

## Architecture & Data Flow
<How the pieces talk to each other; the order of operations; where data changes shape>

## Current Behavior
<What the code actually does right now, pinned to specific functions and lines>

## Test Coverage
<Tests that exist, gaps found, patterns in use>

## DB/API Surface
<Tables, endpoints, models involved>

## Key Findings
<Numbered list of the discoveries that matter, edge cases and gotchas included>

## Open Questions
<What still needs clarifying or digging into>
```

### 4. Save Research

Derive a filename slug from the topic (lowercase, hyphens, nothing special).
Write the doc to: `thoughts/shared/research/YYYY-MM-DD-<slug>.md` (today's date).

### 5. Self-Review

Once the doc is saved, spawn a **separate reviewer Agent** with this prompt:

> You are a senior engineer auditing a codebase research document. Open `thoughts/shared/research/YYYY-MM-DD-<slug>.md` together with the project's `CLAUDE.md`. You are hunting for gaps that would stall an implementation.
>
> 1. **Accuracy**: Take 3-5 concrete claims and check them against the real source files, line numbers, function signatures, described behavior. Where a claim is wrong, supply the correction along with the file and line.
> 2. **Completeness**: Run your own Grep and Glob searches for files, functions, or patterns in this topic area that the doc never mentions. Report anything that belonged in it.
> 3. **Implicit Assumptions**: What is the doc quietly assuming that might not hold? (API format, OS behavior, library versions, data constraints, environment variables, Docker state)
> 4. **Actionability**: Imagine building a feature in this area armed with nothing but this doc and CLAUDE.md. Note every point where you'd be stuck, each one is a gap that must be closed.
> 5. **Open Questions**: Are the listed questions the right ones, or did the research dodge harder questions it should have asked?
>
> Append a `## Review` section to the research doc containing your findings plus a verdict: **PASS** or **NEEDS WORK** (with the specific items to fix).
>
> Do NOT flag: writing style, document formatting, or section ordering.

On a **NEEDS WORK** verdict, resolve every flagged item through further research, update the doc, and send it back through the reviewer.

### 6. Output Next-Step Prompt

Once the research clears review, print:

```
Research complete and reviewed. See: thoughts/shared/research/YYYY-MM-DD-<slug>.md

Next step, copy and run:
/create_plan <feature description>, see thoughts/shared/research/YYYY-MM-DD-<slug>.md
```

### 7. Context Check

Should context usage exceed 60%, suggest the user run `/compact` before moving on.
