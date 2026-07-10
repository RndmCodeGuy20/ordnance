# ordnance

Curated Claude Code skills for backend engineering work — plan, stress-test, decompose, ship. Each skill lives in `.claude/skills/<name>/SKILL.md` and is auto-discovered by Claude Code; invoke with `/<name>`.

## Plan & design

| Skill | Use it when |
|---|---|
| [`architecture-review`](.claude/skills/architecture-review/SKILL.md) | Any feature that changes system architecture or needs a migration — writes the mandatory pre-dev architecture doc (API/Backend + Data Management) and preps it for reviewer sign-off. |
| [`design-api`](.claude/skills/design-api/SKILL.md) | Designing a REST API contract section-by-section (resources, endpoints, auth, errors, pagination). |
| [`grill-me`](.claude/skills/grill-me/SKILL.md) | Interactively stress-testing a plan/design — walks every open decision, one question at a time. |
| [`red-team`](.claude/skills/red-team/SKILL.md) | Adversarially stress-testing a plan with 4 CIA-Tradecraft-style passes (assumptions check, pre-mortem, rival implementation, on-call postmortem) before it survives to implementation. Run before `grill-me` to kill weak plans fast; findings are graded by severity, not a computed verdict. |
| [`decompose-plan`](.claude/skills/decompose-plan/SKILL.md) | Breaking a PRD, issue, or design into an implementation plan using tracer-bullet vertical slices. |

## Issue tracking

| Skill | Use it when |
|---|---|
| [`create-issue`](.claude/skills/create-issue/SKILL.md) | Filing a single GitHub issue or appending context to an existing one. |
| [`create-sub-issues`](.claude/skills/create-sub-issues/SKILL.md) | Converting a plan's phases into independently-grabbable GitHub sub-issues. |

## Implementation support

| Skill | Use it when |
|---|---|
| [`code-commenting`](.claude/skills/code-commenting/SKILL.md) | Deciding whether a comment is worth writing while editing, fixing, or refactoring code. |
| [`systematic-debug`](.claude/skills/systematic-debug/SKILL.md) | Diagnosing a bug or test failure — traces to root cause and proposes a fix plan without implementing. |
| [`postman-to-openapi`](.claude/skills/postman-to-openapi/SKILL.md) | Converting a Postman collection into an OpenAPI 3.0 spec (YAML + JSON). |

## Review & ship

| Skill | Use it when |
|---|---|
| [`code-review`](.claude/skills/code-review/SKILL.md) | Reviewing the current diff for reasoning quality and decision soundness, not just style. |
| [`commit`](.claude/skills/commit/SKILL.md) | Committing staged/unstaged changes with an auto-generated conventional commit message, incl. submodules. |
| [`pr-create`](.claude/skills/pr-create/SKILL.md) | Pushing the branch and opening a pull request with a generated description. |
| [`pr-switch`](.claude/skills/pr-switch/SKILL.md) | Checking out an existing PR locally by link or number. |

## Docs & ops

| Skill | Use it when |
|---|---|
| [`release-notes`](.claude/skills/release-notes/SKILL.md) | Writing user-facing release notes / changelog entries for a feature or fix. |
| [`log-decision`](.claude/skills/log-decision/SKILL.md) | Logging a non-trivial engineering decision or implementation to the Obsidian vault. |

## Typical flow

```
architecture-review (if arch/migration change) → design-api / grill-me / red-team  →  decompose-plan  →  create-sub-issues
        →  (implement, using code-commenting / systematic-debug as needed)
        →  code-review  →  commit  →  pr-create
```
