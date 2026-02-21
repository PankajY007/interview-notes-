# 10 — Git & Version Control (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: Git Basics, Branching, Merge vs Rebase, Cherry-pick, Stash, Workflows, Conflict Resolution

---

## 📌 Table of Contents

1. [Git Fundamentals](#1-git-fundamentals)
2. [Branching Strategies](#2-branching-strategies)
3. [Merge vs Rebase](#3-merge-vs-rebase)
4. [Advanced Git Commands](#4-advanced-git-commands)
5. [Conflict Resolution](#5-conflict-resolution)
6. [Git Workflows](#6-git-workflows)
7. [Interview Questions & Answers](#7-interview-questions--answers)

---

## 1. Git Fundamentals

### Git Areas
```
Working Directory → (git add) → Staging Area → (git commit) → Local Repo → (git push) → Remote Repo
```

### Essential Commands
```bash
# Setup
git init                          # initialize new repo
git clone <url>                   # clone remote repo
git config --global user.name "Name"

# Daily workflow
git status                        # see changed files
git add .                         # stage all changes
git add file.txt                  # stage specific file
git commit -m "message"           # commit staged changes
git push origin main              # push to remote
git pull origin main              # fetch + merge from remote

# Viewing history
git log --oneline --graph --all   # visual commit history
git log --author="John"           # filter by author
git log --since="2 weeks ago"     # filter by date
git diff                          # unstaged changes
git diff --staged                 # staged changes
git diff branch1..branch2         # between branches
git show <commit-hash>            # show specific commit
git blame file.txt                # who changed each line

# Undo
git checkout -- file.txt          # discard unstaged changes (old)
git restore file.txt              # discard unstaged changes (new)
git restore --staged file.txt     # unstage file
git reset HEAD~1                  # undo last commit (keep changes)
git reset --hard HEAD~1           # undo last commit (DISCARD changes)
git revert <commit-hash>          # create new commit that undoes a specific commit
```

### git reset vs git revert
| Command | Effect | History | Safe for shared branches? |
|---------|--------|---------|---------------------------|
| `reset --soft` | Undo commit, keep changes staged | Rewrites | ❌ No |
| `reset --mixed` | Undo commit, keep changes unstaged | Rewrites | ❌ No |
| `reset --hard` | Undo commit, discard changes | Rewrites | ❌ No |
| `revert` | New commit that undoes changes | Preserves | ✅ Yes |

---

## 2. Branching Strategies

```bash
# Create and switch
git branch feature-login          # create branch
git checkout feature-login        # switch to it
git checkout -b feature-login     # create + switch (shorthand)
git switch -c feature-login       # modern equivalent

# Delete
git branch -d feature-login      # delete merged branch
git branch -D feature-login      # force delete unmerged branch
git push origin --delete feature  # delete remote branch

# List
git branch                        # local branches
git branch -r                     # remote branches
git branch -a                     # all branches
```

### Branch Naming Conventions
```
feature/user-authentication
bugfix/login-error
hotfix/payment-crash
release/v2.1.0
chore/update-dependencies
refactor/order-service
```

---

## 3. Merge vs Rebase

### Git Merge — preserves history
```bash
git checkout main
git merge feature-login
# Creates a merge commit that combines both branches
```
```
Before:     main: A—B—C
            feature:  B—D—E

After merge: main: A—B—C———F (merge commit)
                      \   /
                       D—E
```

### Git Rebase — linear history
```bash
git checkout feature-login
git rebase main
# Replays feature commits on top of main
```
```
Before:     main: A—B—C
            feature:  B—D—E

After rebase: main: A—B—C
              feature:      C—D'—E' (new commits, same changes)
```

### When to Use What

| Scenario | Use |
|----------|-----|
| Merging feature into main | **Merge** (preserves context) |
| Updating feature with latest main | **Rebase** (clean history) |
| Shared/public branch | **Merge** (never rewrite shared history) |
| Personal/local branch | **Rebase** (clean up before PR) |
| After code review | **Merge** (creates clear merge point) |

### ⚠️ Golden Rule
> **Never rebase commits that have been pushed to a shared branch.**  
> Rebasing rewrites commit hashes, causing problems for collaborators.

### Interactive Rebase — Clean up commits before PR
```bash
git rebase -i HEAD~3  # interactive rebase last 3 commits

# In the editor:
pick abc1234 Add login form
squash def5678 Fix typo in login    # combine with previous
squash ghi9012 Add validation       # combine with previous
# Result: 3 commits → 1 clean commit
```

---

## 4. Advanced Git Commands

### Cherry-pick — apply specific commit
```bash
git cherry-pick <commit-hash>     # apply one commit
git cherry-pick abc123 def456     # apply multiple commits
git cherry-pick abc123..def456    # apply a range
# Use case: hotfix a specific commit from develop to main
```

### Stash — temporarily save work
```bash
git stash                         # stash changes
git stash save "WIP: login page"  # with message
git stash list                    # see all stashes
git stash pop                     # apply and remove latest
git stash apply stash@{2}        # apply specific stash (keep in list)
git stash drop stash@{0}         # remove a stash
git stash clear                   # remove all stashes
# Use case: switching branches with uncommitted work
```

### Reflog — recover lost commits
```bash
git reflog                        # history of HEAD movements
git checkout <reflog-hash>        # recover deleted branch/commit
# Use case: accidentally ran git reset --hard
```

### Bisect — find bug-introducing commit
```bash
git bisect start
git bisect bad                    # current commit is broken
git bisect good abc123            # this old commit was working
# Git checks out middle commit, you test, mark good/bad
# Binary search finds the exact commit that broke things
git bisect reset                  # exit bisect
```

### Tags
```bash
git tag v1.0.0                    # lightweight tag
git tag -a v1.0.0 -m "Release"   # annotated tag (recommended)
git push origin v1.0.0            # push specific tag
git push origin --tags            # push all tags
```

---

## 5. Conflict Resolution

```bash
# When merge/rebase hits a conflict:
git status  # shows conflicted files

# Conflicted file contents:
<<<<<<< HEAD (your branch)
    String greeting = "Hello World";
=======
    String greeting = "Hi World";
>>>>>>> feature-branch (incoming)

# Steps:
# 1. Edit file to resolve (keep one, combine, or rewrite)
# 2. git add resolved-file.txt
# 3. git merge --continue   (or git rebase --continue)

# Abort if needed:
git merge --abort
git rebase --abort
```

### Tips for Avoiding Conflicts
1. Pull/rebase frequently from main
2. Keep branches short-lived
3. Communicate with team about shared files
4. Use feature flags for parallel development

---

## 6. Git Workflows

### GitFlow
```
main ──────────────────────────────────── (production)
  │
  └─ develop ─────────────────────────── (integration)
       │        │
       └─ feature/login ──┘ (merge to develop)
       └─ feature/payment ─┘
       │
       └─ release/v1.0 ──── (testing) ──→ merge to main + develop
       │
       └─ hotfix/critical ──→ merge to main + develop
```

### Trunk-Based Development (Modern, CI/CD friendly)
```
main ──────────────────── (always deployable)
  ├── short-lived-branch-1 ─┤ (merge in 1-2 days)
  ├── short-lived-branch-2 ─┤
  └── feature-flags for long-running features
```

### GitHub Flow (Simple)
```
1. Create branch from main
2. Commit changes
3. Open Pull Request
4. Code review
5. Merge to main
6. Deploy
```

---

## 7. Interview Questions & Answers

### Q1: What is the difference between `git merge` and `git rebase`?
**A:** Merge creates a merge commit preserving branch history (non-linear). Rebase replays commits on top of another branch (linear history). Use merge for shared branches, rebase for cleaning up local/personal branches before PR.

### Q2: What is `git stash` and when would you use it?
**A:** Stash temporarily saves uncommitted changes without committing. Use when you need to switch branches but have work-in-progress that's not ready to commit. `git stash pop` restores the changes.

### Q3: How do you undo the last commit?
**A:** `git reset --soft HEAD~1` (keep changes staged), `git reset HEAD~1` (keep changes unstaged), `git reset --hard HEAD~1` (discard changes). For pushed commits, use `git revert HEAD` to create a new commit that undoes changes.

### Q4: What is `git cherry-pick`?
**A:** Applies a specific commit from one branch to another without merging the entire branch. Use case: applying a bug fix from develop to a release branch without bringing all develop changes.

### Q5: What is the difference between `git fetch` and `git pull`?
**A:** `git fetch` downloads remote changes but doesn't integrate them. `git pull` = `git fetch` + `git merge`. Use `fetch` when you want to review changes before integrating.

### Q6: How do you resolve merge conflicts?
**A:** Open conflicted files, look for `<<<<<<<`, `=======`, `>>>>>>>` markers, manually resolve by choosing/combining changes, `git add` the resolved files, then `git merge --continue` or `git commit`.

### Q7: What is `git rebase -i` used for?
**A:** Interactive rebase allows you to squash, reorder, edit, or drop commits. Use before creating a PR to clean up messy commit history into logical, well-described commits.

---

> **Next:** [11_React_Basics.md](11_React_Basics.md) →
