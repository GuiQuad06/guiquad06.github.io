---
layout: default
title: Git Workflows
---

# Git Workflows

- Long-lived branches (`main` / `dev`).
- Short-lived branches (features / hotfix).
- Workflow selection parameters: team size & release cadence.
- Branch out a release branch from `dev`, apply a patch if needed, and merge it into
  `main`.
- Branch rules (require a pull request, etc.).
- When merging a fix from a release branch into `main`, also merge it down into `dev`
  to stay up to date.

## Trunk-based workflow

Based on a single long-lived branch: `main`.

- Small team with a high release cadence.
- Adding some feature branches for larger teams.
- For a release:
  - Use a release branch and cherry-pick the RC commit, or…
  - …use feature flags.

[Back to Git Reminder](./)
