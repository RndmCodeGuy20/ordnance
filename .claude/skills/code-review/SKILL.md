---
name: code-review
description: >-
  Review the current diff for reasoning quality and decision soundness.
  Challenges the thinking behind code — not just style. Trigger on: "review
  this", "review my changes", "check my diff", "question my decisions",
  "is this good". Two verdict categories: PERSONAL (your CLAUDE.md principles)
  and GENERAL (universal engineering concerns).
---

# Code Review

Review the current diff for decision quality. Each finding is a verdict block —
decision identified, challenge posed, explicit verdict. Output to terminal only.

User input: $ARGUMENTS

**Usage:**
```
/code-review                    # review current working tree diff
/code-review --staged           # review staged changes only
/code-review path/to/file.go    # review specific file's changes
```

---

## Phase 1 — Get the diff

```bash
git diff HEAD
```

If working tree is clean, try staged:
```bash
git diff --cached
```

If `$ARGUMENTS` contains a file path, scope to that file:
```bash
git diff HEAD -- <path>
```

If diff is empty after both attempts, stop: "Nothing to review — working tree and staging area are both clean."

---

## Phase 2 — Parse

Walk the diff. For each changed file, identify:
- New functions, methods, types, interfaces
- Modified logic blocks
- Error handling (or absence of it)
- New abstractions (interfaces, generics, helpers)
- Exports added or removed
- Scope of the change relative to what the surrounding context suggests was needed

Group changes by file. Understand the intent of the change before evaluating it — don't flag a decision without understanding what it was trying to do.

---

## Phase 3 — PERSONAL pass

Check against CLAUDE.md principles. Flag any violation as a finding.

| Principle | What to look for |
|---|---|
| **Explicit over clever** | Chained one-liners, implicit coercions, overloaded operators, "magic" that requires knowing the language deeply to parse |
| **No premature abstraction** | Any interface, generic, or helper introduced that has exactly one call site in the diff with no clear second use visible |
| **Errors first-class** | Bare `_` on error return (Go), swallowed `.catch()` or unhandled rejection (TS), `fmt.Errorf` without `%w` on a wrappable error, `try/catch` that logs and continues without returning |
| **No `any`** | TypeScript `any` type explicit or inferred, `Promise<any>`, return type omitted on async function |
| **No naked returns** (Go) | Function with multiple return paths using bare `return` — all returns must be explicit |
| **Unexported by default** (Go) | Exported identifier with no consumer visible outside the changed files |
| **Scope creep** | New function, handler, config flag, or type that wasn't required by the stated change — YAGNI |
| **Naming** | Name describes how it works, not what it does (`handleRequest` vs `authorizeUserRequest`); abbreviations that require context to decode |

---

## Phase 4 — GENERAL pass

Check broader engineering concerns independent of personal standards.

| Concern | What to look for |
|---|---|
| **Coupling** | Depends on a concrete type where an interface would decouple; imports an entire package for one function |
| **Untested edge cases** | Visible in the diff: null/nil input path, empty collection, zero value, concurrent access |
| **Inconsistency** | Naming, error handling, or structure inconsistent with the surrounding unchanged code |
| **Validation placement** | Validates at the wrong boundary (internal function validating what the handler should have caught) or not at all at the system boundary |
| **Hidden state** | Function behaviour depends on mutable state not in its signature |
| **Commented-out code** | Dead code left in the diff |
| **Over-defensive** | Handling scenarios that cannot occur given caller guarantees |

---

## Phase 5 — Output verdict blocks

One block per finding. Order: PERSONAL findings first, GENERAL second. Within each group, order by severity (CHANGE RECOMMENDED → NEEDS JUSTIFICATION → ACCEPTED).

```
### [PERSONAL] <short title>

**Decision:** <one sentence: what the code does / what choice was made>
**Challenge:** <the pointed question — why this approach, what happens when X, what's the second use case>
**Verdict:** CHANGE RECOMMENDED | NEEDS JUSTIFICATION | ACCEPTED
```

**Verdict definitions:**
- `CHANGE RECOMMENDED` — the decision is likely wrong; a concrete alternative is clear
- `NEEDS JUSTIFICATION` — the decision may be correct but the reasoning isn't visible in the code; requires explanation or a comment
- `ACCEPTED` — non-obvious choice that is actually sound; acknowledge it was deliberate

When verdict is `CHANGE RECOMMENDED`, add one line:
```
**Suggested fix:** <specific, not prescriptive — what to change, not the full implementation>
```

---

## Phase 6 — Summary

```
---

## Review Summary

| Category | CHANGE RECOMMENDED | NEEDS JUSTIFICATION | ACCEPTED |
|---|---|---|---|
| PERSONAL | N | N | N |
| GENERAL | N | N | N |

<If 0 findings across all categories: "No issues found. Diff is clean against both personal and general criteria.">
```

---

## Rules

1. **Understand before challenging** — read enough surrounding context to understand the intent. Don't flag a decision you haven't understood.
2. **One finding per decision** — don't split one decision into multiple findings. Don't combine two decisions into one.
3. **Challenge the reasoning, not the style** — formatting, whitespace, and aesthetic preferences are not findings.
4. **ACCEPTED is meaningful** — use it when the code makes a non-obvious but correct choice. An empty ACCEPTED column is a signal you only flagged problems.
5. **Concrete challenges** — "Is this right?" is not a challenge. "What happens when this list is empty and the caller expects at least one element?" is.
6. **No implementation in the review** — suggested fix names the change, doesn't write the code.

## Composability

- **Often followed by:** implementing fixes, or `grill-me` if the findings surface a deeper design question
- **Pairs with:** `systematic-debug` when a finding points to a potential bug
- **Trigger after:** any non-trivial implementation before running `pr-create`
