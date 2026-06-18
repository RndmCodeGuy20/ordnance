---
name: grill-me
description: >-
  Use when user wants to stress-test a plan or design, challenge assumptions,
  get grilled on their design, or mentions "grill me".
---

# Grill Me

Stress-test a plan or design by walking every branch of the decision tree, one question at a time, until every decision is explicitly resolved.

User input: $ARGUMENTS

**Usage:**
```
/grill-me                          # plan/design should already be in context
/grill-me ./plans/feature.md       # read plan from file
/grill-me 1234                     # grill a GitHub issue
```

Parse the input:
- If empty: assume the plan/design is in context. If not, ask for it.
- If a file path: read the file.
- If a GitHub issue number or URL: fetch with `gh issue view`.

---

## Iron Law

```
NO DECLARING "ALL RESOLVED" UNTIL EVERY BRANCH IS WALKED
ONE QUESTION AT A TIME — NO BATCHING
```

## HARD-GATE

```
Do NOT declare the plan stress-tested until every decision branch has been
explicitly asked, answered, and recorded. "Looks good overall" is not resolution.
```

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "The plan covers everything, no open questions" | Walk every branch. You WILL find gaps. |
| "I'll ask all 5 questions at once to save time" | ONE at a time. Batching hides dependency issues. |
| "Codebase exploration takes too long, just ask" | Explore first. Half the answers are in the code. |
| "User seems confident, no need to push further" | Your job is to push. Confidence ≠ correctness. |
| "This decision is obvious" | State your recommendation and ask anyway. |

## Red Flags — STOP

- About to batch multiple questions in one message
- About to declare "all resolved" without listing every resolved decision
- Skipping codebase exploration for a question the code could answer
- Accepting "yeah that's fine" without recording the specific resolution
- Moving to next question before recording the resolution of the current one

## Anti-Pattern: "The Rubber Stamp Grill"

Asking surface questions and accepting quick answers is not stress-testing. For each decision: probe the WHY, explore alternatives, check the codebase for conflicts. If the user says "yes" to everything in under 30 seconds, you're not grilling — you're rubber-stamping.

---

## Process

### 1. Load the plan/design

Get the content into context.

### 2. Map the decision tree

Identify all decision points, assumptions, and open questions. Group by dependency — some answers depend on others.

### 3. Walk branches one-by-one

For each decision point, one at a time:

1. **State the question clearly** — what needs to be decided?
2. **Explore the codebase first** — if the answer is discoverable from existing code, present findings rather than asking the user.
3. **Provide your recommended answer** with reasoning.
4. **Ask the user** to confirm, modify, or reject.
5. **Record the resolution** before moving to the next question.

**Dependency ordering**: don't ask about decision B if it depends on decision A. Resolve A first.

### 4. Resolve until done

Continue until every branch of the decision tree is resolved. Don't stop early.

### 5. Summarize

```
## Resolved Decisions

1. **<Decision>**: <Resolution> — <rationale>
2. **<Decision>**: <Resolution> — <rationale>
...

All branches resolved. Plan is stress-tested and ready.
```

If a plan file was provided, offer to update it with the resolved decisions.

---

## Composability

- **Often preceded by:** `decompose-plan`
- **Often followed by:** `create-sub-issues` or `create-issue`
- **Standalone use:** can grill any design, not just plans from the pipeline
