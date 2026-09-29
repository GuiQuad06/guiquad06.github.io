---
layout: default
title: Git Cheatsheet
---

# Git Cheatsheet

## Starting work on a branch

```bash
git pull origin dev        # git fetch + git merge
git switch -c my-branch
```

`[Work…]`

```bash
git status
clang-format -i <files>
git add <files>
git commit -m "..."
```

**OR**, using fixup commits:

```bash
git commit --fixup=toto     # find the target commit with git log
git rebase -i dev --autosquash
git push origin my-branch   # create the remote branch beforehand
```

---

## Catching up with `dev`

If the local branch is behind `dev`:

```bash
git switch dev
git pull origin dev
git switch my-branch
git rebase -i dev
```

---

## Undo / cleanup

**Cancel a commit:**

```bash
git reset --soft HEAD^   # drop the commit only, keep the changes staged
git reset --hard HEAD^   # drop the commit AND the associated changes
```

**Unstage a file (remove it from the next commit):**

```bash
git restore --staged <file>
```

**Shelve a change:**

```bash
git stash
git stash pop   # bring back the last entry from the stash stack
```

> **Note:** you can chain several fixup commits and squash them all with a single
> interactive rebase before pushing.

---

## Publish a repo already managed locally with Git

```bash
git init
git add bidule
git commit -m "Creation"
git remote add origin https://github.com/GuiQuad06/sensor_over_mqtt.git   # create the repo on GH first (or with the GitHub CLI)
git push --set-upstream origin dev
```

**…or create a new repository from the command line:**

```bash
echo "# drying-oven-stm" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M dev
git remote add origin https://github.com/GuiQuad06/drying-oven-stm.git
```

---

## Apply a commit `<toto>` on another branch

```bash
git checkout <other-branch>
git cherry-pick -x <toto>   # -x keeps a reference to the original commit
```

[Back to Git Reminder](./)
