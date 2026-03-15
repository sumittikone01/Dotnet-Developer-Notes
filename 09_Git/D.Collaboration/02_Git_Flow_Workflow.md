
# 02 — Git Flow Workflow

---

## 🎯 One-Line Definition

> **Git Flow is a branching strategy that uses dedicated branches for features, releases, and hotfixes — keeping `main` always production-ready and `develop` as the integration point for ongoing work.**

---

## 🔑 The Branch Structure

```
┌─────────────────────────────────────────────────────────────────┐
│  main        → PRODUCTION code. Every commit here is a release. │
│                Tagged with version numbers (v1.0, v1.1, v2.0)  │
│                                                                  │
│  develop     → Integration branch. All features merge here.     │
│                Represents "next release in progress"            │
│                                                                  │
│  feature/*   → One branch per feature. Branches off develop.    │
│                Merges back into develop when done.              │
│                                                                  │
│  release/*   → Prep for a new release. Branch off develop.      │
│                Only bug fixes here (no new features).           │
│                Merges into main AND develop when ready.         │
│                                                                  │
│  hotfix/*    → Urgent production fix. Branches off main.        │
│                Merges into main AND develop when done.          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 The Full Flow — Visualised

```
main ──────────────────────────────●─────────────────────●────────
                                   │ v1.0                │ v1.1
hotfix ──────────────────────────● │                     │
                                 │ │                     │
develop ──────────────●──────────●─●──────────●──────────●────────
                      │          │             │          │
feature/export ─────●─┘          │             │          │
                                 │             │          │
feature/report ──────────────●───┘         ●──┘          │
                                           │              │
release/1.1 ──────────────────────────────────────────●──┘
```

---

## 🔑 Feature Branch Workflow

```bash
# 1. Start from develop
git checkout develop
git pull

# 2. Create feature branch
git checkout -b feature/employee-excel-export

# 3. Work and commit
git add . && git commit -m "Add export button to grid toolbar"
git add . && git commit -m "Add controller action for Excel download"

# 4. Merge back into develop (via PR or directly)
git checkout develop
git merge --no-ff feature/employee-excel-export
git push origin develop

# 5. Delete feature branch
git branch -d feature/employee-excel-export
git push origin --delete feature/employee-excel-export
```

---

## 🔑 Release Branch Workflow

```bash
# When develop is ready for a release:

# 1. Create release branch from develop
git checkout develop
git checkout -b release/1.2.0

# 2. Only bug fixes here — no new features
git add . && git commit -m "Fix date format in export"
git add . && git commit -m "Fix salary rounding in report"

# 3. When ready: merge into main AND develop
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"   ← tag the release
git push origin main --tags

git checkout develop
git merge --no-ff release/1.2.0                 ← bring fixes to develop too
git push origin develop

# 4. Delete release branch
git branch -d release/1.2.0
```

---

## 🔑 Hotfix Branch Workflow

```bash
# Production has a critical bug — fix it immediately:

# 1. Branch off MAIN (not develop)
git checkout main
git checkout -b hotfix/fix-login-crash

# 2. Fix the bug
git add . && git commit -m "Fix null reference in login controller"

# 3. Merge into main AND develop
git checkout main
git merge --no-ff hotfix/fix-login-crash
git tag -a v1.1.1 -m "Hotfix: fix login crash"
git push origin main --tags

git checkout develop
git merge --no-ff hotfix/fix-login-crash        ← fix goes to develop too
git push origin develop

# 4. Delete hotfix branch
git branch -d hotfix/fix-login-crash
```

---

## 🔑 Git Flow vs Simpler Workflows

```
GIT FLOW — Good for:
  ✅ Scheduled releases (not continuous deployment)
  ✅ Multiple versions maintained simultaneously
  ✅ Large teams with clear release cycles
  ✅ Enterprise software

GIT FLOW — Overkill for:
  ❌ Small teams (1-3 devs)
  ❌ Continuous deployment (deploy on every merge)
  ❌ Simple apps with one production version

SIMPLER ALTERNATIVE — GitHub Flow:
  Just: main + feature branches
  Feature → PR → review → merge to main → deploy
  Good for continuous deployment and small teams
```

---

## ❓ Interview Questions

**Q: What is Git Flow?**

> A branching strategy with five branch types: `main` (production), `develop` (integration), `feature/*` (new features), `release/*` (release preparation), `hotfix/*` (urgent production fixes). Features merge into develop, releases merge into both main and develop, hotfixes merge into both main and develop.

**Q: What is the difference between a feature branch and a hotfix branch?**

> Feature branches branch off `develop` and merge back to `develop` — they're for planned new functionality. Hotfix branches branch off `main` and merge into both `main` AND `develop` — they're for urgent production fixes that can't wait for the next planned release.

**Q: Why do hotfixes merge into both `main` and `develop`?**

> Merging into `main` fixes the production bug and creates a new release tag. Merging into `develop` ensures the fix isn't lost when the next planned release is prepared — without this, the bug would reappear in the next release.
>
