---
name: code-commenting
description: >
  Rules for writing (and not writing) code comments when editing, fixing, or refactoring code.
  Use this skill whenever a coding agent is about to add, modify, or review comments alongside
  code changes. Trigger on: bug fixes, refactors, new functions, edits to existing code, or
  any task where inline comments might be added. This skill governs comment quality only —
  it does not affect reasoning, code logic, or structure.
---

# Code Commenting Skill

This skill defines when and how to write code comments. Its primary goal is **restraint**: most
comments should not exist. The ones that do should earn their place.

---

## The Core Rule

> Comment the **why**, never the **what**.

If a reader can understand a comment by reading the code itself, delete the comment.
Code is already a description of what happens. A comment that restates it is noise.

**Bad** (explains what):
```python
# increment the counter
count += 1

# check if user is authenticated
if user.is_authenticated:
```

**Good** (explains why):
```python
# rate limiter uses token bucket; start at max capacity so first request isn't penalized
tokens = MAX_TOKENS

# billing expects 1-indexed months; datetime gives 0-indexed
month = dt.month + 1
```

---

## The Five Questions (run through these before adding any comment)

1. **Would a new engineer understand this code without the comment?**
   If yes → no comment needed.

2. **Does this comment say anything the code doesn't already say?**
   If no → delete it.

3. **Is this explaining a non-obvious decision, constraint, or tradeoff?**
   If yes → comment is probably warranted.

4. **Am I narrating an edit or fix I just made?**
   If yes → never write this. It belongs in a commit message, not source code.

---

## What Belongs in a Comment

Write a comment when one of these is true:

- **Non-obvious constraint**: Something external forces this code to work a certain way
  (an API quirk, a business rule, a legacy requirement, a known bug in a dependency).
- **Deliberate tradeoff**: You chose a slower/simpler/worse approach for a specific reason
  (readability, thread safety, compatibility). Future devs will be tempted to "fix" it.
- **Subtle algorithm or math**: The logic is correct but requires domain knowledge to verify.
- **Warning to future maintainers**: "Don't remove this — it prevents X from happening."

---

## What Never Belongs in a Comment

These comment types should always be removed or rewritten:

| Type | Example | Why it's bad |
|---|---|---|
| Restates code | `// returns true if active` on `return isActive` | Redundant |
| Edit narration | `// fixed null check here` | Use commit messages |
| Vague attribution | `// handles edge case` | Which edge case? Why? |
| Unexplained project jargon | `// needed for the Zephyr migration` | Meaningless to newcomers |
| Obvious setup | `// initialize the list` above `items = []` | Patronizing |
| Step-by-step walkthrough | `// Step 1: validate... // Step 2: fetch...` | Code structure should show this |
| Commented-out code | `// old_fn(x, y)` | Delete it; git has history |

---

## Style Rules

- **Keep it short.** If a comment needs more than 2 lines, ask whether the code itself needs
  restructuring first. Only genuinely complex decisions warrant more.
- **Write for a newcomer to the codebase.** Avoid unexplained acronyms, internal project
  names, or references to past decisions without context.
- **Don't start with "This function..." or "This method..."** — the function name already
  says what it is.
- **Don't use hedging language** like "basically", "simply", "just", or "obviously".
- **Comments are not documentation.** Public API docs (docstrings, JSDoc, godoc, etc.) are a
  separate concern and follow their own conventions. This skill governs inline comments only.

---

## Language-Specific Notes

### Go
- Exported symbols (`func`, `type`, `const`, `var`) must have a godoc comment in the form
  `// SymbolName does X` — this is enforced by tooling (`golint`, `staticcheck`).
- Unexported internals: apply the core rule — comment the why only, never the what.
- No block comments inside function bodies; Go style strongly disfavors them.
- TODOs: prefer `// TODO(username): explanation` or `// TODO(#issue): explanation` — bare
  `// TODO: clean this up` is noise.

### Python
- Docstrings on public functions/classes are fine and expected — but the body should explain
  *purpose and usage*, not walk through the implementation line by line.
- Inline `#` comments should be rare.

### JavaScript / TypeScript
- JSDoc is acceptable for exported functions. Keep it brief.
- Avoid comment blocks inside function bodies.

### All languages
- Never leave `TODO` or `FIXME` without an issue number or owner.
  `// TODO: clean this up` is noise. `// TODO(#482): remove after migration` is useful.

---

## Quick Checklist Before Committing Comments

- [ ] Every comment explains *why*, not *what*
- [ ] No comment narrates an edit or fix just made
- [ ] No commented-out code
- [ ] Jargon is either explained inline or removed
- [ ] Each comment would make sense to someone who just joined the project
- [ ] No comment exceeds 2 lines unless it documents a genuinely complex decision or tradeoff
