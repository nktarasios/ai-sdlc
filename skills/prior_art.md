# Prior Art

Mine the project's history for knowledge that already exists before beginning new work: $ARGUMENTS

## Instructions

Your job is to sweep the project's accumulated artifacts, research docs, plans, reviews, retrospectives, for anything relevant to a new topic, so that research isn't repeated and hard-won lessons travel forward. Stay brief: what you produce is context for the next command, not a report in its own right.

### 1. Parse Topic

Pull the central topic out of the arguments, then expand it into related search terms. Consider:
- File and module names the topic touches
- Function names, class names, config fields
- Synonyms and neighboring concepts

### 2. Search

Grep across every knowledge artifact:

```
thoughts/shared/research/*.md
thoughts/shared/plans/*.md
thoughts/shared/reviews/*.md
thoughts/shared/retrospectives/*.md
```

Query for the topic terms plus associated file, function, and module names. Go broad rather than narrow, run 3-5 distinct keyword variations.

### 3. Read Selectively

From each file that matched, read ONLY:
- The `## Summary` section (fall back to the first 10 lines if there is no Summary heading)
- The `## Key Findings` section (when one exists)
- The individual paragraph surrounding each keyword hit

Never read whole documents. The full text remains on disk for anyone who needs a deep dive later.

### 4. Verify Currency

Whenever a key finding names a concrete file path, function, config field, or API endpoint:
- Glob to confirm the file is still where the finding says it is
- Grep to confirm the function or class still exists
- Tag every finding as one of:
  - **CURRENT** -- confirmed to still exist
  - **STALE** -- the file was moved, renamed, or deleted, or the function is gone
  - **UNVERIFIED** -- too abstract to check (an architectural pattern, for example)

### 5. Output

Present the results tightly:

```
## Prior Art: <topic>

### Related Documents
- `thoughts/shared/research/YYYY-MM-DD-<slug>.md` -- <one-line summary>
- `thoughts/shared/plans/YYYY-MM-DD-<slug>.md` -- <one-line summary>

### Current Findings (verified in codebase)
- <finding 1 with source doc reference>
- <finding 2>

### Stale Findings (referenced code has changed)
- <finding with what changed>

### Lessons from Past Work
- <lesson from retrospective or review, if any>

### Recommendation
<Build on the existing research, or start over? Which constraints still apply?>
```

If the search turns up nothing, state "No prior art found for this topic" and suggest moving on to `/research_codebase`.

### 6. Context Budget

Cap the whole output at 40 lines. It exists to feed the next command (`/research_codebase` or `/create_plan`): it is not itself a research document. When prior art is plentiful, keep only what is most recent and most relevant.

Do NOT write anything to disk. This output is throwaway context whose only job is to inform the following step.
