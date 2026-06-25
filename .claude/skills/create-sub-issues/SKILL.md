---
name: create-sub-issues
description: >-
  Use when user wants to create GitHub sub-issues from a plan or breakdown,
  convert plan phases to trackable issues, or break a parent issue into
  independently-grabbable tasks.
---

# Create GitHub Sub-Issues

Create GitHub sub-issues from an approved plan breakdown. Creates in dependency order and attaches each immediately.

User input: $ARGUMENTS

**Usage:**
```
/create-sub-issues 1234                           # parent issue #1234, breakdown in context
/create-sub-issues 1234 --plan ./plans/feature.md # parent + plan file
/create-sub-issues https://github.com/.../1234    # parent from URL
```

Parse the input:
- Extract parent issue number from number or URL.
- If `--plan <path>` provided: read the plan file.
- If no plan file: confirm breakdown is in context. If not, ask for it.

---

## Link Guard

Verify GitHub access before proceeding:
```bash
gh auth status
```
If not authenticated, abort: "Run `gh auth login`"

---

## Shell setup

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null)
OWNER=$(echo $REPO | cut -d/ -f1)
REPO_NAME=$(echo $REPO | cut -d/ -f2)
echo "REPO=$REPO OWNER=$OWNER REPO_NAME=$REPO_NAME"
```

---

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "GraphQL sub-issue API failed, just create standalone issues" | Fall back to checklist comment. Tell the user. Don't silently degrade. |
| "I'll create issues in parallel for speed" | Dependency order is mandatory. Blockers first. |
| "I'll create all issues first, then attach them" | Attach IMMEDIATELY after each creation. Batching causes orphaned issues. |
| "This slice is too small for its own issue" | If it's in the plan, it gets an issue. |

---

## Process

### 1. Load the breakdown

If a plan file was provided, read it. Each slice requires: title, user story, what to build, and acceptance criteria. If any required field is absent for a slice, ask the user to provide it before creating that issue. Do not fabricate acceptance criteria.

### 2. Quiz the user

Present the proposed breakdown. For each phase show:
- **Title**
- **Type**: HITL / AFK
- **Blocked by**: which issues (if any) must complete first
- **User stories covered**
- **Layers**: which layers this phase cuts through (schema, service, API, tests as applicable)

Note: even if the breakdown came from a pre-approved `decompose-plan` run, re-confirm HITL/AFK and dependency ordering here — the plan may have changed, or this skill may be running without a prior decompose-plan run.

If the user changes dependency ordering during the quiz, re-present the updated ordering for explicit confirmation before proceeding.

Iterate until the user approves.

### 3. Get parent issue node ID

```bash
PARENT_ID=$(gh api graphql -f query='
  query {
    repository(owner: "'$OWNER'", name: "'$REPO_NAME'") {
      issue(number: <parent_number>) {
        id
      }
    }
  }
' -q '.data.repository.issue.id')
echo "PARENT_ID=$PARENT_ID"
```

### 4. Create issues in dependency order — attach each immediately

**Create blockers first** so their issue numbers can be referenced in "Blocked by".

**For each slice: create the issue AND attach it as a sub-issue BEFORE moving to the next.** Do not batch.

**Step A — Create the issue:**

```bash
CHILD_NUM=$(gh issue create --repo $REPO \
  --title "<slice title>" \
  --label "<label1>,<label2>" \
  --body "$(cat <<'EOF'
## Parent

#<parent-issue-number>

## User story

As a [role], I want [capability] so that [benefit].

## Implementation details

<End-to-end behavior description. Reference parent issue sections rather than duplicating.>

### Layers touched

- **Schema**: <changes, if any>
- **API**: <endpoint / service behavior>
- **Tests**: <derived from acceptance criteria — each criterion becomes a failing test before implementation>

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- #<issue-number> (if any, else "None — can start immediately")

## Type

**AFK** / **HITL**
<if HITL, explain what human decision is needed>
EOF
)" | grep -oP '\d+$')
```

If `CHILD_NUM` is empty, `gh issue create` failed. Abort this slice with the error message. Do not proceed to Step B.

**Step B — Immediately attach as sub-issue:**

```bash
CHILD_ID=$(gh api graphql -f query='
  query {
    repository(owner: "'$OWNER'", name: "'$REPO_NAME'") {
      issue(number: '$CHILD_NUM') {
        id
      }
    }
  }
' -q '.data.repository.issue.id')

gh api graphql -f query='
  mutation {
    addSubIssue(input: {
      issueId: "'$PARENT_ID'"
      subIssueId: "'$CHILD_ID'"
    }) {
      issue { id }
    }
  }
'
```

If `addSubIssue` mutation fails (feature not enabled for the repo), fall back to a checklist comment on the parent issue. Tell the user.

**Then move to the next slice.**

### 5. Summary

```
## Sub-Issues Created

| # | Title | Type | Blocked By | Labels |
|---|-------|------|------------|--------|
| #<num> | <title> | AFK | — | backend |
| #<num> | <title> | HITL | #<num> | frontend |

All issues attached as sub-issues to #<parent>.
```

---

## Rules

1. **Vertical, not horizontal**: every issue must cut through multiple layers (schema, service, API, tests as applicable).
2. **Dependency order**: create blocker issues first so real issue numbers can be referenced.
3. **Quiz before creating**: never create issues until the user approves the breakdown.
4. **Parent issue**: don't close or edit the parent issue body. A checklist comment on the parent is allowed only as a fallback when `addSubIssue` fails.
5. **HITL vs AFK**: AFK = no open decisions, no approvals, no required human interaction. If in doubt, mark as HITL.
6. **Human-readable titles**: no dev jargon. Write titles anyone can understand.

## Composability

- **Often preceded by:** `decompose-plan` — pass the plan file via `--plan`
- **Execution strategy by type:**
  - **AFK issues** → `superpowers:subagent-driven-development`
  - **HITL issues** → `superpowers:executing-plans`
  - **Independent AFK issues** → `superpowers:dispatching-parallel-agents`
- **Pairs with:** `commit`, `pr-create`
