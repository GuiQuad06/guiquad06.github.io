---
layout: default
title: Git Merge Strategies Overview
---

# Git Merge Strategies on Hosting Platforms

Overview of the pull/merge request integration options offered by GitHub, GitLab,
Azure DevOps, Gitea and Bitbucket, with history diagrams and recommendations for very
small teams (1-3 developers).

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({ startOnLoad: true });</script>

- [1. Core strategies](#1-core-strategies)
- [2. Comparison](#2-comparison)
- [3. Platform availability](#3-platform-availability)
- [4. Adjacent options often bundled in the UI](#4-adjacent-options-often-bundled-in-the-ui)
- [5. General guidance](#5-general-guidance)
- [6. Advice for a small team (1-3 developers)](#6-advice-for-a-small-team-1-3-developers)

---

## 1. Core strategies

### 1.1 Merge commit (no fast-forward, "true merge")

Creates a new commit with two parents. Full history preserved, branch topology stays
visible.

- Git equivalent: `git merge --no-ff feature`
- Platform names: GitHub "Create a merge commit", GitLab "Merge commit", Azure DevOps
  "Merge (no fast forward)", Gitea "Create a merge commit", Bitbucket "Merge commit"

<pre class="mermaid">
gitGraph
   commit id: "A"
   commit id: "B"
   branch feature
   commit id: "F1"
   commit id: "F2"
   checkout main
   commit id: "C"
   merge feature id: "M"
   commit id: "D"
</pre>

### 1.2 Fast-forward

Only moves the target branch pointer. No merge commit at all. Possible only when the
source branch is a direct descendant of the target.

- Git equivalent: `git merge --ff-only feature`
- Platform names: GitLab "Fast-forward merge", Gitea "Fast-forward only", Bitbucket
  "Fast forward"
- Not exposed by GitHub or Azure DevOps

<pre class="mermaid">
gitGraph
   commit id: "A"
   commit id: "B"
   commit id: "F1"
   commit id: "F2"
</pre>

Before the merge the graph looks like a branch; after the fast-forward the commits
simply become part of `main` with no trace of the branch (unless the branch ref is
kept).

### 1.3 Squash

All commits of the branch collapse into one single commit on the target. Linear
history, intermediate commits are lost.

- Git equivalent: `git merge --squash feature && git commit`
- Platform names: GitHub "Squash and merge", GitLab "Squash commits", Azure DevOps
  "Squash commit", Gitea "Squash", Bitbucket "Squash"
- Variants: GitLab and Gitea can squash **and** still create a merge commit

<pre class="mermaid">
gitGraph
   commit id: "A"
   commit id: "B"
   commit id: "S (F1+F2)"
   commit id: "C"
</pre>

### 1.4 Rebase (linear, no merge commit)

Replays each source commit on top of the target tip. Individual commits are preserved
but their SHAs are rewritten. No merge commit.

- Git equivalent: `git rebase main feature && git merge --ff-only feature`
- Platform names: GitHub "Rebase and merge", GitLab "Fast-forward with rebase", Azure
  DevOps "Rebase and fast-forward", Gitea "Rebase then fast-forward", Bitbucket
  "Rebase, fast-forward"

<pre class="mermaid">
gitGraph
   commit id: "A"
   commit id: "B"
   commit id: "C"
   commit id: "F1'"
   commit id: "F2'"
</pre>

### 1.5 Semi-linear (rebase + merge commit)

Rebase the source commits onto the target, **then** create a merge commit anyway.
Result: linear commit content plus an explicit merge point marking the PR boundary.

- Git equivalent: `git rebase main feature && git checkout main && git merge --no-ff feature`
- Platform names: Azure DevOps "Rebase with merge commit" (semi-linear), Gitea "Rebase
  then create merge commit", Bitbucket "Rebase, merge commit"
- Not offered natively by GitHub or GitLab

<pre class="mermaid">
gitGraph
   commit id: "A"
   commit id: "B"
   commit id: "C"
   branch feature
   commit id: "F1'"
   commit id: "F2'"
   checkout main
   merge feature id: "M"
</pre>

The rebased commits sit directly on top of `C`, and the merge commit `M` records that
they arrived together as one PR.

---

## 2. Comparison

| Strategy | Merge commit? | SHAs rewritten? | History shape | Individual commits kept | Revert whole PR easily |
|---|---|---|---|---|---|
| Merge commit | Yes | No | Branchy | Yes | Yes (`revert -m 1`) |
| Fast-forward | No | No | Linear | Yes | No (commit by commit) |
| Squash | No (usually) | Yes (collapsed) | Linear | No | Yes (single revert) |
| Rebase | No | Yes | Linear | Yes | No (commit by commit) |
| Semi-linear | Yes | Yes | Linear + merge points | Yes | Yes (`revert -m 1`) |

---

## 3. Platform availability

| Option | GitHub | GitLab | Azure DevOps | Gitea | Bitbucket DC |
|---|---|---|---|---|---|
| Merge commit | Yes | Yes | Yes | Yes | Yes |
| Fast-forward only | No | Yes | No | Yes | Yes |
| Squash | Yes | Yes | Yes | Yes | Yes |
| Rebase (linear) | Yes | Yes | Yes | Yes | Yes |
| Semi-linear | No | No | Yes | Yes | Yes |
| Manual / no auto-merge | No | No | No | Yes | No |

---

## 4. Adjacent options often bundled in the UI

- **Merge queue / merge trains** (GitHub merge queue, GitLab merge trains): serialize
  merges and re-test each PR against the updated target before landing, avoiding
  semantic conflicts.
- **Auto-merge / merge when pipeline succeeds**: a trigger, not a strategy.
- **Fast-forward-if-possible-else-merge**: plain `git merge` default behaviour;
  sometimes shown as "merge if necessary".
- **Commit message templating**: squash and merge-commit modes usually let you
  configure the resulting message (PR title, description, commit list).
- **Conflict resolution strategies** (`-s ort`, `-X ours`, `-X theirs`): git-level
  options, rarely exposed in web UIs. Do not confuse them with the PR-level options
  above.

---

## 5. General guidance

- Clean bisectable linear history with short-lived branches → **squash**
- Preserve meaningful commit-by-commit work → **rebase** or **semi-linear**
- Auditability of "what came in as one unit" while staying linear → **semi-linear**
- Long-lived release branches, or exact SHAs must be preserved (signed commits,
  published refs) → **merge commit**; rebase and squash rewrite history and invalidate
  signatures

---

## 6. Advice for a small team (1-3 developers)

With 1-3 developers, the dominant cost is not merge conflicts, it is **being able to
answer "what changed between release X and Y?" and "how do I undo this?"**. Optimise
for that.

### 6.1 Frequent releases (continuous delivery, trunk-based)

Context: you ship several times a week or per day, everything lives on `main`, feature
branches last hours to a couple of days, no maintenance of old versions.

**Recommendation: squash merge onto a single `main`, tag every release.**

- One commit per feature or fix on `main`. The `main` log becomes a readable changelog.
- `git bisect` is very effective: each commit is one self-contained change that built
  and passed CI.
- Reverting a bad feature is a single `git revert`, which matters when you ship fast.
- No release branches at all; a release is just an annotated tag on `main`
  (`v1.4.7`). If a hotfix is needed, it goes to `main` and you ship again immediately.
- Enable "delete source branch after merge" and "require pipeline to pass". Skip merge
  queues, at this team size they only add latency.
- Use feature flags rather than long-lived branches to keep unfinished work off the
  critical path.

<pre class="mermaid">
gitGraph
   commit id: "init"
   commit id: "feat: audio path" tag: "v1.0.0"
   commit id: "fix: gain clamp"
   commit id: "feat: telemetry" tag: "v1.1.0"
   commit id: "fix: dma race" tag: "v1.1.1"
</pre>

**Avoid:** merge commits. With daily merges the graph becomes a braid of noise for zero
benefit, since nobody ever needs to inspect a two-day-old branch topology.

### 6.2 Occasional releases (a few per year, versions to maintain)

Context: firmware or product releases every few months, you must patch a version that
is already in the field while `main` has moved on.

**Recommendation: `main` + release branches; semi-linear (or merge commit) into
release branches, squash into `main`.**

- Keep squash merges for normal feature work into `main`: same changelog benefit as
  above.
- Cut a `release/1.4` branch at feature freeze. Only fixes go there.
- For fixes, prefer **fix on `main` first, then cherry-pick to the release branch**
  (forward-fix). This prevents the classic small-team failure mode where a field fix
  exists only on the release branch and silently regresses in the next version.
- If a fix must be authored on the release branch (urgent field issue), merge it back
  to `main` with a real **merge commit** so git records the ancestry and future merges
  do not replay it. Never rebase a branch that another branch was cut from or that
  someone else has pulled.
- Tag every shipped build, including release candidates (`v1.4.0-rc1`), and never move
  a tag.
- If your platform offers it, **semi-linear** is a good default for merging feature
  branches into a long-lived release branch when you want the individual commits kept
  but the history readable.

<pre class="mermaid">
gitGraph
   commit id: "A"
   commit id: "feat: X"
   branch release/1.4
   commit id: "rc prep" tag: "v1.4.0"
   checkout main
   commit id: "feat: Y"
   checkout release/1.4
   commit id: "fix: field bug" tag: "v1.4.1"
   checkout main
   merge release/1.4 id: "back-merge"
   commit id: "feat: Z" tag: "v1.5.0"
</pre>

**Avoid:** cherry-picking in both directions without tracking it, and rebasing release
branches. Both produce duplicated commits with different SHAs and make "is this fix in
that build?" unanswerable.

### 6.3 Practices worth more than the merge strategy itself

At this team size these matter more than which button you click:

1. **Protect `main`** (and release branches): no force-push, no direct push, PR
   required even for solo work. A PR is your review record and your CI gate.
2. **Self-review your own PR** when you are alone. Reading your own diff in the web UI
   catches a surprising amount.
3. **Conventional commit subjects** (`feat:`, `fix:`, `refactor:`). Combined with squash
   merges you get a generated changelog for free.
4. **One PR = one intent.** This is what makes squash merges readable and reverts safe.
5. **Delete branches after merge**, and enforce "branch must be up to date with target"
   so CI tests what actually lands.
6. **Tag releases immutably** and record the exact commit SHA in the build artefacts,
   especially for firmware where the binary outlives the repo state in someone's
   memory.

### 6.4 Quick decision table

| Situation | Strategy |
|---|---|
| Short-lived feature branch → `main`, frequent releases | Squash |
| Short-lived feature branch → `main`, occasional releases | Squash |
| Long-lived feature branch you want to keep commit-by-commit | Semi-linear (or rebase if unavailable) |
| Release branch → `main` (back-merge) | Merge commit, never rebase |
| `main` → release branch (before freeze) | Merge commit |
| Single-commit trivial fix, branch already up to date | Fast-forward or squash, identical result |
| Commits are signed and signatures must survive | Merge commit or fast-forward only |

[Back to Git Reminder](./)
