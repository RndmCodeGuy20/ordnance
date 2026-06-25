---
name: log-decision
description: >-
  Log a non-trivial engineering decision or implementation to the Obsidian vault.
  Trigger on: "log this", "add to vault", "document this". Also trigger when
  completing a non-trivial implementation (new abstraction, service, pattern,
  data model change) if the user hasn't explicitly asked to skip logging.
---

# Log Decision

Extract the most significant decision from context, search the vault index for
related entries, draft the schema entry, and write it after confirmation.

User input: $ARGUMENTS

**Usage:**
```
/log-decision                    # extract from current conversation context
/log-decision auth middleware    # hint at which decision to log
```

Vault path: `~/Documents/Obsidian/.mind`
Index path: `~/Documents/Obsidian/.mind/index.md`

---

## Phase 1 — Extract from context

Identify the most significant non-trivial decision or implementation from the
conversation. Prefer the most recent substantive one. Draft all schema fields:

**Infer automatically:**
- `category`: `feature` (new capability) | `fix` (corrected behaviour) | `arch` (structural/cross-cutting) | `summary` (multi-decision recap)
- `stack`: `go` | `ts` | `both` — from what was discussed
- `complexity`: `low` (straightforward, one layer) | `medium` (multi-layer, non-obvious tradeoffs) | `high` (cross-cutting, significant tradeoffs, broad impact)
- `status`: `active` (default)
- `supersedes`: leave blank until Phase 2
- `related`: leave blank until Phase 2

**Draft content sections:**
- **What** — 1-3 lines: what was built or decided
- **Why** — actual reason: constraints, tradeoffs, what was rejected
- **How (key parts)** — signatures, data shapes, patterns — enough to jog memory
- **Gotchas** — non-obvious things that will bite later
- **Open threads** — unresolved questions, future considerations

If a field cannot be inferred confidently, leave a `<TBD>` placeholder — do not fabricate.

---

## Phase 2 — Search index

Read `~/Documents/Obsidian/.mind/index.md`.

If the file does not exist, create it:
```markdown
# .mind Index

<!-- Maintained by log-decision. Do not edit entry blocks manually. -->

## Entries

```
Then continue with an empty match set.

**Match logic:**
1. Extract 3-6 keywords from the current decision's What + Why + How
2. Scan each index entry's **Topics** and **What** fields for overlap
3. Classify matches:
   - **Potential supersedes**: `Status: active` entry on the same topic — this decision may replace it
   - **Related**: any other entry with ≥2 keyword overlaps

**Populate fields:**
- `supersedes`: slug of the potential supersedes entry (confirm with user in Phase 3)
- `related`: list of related entry slugs

---

## Phase 3 — Draft and confirm

Present the full drafted entry to the user:

```
## Draft entry

**Path:** ~/Documents/Obsidian/.mind/{category}/{decision-name}.md

---
tags: [decision, {category}, {stack}]
status: {status}
supersedes: "{supersedes slug or blank}"
related: [{related slugs or blank}]
complexity: {complexity}
---

# {Title}

## What
{what}

## Why
{why}

## How (key parts)
{how}

## Gotchas
{gotchas}

## Open threads
{open threads}
```

If a supersedes match was found, also show:
> "This looks like it may supersede **{slug}** (`{path}`): _{one-line What from that entry}_. Confirm?"

Wait for user to approve or amend any field before writing.

---

## Phase 4 — Write the entry

```bash
# Ensure category directory exists
mkdir -p ~/Documents/Obsidian/.mind/{category}
```

Write the file to `~/Documents/Obsidian/.mind/{category}/{decision-name}.md`.

Use this exact template:
```markdown
---
tags: [decision, {category}, {stack}]
status: {status}
supersedes: "{supersedes}"
related: [{related}]
complexity: {complexity}
---

# {Title}

## What

{what}

## Why

{why}

## How (key parts)

{how}

## Gotchas

{gotchas}

## Open threads

{open threads}
```

---

## Phase 5 — Update index

### Append new entry block

Add to the bottom of the `## Entries` section in `index.md`:

```markdown
### {decision-name}
- **Path**: {category}/{decision-name}.md
- **Cat**: {category} | **Stack**: {stack} | **Status**: {status}
- **Topics**: {comma-separated keywords}
- **What**: {one-line summary of What section}
```

### Mark superseded entry (if applicable)

In the superseded entry's index block, replace:
```
- **Cat**: {cat} | **Stack**: {stack} | **Status**: active
```
with:
```
- **Cat**: {cat} | **Stack**: {stack} | **Status**: superseded
```

Do NOT update the superseded entry's actual `.md` file — index only.

---

## Phase 6 — Confirm

```
✓ Written: ~/Documents/Obsidian/.mind/{category}/{decision-name}.md
✓ Index updated: {N} total entries
{if superseded: ✓ Marked superseded: {slug}}
```

---

## Rules

1. **Draft before writing** — always show the full entry and wait for confirmation. Never write without approval.
2. **Infer, don't ask** — extract category, stack, complexity from context. Only ask if genuinely ambiguous after reading context.
3. **No fabrication** — if a section can't be inferred confidently, use `<TBD>` and let the user fill it.
4. **Index is one-directional** — new entries link to related; existing entries are not updated to point back.
5. **Index over file traversal** — always read index.md for search. Never list or read individual vault files unless the user explicitly asks to open one.
6. **Supersedes requires confirmation** — never mark something superseded without the user explicitly agreeing.

## Composability

- **Trigger phrases (from CLAUDE.md):** "log this", "add to vault", "document this"
- **Often follows:** any non-trivial implementation, `design-api`, `decompose-plan`
- **Index location:** `~/Documents/Obsidian/.mind/index.md`
