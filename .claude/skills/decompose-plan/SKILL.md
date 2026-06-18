---
name: decompose-plan
description: >-
  Use when user wants to break down a PRD, GitHub issue, or design into an
  implementation plan. Mentions "tracer bullets", "vertical slices", or has a
  spec that needs decomposition before coding starts.
---

# Decompose into Plan

Break a PRD, GitHub issue, or design into a phased implementation plan using vertical slices (tracer bullets). Output is a Markdown file saved to the project's plans directory.

User input: $ARGUMENTS

**Usage:**
```
/decompose-plan                          # PRD should already be in context
/decompose-plan path/to/prd.md           # read PRD from local file
/decompose-plan 1234                     # GitHub issue #1234
/decompose-plan https://github.com/...   # GitHub issue URL
```

Parse the input:
- If empty: assume the PRD is already in the conversation context. If not, ask for it.
- If a file path: read the file.
- If a GitHub issue URL or number: fetch with `gh issue view <number> --repo $REPO --comments`.

---

## Iron Law

```
NO PLAN WITHOUT USER QUIZ APPROVAL FIRST
NO PROCEEDING WITH INACCESSIBLE LINKS
```

## HARD-GATE

```
Do NOT write the plan file until the user has approved the breakdown.
Do NOT proceed if any linked content (GitHub, Figma, Notion) is inaccessible.
```

---

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "The PRD is detailed enough, skip the quiz" | ALWAYS quiz. Users catch blind spots you won't. |
| "I can't access the Figma link but I have enough context" | Linked content is critical. ABORT. |
| "This only touches one layer, horizontal slices are fine" | Vertical slices. No exceptions. |
| "The user seems impatient, skip brainstorming" | Rushing = rework. Brainstorm first. |
| "This is a small feature, one phase is enough" | Small features still get decomposed. |

## Red Flags — STOP

- About to write plan without quizzing user
- Creating horizontal slices (all migrations, all UI, all tests)
- Putting file names or function signatures in the plan
- Proceeding after a link returned 404 or auth error

---

## Process Flow

```
Input → Links accessible? → Brainstorm → Explore codebase →
Draft vertical slices → Quiz user → Approved? → Write plan file
```

---

## Process

### 1. Link Guard

Verify access to any linked content before proceeding:
- **GitHub** → `gh auth status`
- **Figma** → verify MCP server
- **Notion / Confluence / Linear** → verify access

Do not proceed with partial information.

### 2. Load the PRD/issue

Fetch from GitHub, read file, or confirm content is in context.

If the input is a lightweight issue (no user stories, no acceptance criteria), prompt:
> "This issue looks lightweight — what are the key user stories and scope boundaries?"

### 3. Brainstorm

Use `superpowers:brainstorming` if available to explore intent, clarify scope, and align on requirements before planning. This surfaces blind spots that decomposition alone misses.

### 4. Explore the codebase

Understand what exists before slicing. Focus on:
- Data model and schema
- API layer and existing endpoints
- Auth / permission model
- Key service boundaries

### 5. Identify durable architectural decisions

Before slicing, identify decisions that persist across all phases:
- Data model shape and relations
- API design (REST / GraphQL / events)
- Auth approach
- Edition or deployment scope (if applicable)

These go in the plan header so every phase can reference them.

### 6. Draft vertical slices

Break the PRD into **tracer bullet** phases. Each phase is a thin vertical slice cutting through ALL layers end-to-end.

Classify each slice as:
- **HITL** (Human In The Loop): requires human decision-making
- **AFK** (Away From Keyboard): can be implemented by an agent without human interaction

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every relevant layer:
  - Schema / migration (if data changes)
  - Service / backend logic
  - API endpoint
  - Tests
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Do NOT include specific file names, function names, or implementation details
- DO include durable decisions: route paths, schema shapes, data model names
</vertical-slice-rules>

### 7. Quiz the user

Present the proposed breakdown. For each phase show:
- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which phases must complete first
- **User stories covered**
- **Layers**: which layers this phase cuts through

Ask:
- Does the granularity feel right?
- Are dependency relationships correct?
- Should any phases be merged or split?
- Are HITL/AFK classifications correct?

Iterate until the user approves.

### 8. Write the plan file

```
./plans/<feature-name>.md
```

<plan-template>
# Plan: <Feature Name>

> Source: <link, issue number, or brief identifier>

## Architectural decisions

Durable decisions that apply across all phases:

- **Data model**: schema shape, key relations
- **API design**: REST / GraphQL / events, route conventions
- **Auth**: permission model, guards
- (add/remove sections as appropriate)

---

## Phase 1: <Title>

**User stories**: <list from PRD>
**Type**: HITL / AFK
**Blocked by**: — / Phase N

### What to build

A concise description of this vertical slice. Describe end-to-end behavior, not layer-by-layer implementation.

### Layers

- **Schema**: <changes, if any>
- **Service**: <logic / behavior>
- **API**: <endpoint behavior>
- **Tests**: <derived from acceptance criteria>

### Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

---

## Phase 2: <Title>

<!-- Repeat for each phase -->
</plan-template>

---

## Rules

1. **Vertical, not horizontal**: every phase must cut through multiple layers.
2. **Durable decisions only**: no file names or function signatures — they change.
3. **Quiz before writing**: never write the plan file until the user approves.
4. **One plan file per feature**: if re-running, update the existing file.
5. **Abort on inaccessible links**: stop and tell the user what to set up.
6. **HITL vs AFK**: if in doubt, mark as HITL.

## Composability

- **Often followed by:** `grill-me` to stress-test the plan, then `create-sub-issues`
- **Pairs with:** `commit`, `pr-create`
