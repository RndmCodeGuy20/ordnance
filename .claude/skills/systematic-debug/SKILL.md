---
name: systematic-debug
description: >-
  Diagnose any bug, test failure, or unexpected behavior. Traces to root cause
  and produces a fix plan without implementing. Trigger on: error messages,
  stack traces, failing tests, "why is X doing Y", "this isn't working",
  "help me debug". Also trigger proactively when encountering unexpected behavior
  during any task.
---

# Systematic Debug

Trace a bug to its root cause and produce a concrete fix plan. Claude drives the investigation — you review findings.

User input: $ARGUMENTS

**Usage:**
```
/debug                              # paste error/stack trace, or describe behavior
/debug "cannot read property of undefined in AuthService"
/debug "TestUserCreate fails on line 42"
/debug "POST /orders returns 200 but order never saves"
```

---

## Phase 1 — Classify entry

Detect what was provided in `$ARGUMENTS` + conversation context:

| Entry type | Signals | Investigation start |
|---|---|---|
| **Stack trace / error message** | File paths, line numbers, error type | Extract symbols → codegraph trace |
| **Failing test** | Test name, assertion, framework | Read test → identify what's being asserted |
| **Behavioral description** | "should do X but does Y" | Identify the surface area → trace inward |

If the entry type is ambiguous, classify as behavioral and proceed.

---

## Phase 2 — Parse and extract

### Stack trace / error message

Extract:
- Error type and message (verbatim)
- File paths and line numbers mentioned
- Function/method names in the stack

**Go-specific signals:**
- `panic: runtime error: invalid memory address or nil pointer dereference` → nil receiver or uninitialized pointer
- `panic: interface conversion: interface is nil` → type assertion on nil
- `fatal error: concurrent map read and map write` → missing mutex
- `goroutine N [chan receive]` in goroutine dump → channel deadlock
- Unwrapped error chains → missing `%w` in `fmt.Errorf`

**TypeScript-specific signals:**
- `TypeError: Cannot read properties of undefined` → optional chaining missing or async not awaited
- `Type 'X' is not assignable to type 'Y'` → type narrowing needed or wrong type at boundary
- `ReferenceError: X is not defined` → import missing or circular dependency
- Unhandled promise rejection without stack → missing `await` or `.catch`

### Failing test

Extract:
- Test file and test name
- The assertion that failed (expected vs. actual)
- Which function/method is being tested

### Behavioral description

Extract:
- The surface area (endpoint, function, UI action, CLI command)
- Expected behavior
- Actual behavior
- Any conditions that trigger it (specific inputs, state, timing)

---

## Phase 3 — Trace (codegraph-first)

Use codegraph before reading any source files. Limit to 3-4 targeted lookups.

**Sequence:**
1. `codegraph_search` — find the symbol named in the error or the function under test
2. `codegraph_trace` — trace the call path from entry point to the failure site
3. `codegraph_context` — get focused context on the area around the failure
4. `codegraph_callers` — find what calls the failing function (if root not yet clear)

Only open source files when codegraph points to a specific location. Read the minimum needed — the function body containing the failure, not the entire file.

Check git history at the failure site if the bug is recent:
```bash
git log -p --follow -S "<symbol or string>" -- <file> | head -60
```

---

## Phase 4 — Generate and eliminate hypotheses

Generate 2-4 ranked hypotheses. For each:
- What it explains
- One piece of evidence for it
- One thing that would disprove it

Eliminate by checking against code and git history. Cross off any hypothesis that the code contradicts. If all hypotheses are eliminated, generate new ones — do not stop.

---

## Phase 5 — Confirm root cause and write fix plan

Once one hypothesis survives and is supported by evidence, confirm it and output:

```
## Root Cause

<One sentence. Name the specific function, condition, or missing check.>

## Evidence

- <file:line> — <what the code shows>
- <git commit or blame line if relevant> — <what changed>

## Fix Plan

- <file:line> — <what to change, described precisely. No implementation.>
- <file:line> — <any additional change required>

## Confidence: High / Medium / Low

<Only if not High: explain what's uncertain and what would confirm it.>
```

Stop here. Do not implement the fix unless explicitly asked.

---

## Rules

1. **Codegraph before grep** — use structural search first; grep only for literal strings codegraph can't index (log messages, comments).
2. **Minimum reads** — read the function containing the failure, not the whole file.
3. **No silent assumptions** — if the root cause is uncertain, say so and state confidence.
4. **Fix plan = description, not code** — name the change precisely without writing the implementation.
5. **One root cause** — if multiple bugs are found, report them separately. Don't conflate.

## Composability

- **Often followed by:** implementation (manual or via `superpowers:test-driven-development`)
- **Pairs with:** `grill-me` to stress-test a proposed fix before implementing
