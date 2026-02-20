# GitHub PR Code Review

## Purpose

Perform a quick code review analysis on a GitHub pull request, summarizing changes, flagging issues, and providing actionable feedback.

## Prompt

```
Review this GitHub PR: <PR_URL>

Provide a code review covering:

1. **Summary** — What does this PR do? Summarize the intent and scope of the changes.
2. **File-by-file breakdown** — For each changed file, briefly describe what changed and why it matters.
3. **Issues & risks** — Flag any bugs, security concerns, performance problems, or logic errors.
4. **Style & maintainability** — Note any readability, naming, or structural concerns.
5. **Suggestions** — Provide specific, actionable improvements with code examples where helpful.
6. **Verdict** — One of: Approve, Request Changes, or Comment. Include a short rationale.
```

## Parameters

| Parameter | Description                              | Example                                        |
| --------- | ---------------------------------------- | ---------------------------------------------- |
| `PR_URL`  | Full URL to the GitHub pull request      | `https://github.com/username/repo/pull/42`     |

## Expected Outcome

A structured code review with clear feedback organized by the sections above, ready to act on or paste into the PR as a review comment.
