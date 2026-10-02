# PM Spec

Distill a finalized features doc into an engineering-ready PM spec for: $ARGUMENTS

## Instructions

You are acting as a principal product manager turning a finalized features document into a clean handoff spec for engineering. Given the path to a `features-final.md` file, produce a structured PM spec complete enough that a senior engineer never needs to open a single earlier PM doc. Work through these steps exactly.

### 1. Load Context

- Read `CLAUDE.md` in full and pull out every rule that constrains implementation (security, scalability, code quality, hard rules).
- Read the `features-final.md` at the path in `$ARGUMENTS`.
- Derive the spec directory root (drop `/features/features-final.md` from the path) and read:
  - `<spec-root>/requirements/requirements.md`: every requirement plus its acceptance criteria
  - `<spec-root>/pm-brief.md`: the one-page context (persona, problem, goal, metric)
- Look in `thoughts/shared/research/` for prior codebase research overlapping this feature area, where it exists, note it so the next step extends it instead of redoing it.

### 2. Synthesize the PM Spec

Write a PM spec using the structure below. Its audience is the engineer who will run `/research_codebase` next, so every section must tell them what to do, not merely describe things. Discard all PM process residue (pre-flight audit analysis, design alternatives considered, reviewer checklists). What survives is decisions and their consequences, nothing else.

```markdown
# PM Spec: <Feature Set Title>
**Date:** YYYY-MM-DD
**Spec slug:** <slug, same as the spec root directory name>
**Source:** <path to features-final.md>
**Branch target:** feature/<slug>

## Executive Brief
<3–4 sentences covering: the persona, the primary goal, the urgency of shipping now, and the
single most important gap this closes. Written as one paragraph, no bullets, so an engineer
with zero context can read it cold and grasp why the work matters.>

## CLAUDE.md Constraints Relevant to This Spec
<Numbered list of CLAUDE.md constraints that bound implementation decisions in this spec.
Restrict it to rules with direct relevance, leave out generic ones unless some specific
feature violates or depends on them.>
1. <Constraint>: <How it applies to this spec specifically>
2. ...

## Feature Implementation Order
| Order | Feature | Phase | MoSCoW | Satisfies | Depends On |
|-------|---------|-------|---------|-----------|------------|
| 1 | <Feature title> | 1 | Must | R-XX | None |
| 2 | <Feature title> | 1 | Must | R-XX | Feature N |
| ... | | | | | |
| Phase 2+ | <Feature title> | 2 | Should | R-XX | Feature N |

**Build sequencing rationale:** <1–3 sentences justifying this order, naming the concrete
blocking dependencies. Point out the quickest independent win ("can start immediately") and
the riskiest / most-entangled item ("start last").>

## Per-Feature Engineering Briefs

---

### Feature <N>: <Title>
**Requirements:** <R-XX, R-XX, NF-XX>
**Phase / MoSCoW:** Phase <N> / <Must | Should | Could>
**Build order:** <first | in parallel with Feature N | after Feature N is merged>

**What changes, explicit scope:**
- `<file-path>`: <the exact change, additions, removals, modifications, stated concretely
  enough that a Grep search would land on the right symbol.>
- `<file-path>`: ...

**What does NOT change, explicit out-of-scope:**
- <Item>: <reason, this is what keeps engineering from drifting past the boundary>
- ...

**Key decisions consolidated from PM + Design:**
- <Decision 1>: <the decision, plus the constraint or rationale that makes it non-obvious.
  Belongs here only if an engineer reading the code cold would otherwise be surprised.>
- <Decision 2>: ...

**Open design questions resolved:**
<For each open question in the features-final Designer Pass for this feature, record its
resolution. Where a question can't be answered without reading code, promote it to the
"Research topics" list below.>
- Q: <question from features-final> → A: <resolution OR "see Research topic N below">

**Acceptance criteria (verbatim from requirements.md):**
- [ ] <AC-1>
- [ ] <AC-2>

**Dependencies:** <None | Feature N must be merged first | Feature N's design must be decided (not shipped)>

**Research topics for /research_codebase:**
<Concrete questions the codebase research has to answer before implementation can start.
Each one must be pointed enough to drive a Glob/Grep search. Never "look at this file", 
instead "confirm whether X component uses config-driven tokens or hardcoded values for Y
property.">
- RQ-<N>: <Specific researchable question>
- RQ-<N>: ...

---

(repeat per feature)

---

## Global Research Topics

<Questions cutting across multiple features that must be settled before any implementation
starts. These belong in the first /research_codebase call, not in a feature-specific one.>

- GRQ-1: <question>
- GRQ-2: ...

## Requirements Coverage Check

| Req ID | Statement (short) | Feature | Phase | Covered? |
|--------|-------------------|---------|-------|----------|
| R-01 | <brief label> | Feature N | 1 | Yes |
| R-02 | ... | ... | ... | ... |
| NF-07 | ... | ... | ... | ... |

## Next Steps

```
Research complete for <phase>. See: <path to this pm-spec file>

Next step, copy and run:
/research_codebase <first feature to implement>, see <pm-spec-path>
```

The research topic is <feature title of the first build-order item>. Where several features
are independent and could begin in parallel, list a separate `/research_codebase` call for
each in the next steps.
```

### 3. Self-Consistency Check

Before the file gets written, confirm:

- Every requirement ID from `requirements.md` shows up in the Requirements Coverage Check table.
- Every open design question from `features-final.md` is either answered under "Open design questions resolved" or promoted to a Research Topic.
- The Feature table's build order agrees with the stated dependencies (nothing depends on a feature that comes after it).
- No scope item collides with a CLAUDE.md hard rule, and if one does, make the conflict loudly visible in the brief.

### 4. Save PM Spec

Work out the spec root directory from the `features-final.md` path (two levels above `/features/features-final.md`).

Write to: `<spec-root>/pm-spec/pm-spec.md`

If the `pm-spec/` subdirectory is missing, just Write directly, the directory comes into existence automatically.

### 5. PM Expert Review

Once saved, spawn a **separate reviewer Agent** with this prompt:

> You are a principal PM and engineering manager reviewing a PM spec ahead of engineering handoff. Read the PM spec at `<pm-spec-path>` together with its sources: the `features-final.md` at `<features-final-path>` and `requirements/requirements.md` under the same spec root. Read `CLAUDE.md` as well.
>
> You are looking for anything that would steer the engineer wrong, or leave them unable to formulate a research query.
>
> 1. **Coverage**: Match every requirement ID in requirements.md against the Requirements Coverage Check table. A requirement with no covering feature is a CRITICAL gap.
> 2. **Scope precision**: Per feature, could an engineer turn each scope item into a concrete Grep search? "changes `DEFAULT_PAGE_SIZE` in `pagination.ts`" qualifies; "update the pagination system" does not. Flag every scope item too fuzzy to search on.
> 3. **Decision completeness**: Per open design question in features-final, confirm it is either resolved in the PM spec or carried as a research topic. Report any that vanished.
> 4. **Dependency coherence**: Inspect the Feature Implementation Order table. Where Feature B depends on Feature A, confirm A precedes B. Report any cycle or dependency that went unrecorded.
> 5. **CLAUDE.md compliance**: Does any scope item or decision run against a CLAUDE.md hard rule? (for instance rules like: parameterize all SQL, keep privileged keys out of client bundles, keep sensitive data server-side)
> 6. **Research topic actionability**: Per Research Topic (RQ-N, GRQ-N), would a `/research_codebase` call built on that question land on a specific file and function, or is it too broad? "Understand the data flow" is not actionable; "confirm whether `buildQuery` in `lib/db/query.ts` accepts a filter param" is.
>
> Append a `## PM Review` section to the pm-spec file containing:
> - Per-finding: CRITICAL / WARNING / SUGGESTION + where in the spec + the specific fix
> - Final verdict: **APPROVED** or **REVISE** (with a numbered list of required changes)
>
> Do NOT flag: writing style, section ordering, number of features, document length.

### 6. Address Review Findings

On a **REVISE** verdict:
- Edit the PM spec to resolve every CRITICAL and WARNING finding.
- Spawn the reviewer again with the identical prompt against the updated file.
- Keep going until the verdict is **APPROVED**.

### 7. Output Next-Step Prompt

Once the review lands on **APPROVED**, print for the user:

```
PM spec approved and saved. See: <pm-spec-path>

Open design questions resolved: <count>
Requirements covered: <N>/<total> (all Must + Should)
Features: <count Phase 1 Must> Phase 1 Must, <count Phase 2> Phase 2 Should/Could

Next step, copy and run:
/research_codebase <first feature>, see <pm-spec-path>
```

Where several features are independent and researchable in parallel, list each one separately.

### 8. Context Check

Should context usage exceed 60%, suggest the user run `/compact` before moving on to `/research_codebase`.
