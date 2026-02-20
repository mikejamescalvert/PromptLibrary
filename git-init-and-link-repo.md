# Git Init and Link to GitHub

## Purpose

Initialize a local directory as a git repository, link it to an existing GitHub remote, and set up a branching workflow where all changes go through PR review.

## Prompt

```
Initialize this directory as a git repo and link it to <GITHUB_REPO_URL>.

- Create an initial commit on the `main` branch with a README and push it to the remote.
- Create a feature branch off `main` for all subsequent work.
- All changes should be committed to the feature branch, pushed, and submitted as a PR for my review on GitHub before merging to `main`.
```

## Parameters

| Parameter          | Description                                      | Example                                                    |
| ------------------ | ------------------------------------------------ | ---------------------------------------------------------- |
| `GITHUB_REPO_URL`  | HTTPS or SSH URL of the target GitHub repository | `https://github.com/username/repo.git`                     |

## Expected Outcome

1. Local git repo initialized in the working directory.
2. Remote `origin` set to the provided URL.
3. `main` branch pushed to GitHub with an initial commit.
4. A feature branch checked out and ready for work.
5. All future changes go through the PR workflow for review.
