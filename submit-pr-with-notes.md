# Submit PR with Code Notes

## Purpose

Commit all current changes to a feature branch, push to GitHub, and open a pull request with a well-structured description including summary, file-level change notes, and a test plan.

## Prompt

```
Commit and submit a PR for the current changes on this branch.

- Write a clear, concise PR title (under 70 characters).
- In the PR body, include:
  1. **Summary** — A short description of what this PR does and why.
  2. **Changes** — A file-by-file breakdown noting what changed and why.
  3. **Notes for reviewers** — Anything the reviewer should pay attention to: trade-offs made, areas of uncertainty, or decisions that need validation.
  4. **Test plan** — A checklist of how to verify the changes work correctly.
- Push the branch and open the PR against `main` (or `<BASE_BRANCH>` if specified).
```

## Parameters

| Parameter      | Description                                         | Default  |
| -------------- | --------------------------------------------------- | -------- |
| `BASE_BRANCH`  | Branch to merge into                                | `main`   |

## Expected Outcome

1. All changes committed to the current feature branch with a descriptive commit message.
2. Branch pushed to the remote.
3. A GitHub PR opened with a structured description covering summary, per-file changes, reviewer notes, and test plan.
