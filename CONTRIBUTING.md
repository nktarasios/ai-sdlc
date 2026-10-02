# Contributing

This repo is a compact, opinionated toolkit rather than a framework, and its scope is kept deliberately tight, contributions are welcome inside those bounds.

## Ways to contribute

- **Sharpen a skill.** If working with one of the files in `skills/` surfaced a crisper way to state a rule, or exposed a guardrail that should exist and doesn't, send a PR against that one file. Two constraints: the stage boundaries stay where they are (the README explains which failure each stage exists to block), and the skills stay project-neutral. Anything specific to your project belongs in your own `CLAUDE.md`, never in these files.
- **Add a case study.** Ran the loop on a real, public project of your own? Document it in this shape:
  - `## Task`: a sentence or two naming the actual feature or task the study covers.
  - One `## <Stage>` section for every stage you exercised (Research, Plan, Test, Implement, Validate, Review), each containing the genuine artifacts that stage produced, linked to real files and commits in your public repository.
  - `## Outcome`: a sentence or two on what shipped, and whether running the loop altered the result in any way.
  A case study must point at real, public, checkable artifacts, hypotheticals don't qualify. And if some stage came up empty in your example, write that down plainly; do not manufacture a finding to fill the gap.
- **Report an issue.** When a skill behaves confusingly or produces something unhelpful on a real task, open an issue that describes the task and where things went sideways.

## What this project is not looking for

- Restructuring proposals, merging stages, inventing new ones, reorganizing the whole thing. If you believe the core loop itself needs to change, start with an issue and make the case there first.
- Tooling or framework additions (IDE plugins and the like). This remains a repo of documentation and skills. The one piece of code the skills reference, the autopilot conductor script, is deliberately **not** bundled; `skills/autopilot.md` documents the contract yours must satisfy, and PRs adding it (or any other tooling) here are out of scope.
