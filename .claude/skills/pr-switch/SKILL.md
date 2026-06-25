---
name: pr-switch
description: >-
  Switch to a GitHub PR locally. TRIGGER when: user provides a PR link, PR number,
  or says "switch to PR", "check out PR", "review PR locally". Do NOT trigger for
  creating PRs (use pr-create instead).
---

# pr-switch: Switch to a PR Locally

Switch to a GitHub PR in the current repo. Ask whether to use a new isolated worktree or switch the current branch, then execute and sync submodules.

## Arguments
$ARGUMENTS — PR number, PR URL, or branch name. Ask if not provided.

---

## Phase 1: Gather Context

```bash
git branch --show-current
git status --porcelain
git rev-parse --show-toplevel
```

Parse $ARGUMENTS:
- Empty → ask: "What PR number, URL, or branch name do you want to switch to?"
- GitHub URL (contains `/pull/`) → extract the numeric PR ID
- Numeric → treat as PR number: `gh pr view <num> --json number,title,headRefName`
- String → treat as branch name directly. If it does not exactly match a remote branch, run `git branch -r | grep <input>` and present matches for the user to choose.

---

## Phase 2: Ask User

```
PR #<num>: "<title>" → <branch>

How do you want to work on this?
  [w] New worktree  — isolated copy, doesn't affect current branch (recommended)
  [b] Switch branch — checkout here (use this for a quick look you'll switch back from immediately)

Current branch: <branch> | Working tree: <clean|dirty>
```

Wait for the user to choose before proceeding.

---

## Phase 3a: New Worktree (if user chose w)

```bash
BRANCH=<branch>
WORKTREE_PATH=../$(basename $(git rev-parse --show-toplevel))-${BRANCH//\//-}
```

If `$WORKTREE_PATH` already exists, ask the user: "Worktree at `<path>` already exists. Reuse it or choose a different path?"

```bash
git worktree add "$WORKTREE_PATH" "$BRANCH" || \
  git worktree add "$WORKTREE_PATH" --track "origin/$BRANCH"
```

If both worktree add attempts fail, report the error and stop. Do not proceed.

```bash
cd "$WORKTREE_PATH" && git submodule update --init --recursive
```

Report the worktree path.

---

## Phase 3b: Switch Current Branch (if user chose b)

**Dirty check:** If `git status --porcelain` has output, abort:
```
Error: working tree has uncommitted changes. Commit or stash first.
<list dirty files>
```

**If clean:**
```bash
# For PR number:
gh pr checkout <num>

# For branch name:
git fetch origin
git switch <branch> 2>/dev/null || git switch --track origin/<branch>

# Always sync submodules:
git submodule update --init --recursive
```

---

## Phase 4: Summary

```
Switched to PR #<num>: "<title>"
Branch:   <branch>
Path:     <path>

Submodules synced: <submodule-path: HEAD-sha, or "none">
```

## Rules
- Never do both worktree add AND branch switch — pick one path only.
- Always run `git submodule update --init --recursive` after switching or creating a worktree.
- If the branch doesn't exist on remote, report the error and stop.
