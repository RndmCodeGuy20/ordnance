---
name: pr-create
description: >-
  TRIGGER when: user asks to create, open, make, submit, or update a PR/pull request.
  Pushes the current branch, creates submodule PRs if needed, and creates the main PR
  with a generated description.
---

Create a pull request for the current branch. Pushes, handles submodule PRs if needed, and creates the main PR with a generated description.

User input: $ARGUMENTS

Parse the input:
- If empty: auto-detect the base branch.
- Otherwise: use the entire input as the **base branch** name.

---

## Phase 1 — Analysis

### Step 1: Branch and base detection

```bash
git rev-parse --abbrev-ref HEAD
git ls-remote --heads origin main develop 2>/dev/null
```

Base branch detection (if not provided via `$ARGUMENTS`):
1. If `origin/main` exists → use `main`
2. Else if `origin/develop` exists → use `develop`
3. Else ask the user: "What should the base branch be? (e.g. main, develop, release/x.y)"

### Step 2: Gather commits and diff

```bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
BASE="<detected base>"
echo "=== COMMITS ===" && git log --oneline --no-merges "origin/${BASE}..HEAD"
echo "=== DIFF STAT ===" && git diff --stat "origin/${BASE}..HEAD"
```

If there are **no commits** ahead of the base branch, say so and **stop**.

### Step 3: Check for existing PRs

```bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
gh pr list --head "$BRANCH" --json url,title,state,number 2>/dev/null
```

### Step 4: Generate PR content

**PR Title:**
- Format: `Feature:`, `Fix:`, `Chore:`, `Refactor:`, `Docs:`, `Test:`, `Perf:`, `CI:` + running sentence case
- Under 72 chars total
- Branch name hints: `feature/` → Feature, `fix/` → Fix, `chore/` → Chore, etc.

**Writing style:**
- "What this does" = elevator pitch (1-2 sentences, high-level why)
- "Changes" = concrete what changed — no overlap with summary
- No file paths or function names unless they ARE the change
- Change bullets: past tense, one line each, max 5 (PR descriptions describe what was done; commit messages describe what the commit does — past tense is correct here)
- Test steps: action-first, short

**Issue linking:**
- `Closes #123` if all issue acceptance criteria pass after this PR merges; `Relates to: #123` if any criterion remains open
- Sub-issues: list numbers only — GitHub auto-renders titles

**Conditional sections — include only when applicable:**
- **Architecture**: new entities, permission models, complex flows. Use mermaid `erDiagram`/`sequenceDiagram`. Skip for bug fixes.
- **API Reference**: adds or modifies HTTP endpoints. Use a table with Method, Route, Auth, Request, Response. Skip for internal-only changes.
- **Screenshots**: only if a dev server is running. To check: `curl -s -o /dev/null -w '%{http_code}' http://localhost:<port>` — if not 200/301, omit entirely. Never add an empty Screenshots heading.

**PR body template:**
```
## 📝 What this does
<1-2 sentence elevator pitch>

## 📎 References
<omit if no PRD or design links>

<Closes #issue OR Relates to: #issue — omit if none>

## 🏗️ Architecture
<omit if no new entities or flows>

## 🔌 API Reference
<omit if no endpoint changes>
| Method | Route | Auth | Request | Response |
|--------|-------|------|---------|----------|

## 🔀 Changes
- <past tense, max 5 bullets>

## 📸 Screenshots
<omit if no UI changes or no dev server>

## 🧪 How to test
- [ ] <short action-first step>
```

---

## Phase 2 — Create PRs

### Step 1: Push

```bash
git push -u origin <branch>
```

If `git push` exits non-zero, abort and surface the error. Do not create the PR with an unpushed branch.

For monorepos with submodules, push submodules first:
```bash
git submodule foreach 'git push -u origin $(git rev-parse --abbrev-ref HEAD) 2>/dev/null || true'
git push -u origin <branch>
```

### Step 2: Create submodule PRs (if applicable)

For each submodule with commits ahead of base:

```bash
cd <submodule> && gh pr create --base <base> --title "<TITLE>" --body "$(cat <<'PREOF'
## 📝 What this does
<1-2 sentence summary>
- [main repo](<PR url or PENDING>)

## 🔀 Changes
- <what changed, past tense>
PREOF
)" && cd -
```

Before creating a submodule PR, check if one already exists: `gh pr list --head <branch> --json url,number` in the submodule directory. If a PR exists, use `gh pr edit` instead of creating. Capture all submodule PR URLs.

### Step 3: Create or update the main PR

**If no existing PR:**
```bash
gh pr create --base <base> --title "<TITLE>" --body "$(cat <<'PREOF'
<MAIN_BODY>
PREOF
)"
```

**If existing PR:**
```bash
gh pr edit <number> --title "<TITLE>" --body "$(cat <<'PREOF'
<MAIN_BODY>
PREOF
)"
```

### Step 4: Output

```
PR created: <url>
Submodule PRs: <urls or "none">
```

---

## Rules

1. Always use heredoc format (`cat <<'PREOF'`) for PR bodies.
2. If no commits ahead of base, say so and stop.
3. Do NOT ask for review before creating — just create. User can re-run to update.
4. Do NOT create duplicate PRs — check first, use `gh pr edit` to update.

## Composability

- **Often preceded by:** `commit` to commit changes
