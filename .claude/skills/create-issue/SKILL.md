---
name: create-issue
description: >-
  Use when user wants to create a single GitHub issue, append context to an
  existing issue, or file a ticket. Escalates to create-sub-issues if scope
  requires multiple vertical slices.
---

# Create or Update GitHub Issue

Create a new GitHub issue or append to an existing one. If the work involves multiple vertical slices, delegate to `create-sub-issues`.

User input: $ARGUMENTS

**Usage:**
```
/create-issue                     # create from context
/create-issue 1234                # append to existing issue #1234
/create-issue path/to/spec.md     # create from file
```

Parse the input:
- If a number: append mode — add content as a comment to that issue.
- If a file path: read the file as the issue content source.
- If empty: assume the issue content is in the conversation context.

---

## Iron Law

```
NO ISSUE WITHOUT PROPER TEMPLATE
NO APPENDING WITHOUT PRESERVING ORIGINAL BODY
```

---

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "This work has multiple phases but they're small" | Multiple phases = sub-issues. Use create-sub-issues. |
| "User didn't specify labels" | Infer labels from content. Always label. |
| "Scope is obvious, skip the template" | Template is not optional. Every issue gets it. |
| "This user story is close enough" | Must follow "As a [role], I want [capability] so that [benefit]" exactly. |
| "I'll just edit the existing issue body" | APPEND ONLY. Never modify original. |

---

## Link Guard

Verify access to any linked content before proceeding:
- **GitHub** → `gh auth status`. Abort → "Run `gh auth login`"
- **Figma / Notion / Linear** → verify MCP server access

Do not proceed with partial information.

---

## Shell setup

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null)
echo "REPO=$REPO"
```

---

## Process

### 1. Determine mode

- **Append mode** (issue number provided): add content as a comment using `gh issue comment <number> --repo $REPO --body "..."`. Never overwrite the original issue body.
- **Create mode** (no issue number): proceed to step 2.

### 2. Assess scope

Before creating, evaluate the work scope:
- If the work involves **multiple vertical slices**: delegate to `create-sub-issues`.
- If the work is a **single coherent unit**: proceed to create.

### 3. Create the issue

```bash
gh issue create --repo $REPO \
  --title "<title>" \
  --label "<label1>,<label2>" \
  --body "$(cat <<'EOF'
## User story

As a [role], I want [capability] so that [benefit].

## Implementation details

<Concise description of what to build end-to-end>

### Layers touched

- **Schema**: <changes, if any>
- **API**: <endpoint / service behavior>
- **Tests**: <what to verify>

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3
EOF
)"
```

### 4. Report

Print the created issue URL and a one-line summary.

---

## Rules

1. **Append-only for existing issues**: never modify the original body. Always add a comment.
2. **Abort on inaccessible links**: stop and tell the user what to set up.
3. **Scope check**: if work is too large for a single issue, delegate to `create-sub-issues`.
4. **Human-readable titles**: no dev jargon or internal planning terms ("tracer bullet", "app-context"). Write titles anyone on the team can understand.
5. **Always label**: infer appropriate labels if the user didn't specify.

## Composability

- **Escalates to:** `create-sub-issues` when scope exceeds a single issue
- **Often preceded by:** `decompose-plan` or `grill-me`
- **Pairs with:** `commit`, `pr-create`
