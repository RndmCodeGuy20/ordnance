---
name: commit
description: >-
  Smart commits with auto-generated messages. Detects dirty repos, generates
  conventional commit messages, handles monorepos with submodules.
  Often followed by pr-create.
---

Create a commit (or commits) for the current repo. Detects staged vs unstaged changes, generates a message from the diff, and handles submodule pointer updates in monorepos.

User input: $ARGUMENTS

**Usage:**
```
/commit                      # auto-generate commit message from diff
/commit fix null pointer      # use this message as-is
```

Parse the input:
- If empty: auto-generate a commit message per dirty repo based on its diff.
- If non-empty: use the entire input as the commit message.

---

## Phase 1 — Detect dirty repos

```bash
ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
if [ -z "$ROOT" ]; then echo "Error: not inside a git repository." && exit 1; fi
echo "ROOT=$ROOT"

# Detect submodules
git submodule status 2>/dev/null | while read sha path rest; do
  status=$(git -C "$ROOT/$path" status --porcelain 2>/dev/null)
  staged=$(git -C "$ROOT/$path" diff --cached --stat 2>/dev/null)
  echo "SUB_PATH=$path"
  echo "SUB_DIRTY=$([[ -n "$status" ]] && echo YES || echo NO)"
  echo "SUB_STAGED=$([[ -n "$staged" ]] && echo YES || echo NO)"
done

root_status=$(git -C "$ROOT" status --porcelain 2>/dev/null)
root_staged=$(git -C "$ROOT" diff --cached --stat 2>/dev/null)
echo "ROOT_DIRTY=$([[ -n "$root_status" ]] && echo YES || echo NO)"
echo "ROOT_STAGED=$([[ -n "$root_staged" ]] && echo YES || echo NO)"
```

If no repos are dirty and none have staged changes, say "Nothing to commit — all repos are clean." and stop.

---

## Phase 2 — Commit each dirty repo

Process submodules first, then root (so root can capture updated submodule pointers).

For each dirty repo:

### Step 1: Check staging state

```bash
cd <path>
echo "=== STAGED ===" && git diff --cached --stat
echo "=== UNSTAGED ===" && git diff --stat
echo "=== UNTRACKED ===" && git ls-files --others --exclude-standard
```

### Step 2: Stage if needed

- If already staged: commit only those — do NOT add more. Note which unstaged files are excluded in the summary.
- If nothing staged: stage everything. For the root repo, exclude submodule entries:
  ```bash
  # submodule
  cd <path> && git add -A

  # root only — exclude submodule dirs
  SUBMODS=$(git submodule status 2>/dev/null | awk '{print ":!" $2}' | tr '\n' ' ')
  if [ -z "$SUBMODS" ]; then
    cd <root> && git add -A
  else
    cd <root> && git add -A -- $SUBMODS
  fi
  ```

### Step 3: Get diff for message generation

```bash
cd <path> && git diff --cached   # used for auto-generated message below
```

### Step 4: Commit

**If user provided a message:**
```bash
cd <path> && git commit -m "<user message>"
```

**If auto-generating**, analyze the cached diff and generate a message:

**Subject line:**
- Format: `<type>: <summary>` — type is one of `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `perf`, `ci`
- Imperative mood: "Add feature" not "Added" or "Adds"
- Under 72 chars, no trailing period
- Describe *what* the commit does, not *how*

**Body (only when the subject line cannot fully capture the change — i.e. more than one logical sub-change, or any change that affects a public API or schema):**
- One blank line after subject, then a bullet list
- Each bullet: imperative verb + one specific change (Add, Fix, Remove, Update, Extract, etc.)
- One line per bullet, no trailing periods, no filler words

**Never include:**
- `Co-Authored-By:` or any AI attribution trailer
- File lists or function names unless the rename/move IS the change
- Redundant context already obvious from the diff

**Example (good):**
```
fix: guard null check before dereferencing config

- Return early when config object is absent on startup
- Add missing default fallback in parseOptions
- Cover both code paths in unit tests
```

**Example (bad):**
```
Updated some stuff and fixed a few things

- I made some changes to the config file because it was crashing
- Also fixed the thing
- Co-authored-by: Claude <noreply@anthropic.com>
```

```bash
cd <path> && git commit -m "$(cat <<'EOF'
<generated message>
EOF
)"
```

If `git commit` exits non-zero (e.g. pre-commit hook rejection), surface the error verbatim and stop. Do not retry or modify the message.

---

## Phase 3 — Update submodule pointers (monorepos only)

Only if any submodule was committed in Phase 2:

```bash
cd <root> && git add $(git submodule status | awk '{print $2}') && git diff --cached --stat
```

If there are staged submodule pointer changes, commit:
```bash
cd <root> && git commit -m "chore: update submodule pointers"
```

---

## Phase 4 — Summary

```
## Commit Summary

| Repo | Status | Commit |
|---|---|---|
| <submodule> | ✓ committed / — clean | <hash> <subject> |
| root | ✓ committed / — clean | <hash> <subject> |
| root (pointers) | ✓ updated / — no changes | <hash if committed> |
```

---

## Rules

1. **Submodules before root** — so root captures updated pointers.
2. **Respect existing staging** — if files are already staged, commit only those. Note excluded unstaged files in the summary.
3. **Never** force-push, reset --hard, or use --no-verify.
4. **No `Co-Authored-By` trailers** — ever. One imperative subject line; body only when subject alone is insufficient.
5. **Amend vs new commit** — amend only if the new change corrects or completes what the preceding commit intended AND that commit hasn't been pushed. New commit for anything conceptually distinct.
6. **Never stage local artefact directories** (plans, scratch files) from the root repo.

## Composability

- **Often followed by:** `pr-create` to push and open a pull request
