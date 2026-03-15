
# 02 — Git vs Centralized VCS

---

## 🎯 One-Line Definition

> **Centralized VCS (like SVN) has one server everyone depends on — Git is distributed, meaning every developer has the complete repository, so you can work fully offline and the project can never be lost because one server fails.**

---

## 🔑 The Two Models — Visual Comparison

### Centralized VCS (SVN, TFS, CVS)

```
                    ┌─────────────────┐
                    │  CENTRAL SERVER  │
                    │  (single source) │
                    │  - all history   │
                    │  - all branches  │
                    └────────┬────────┘
                             │
               ┌─────────────┼─────────────┐
               │             │             │
        ┌──────▼──┐   ┌──────▼──┐   ┌──────▼──┐
        │  Alice   │   │   Bob   │   │  Carol  │
        │ (working │   │ (working│   │ (working│
        │  copy)   │   │  copy)  │   │  copy)  │
        └──────────┘   └─────────┘   └─────────┘

  Each developer has: ONLY the latest version (working copy)
  History lives:      ONLY on the central server
  To commit:          MUST be connected to server
  Server goes down:   NOBODY can commit, branch, or see history
```

### Distributed VCS (Git)

```
        ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
        │    Alice     │      │     Bob      │      │   Carol      │
        │  FULL REPO   │      │  FULL REPO   │      │  FULL REPO   │
        │  - all history│     │  - all history│     │  - all history│
        │  - all branches│    │  - all branches│    │  - all branches│
        └──────┬───────┘      └──────┬───────┘      └──────┬───────┘
               │                     │                      │
               └─────────────────────┼──────────────────────┘
                                      │
                              ┌───────▼───────┐
                              │  REMOTE REPO  │
                              │  (GitHub etc) │
                              │  shared copy  │
                              └───────────────┘

  Each developer has: COMPLETE history + all branches
  Remote is just:     A convenient shared copy (not the only copy)
  To commit:          Work OFFLINE — no server needed
  Server goes down:   Everyone still has everything, work continues
```

---

## 🔑 Side-by-Side Comparison

| Feature                 | Centralized (SVN)                     | Distributed (Git)                    |
| ----------------------- | ------------------------------------- | ------------------------------------ |
| History stored          | Server only                           | Every developer's machine            |
| Work offline            | ❌ No — need server connection       | ✅ Yes — full functionality offline |
| Single point of failure | ✅ Yes — server down = no work       | ❌ No — every clone is a backup     |
| Commit speed            | Slow (network call to server)         | Fast (local operation)               |
| Branching               | Heavy (usually copy of entire folder) | Lightweight (just a pointer)         |
| Branch cost             | Expensive — creates a full copy      | Cheap — creates a 41-byte file      |
| Merging                 | Difficult, often error-prone          | Designed for frequent merging        |
| Access control          | Per-file / per-folder                 | Per-repository                       |
| Setup complexity        | Simpler                               | Slightly more complex                |
| Learning curve          | Lower initially                       | Higher initially                     |
| Industry use today      | Legacy only                           | Standard everywhere                  |

---

## 🔑 The Key Advantage — Offline Work

```
Scenario: You're on a train with no internet.

SVN:                                Git:
─────────────────────────────       ─────────────────────────────
Can't commit → no server            Commit freely → local
Can't see history → no server       See all history → local
Can't create branches → no server   Create branches → local
Can't diff files → no server        Diff any version → local
  ↓                                   ↓
No work gets saved until you        All commits saved locally
have internet again                 Push when back online
```

---

## 🔑 The Key Advantage — Speed

Git operations are almost all local:

```
git log        → reads local .git/  (milliseconds)
git diff       → reads local files  (milliseconds)
git commit     → writes local .git/ (milliseconds)
git branch     → reads local .git/  (milliseconds)
git checkout   → reads local .git/  (milliseconds)

Only these need network:
  git push     → sends commits to remote
  git pull     → fetches commits from remote
  git fetch    → fetches without merging
  git clone    → first-time copy from remote
```

---

## 🔑 The Key Advantage — Branching

In SVN, a branch is a copy of the entire codebase directory. Creating one means copying thousands of files.

In Git, a branch is just a 41-byte text file containing a commit hash.

```
SVN branch = copies /trunk to /branches/feature
             → thousands of files duplicated
             → takes time, uses storage

Git branch = creates .git/refs/heads/feature containing "a3f8c12"
             → one tiny file
             → instant, no storage cost
             → developers create branches for every task, every bug
```

---

## 🔑 Why Git Won — The Summary

```
Timeline:
  2005: Linus Torvalds creates Git for Linux kernel development
  2008: GitHub launches — makes sharing repositories easy
  2010: Git overtakes SVN in popularity
  2015: Git used by ~70% of developers
  2023: Git used by ~95%+ of developers

Why it won:
  ✓ Speed — every common operation is local
  ✓ Branching — lightweight, developers branch constantly
  ✓ Distributed — no single point of failure
  ✓ GitHub/GitLab/Azure DevOps built on top of it
  ✓ Pull Requests became the standard code review model
  ✓ Open source and free
```

---

## 🔑 In Your Daily ASP.NET Work

```
What this means practically for you:

  Every clone of your company's repo IS a full backup.
  If GitHub goes down, you still have everything locally.

  You can experiment freely:
    git checkout -b experiment/new-grid-approach  ← instant
    (write code, try ideas)
    git checkout main                              ← instant back
    git branch -d experiment/new-grid-approach    ← instant delete
    (no trace left, no files created/deleted)

  You commit often and push when ready:
    No need for big commits — commit every logical step
    Push at end of day or when feature is done
    Your local repo is your safety net
```

---

## ❓ Interview Questions

**Q: What is the difference between centralized and distributed version control?**

> Centralized (like SVN) stores all history on one server — developers only have a working copy and need server access to commit or see history. Distributed (like Git) gives every developer a complete copy of the entire repository with all history, allowing full offline work and no single point of failure.

**Q: Why is Git's branching faster than SVN?**

> In SVN a branch copies the entire codebase directory — expensive in time and storage. In Git a branch is just a 41-byte file pointing to a commit hash — created instantly with no storage overhead. This is why Git encourages creating branches for every feature and bug fix.

**Q: Is the remote repository (GitHub) the "authoritative" copy in Git?**

> By convention yes, but technically no. Every clone is a complete, equal copy. The remote is just a shared collaboration point agreed upon by the team. If GitHub disappeared, every developer's local clone is a full backup.
>
