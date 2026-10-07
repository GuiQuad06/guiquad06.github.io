---
layout: default
title: Git Merge Options
---

# Git Merge Options

For a small team (1 to 3 devs):

## Frequent releases (trunk-based dev)

**Squash merge** onto a single `main` branch, and add a tag for every release:

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin --tags
```

Avoid merge commits — they only add noise pollution at this cadence.

## Occasional releases

**Semi-linear** merge: keeps the full history even when feature branches are deleted,
while still having a merge commit carrying the release tag on `main`.

[Back to Git Reminder](./)
