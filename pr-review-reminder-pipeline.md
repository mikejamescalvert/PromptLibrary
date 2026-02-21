# PR Review Reminder Pipeline

## Purpose

Set up a GitHub Actions pipeline that checks a repository for open pull requests on a daily schedule and creates a GitHub Issue reminder when any exist. GitHub's built-in notifications handle email delivery — no external SMTP configuration required. Follows the ETL pattern: extract open PRs from GitHub, transform into a markdown summary, load as a GitHub Issue.

## Prompt

```
Set up a PR review reminder pipeline for this repository.

1. Ask me for:
   - **Schedule** — when should the check run? (default: weekdays at 2:00 PM UTC)
   - **Notification method** — GitHub Issue (default, no setup needed), Slack webhook, or SMTP email.

2. Create a GitHub Actions workflow (`.github/workflows/pr-reminder.yml`) that:

   **Extract:**
   - Uses `gh pr list` to fetch all open PRs with details (number, title, author, created date, URL, branch name).
   - If there are no open PRs, skip the notification and exit successfully.

   **Transform:**
   - Formats the PR data into a markdown table showing each PR's number (linked), title, author, branch, and opened date.
   - Includes the total count of open PRs in the issue title.

   **Load:**
   - Creates a GitHub Issue with the label `pr-reminder`.
   - Before creating a new issue, closes any previous open `pr-reminder` issues (so only the latest reminder is active).
   - Uses `GITHUB_TOKEN` (automatic, no secrets to configure).

3. The workflow should also support `workflow_dispatch` so it can be triggered manually for testing.

4. Ensure GitHub notification settings will deliver the reminder:
   - The repo owner automatically receives notifications for new issues on their repo.
   - Verify under GitHub Settings > Notifications that "Issues" email notifications are enabled.

5. Commit the workflow and push for review.
```

## Parameters

| Parameter        | Description                                      | Example                            |
| ---------------- | ------------------------------------------------ | ---------------------------------- |
| `SCHEDULE`       | Cron expression for when to check                | `0 14 * * 1-5` (weekdays 2pm UTC) |

## Reference

### How notifications work

No external email service or secrets are needed. The flow is:

1. Workflow runs on schedule.
2. If open PRs exist, a GitHub Issue is created with the label `pr-reminder`.
3. GitHub sends you an email notification for the new issue (built-in).
4. The next time the workflow runs, it closes the old reminder and creates a fresh one.

### GitHub notification settings

To ensure you receive email notifications for reminder issues:

1. Go to **GitHub Settings > Notifications**.
2. Under "Subscriptions > Participating and @mentions", ensure email is enabled.
3. You are automatically subscribed to issues on repos you own.

### Issue output example

**Title:** `PR Review Reminder: 2 open PR(s) — 2026-02-20`

**Body:**

| # | Title | Author | Branch | Opened |
|---|-------|--------|--------|--------|
| [#12](https://github.com/...) | Add user authentication | @dev-alice | `feat/auth` | 2026-02-18 |
| [#10](https://github.com/...) | Fix invoice calculation bug | @dev-bob | `fix/invoice-calc` | 2026-02-15 |

### Adapting to other notification channels

The default uses GitHub Issues (zero config). To use other channels instead:

**Slack (using webhook):**

```yaml
- name: Notify via Slack
  if: success()
  run: |
    curl -X POST "${{ secrets.SLACK_WEBHOOK_URL }}" \
      -H "Content-Type: application/json" \
      -d "{\"text\": \"PR Review Reminder: ${COUNT} open PR(s) in ${{ github.repository }}\"}"
```

**SMTP email (requires secrets):**

```yaml
- name: Send email
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: ${{ secrets.SMTP_SERVER }}
    server_port: ${{ secrets.SMTP_PORT }}
    username: ${{ secrets.SMTP_USERNAME }}
    password: ${{ secrets.SMTP_PASSWORD }}
    subject: "PR Review Reminder: ${COUNT} open PR(s)"
    to: ${{ secrets.NOTIFY_EMAIL }}
    from: ${{ secrets.SMTP_USERNAME }}
    html_body: file://email_body.html
```

## Expected Outcome

1. A `.github/workflows/pr-reminder.yml` workflow committed to the repo.
2. Pipeline runs on the specified schedule and on manual trigger.
3. If open PRs exist, a GitHub Issue is created with a markdown table of all open PRs.
4. Previous reminder issues are auto-closed so only the latest is active.
5. If no open PRs exist, the workflow exits silently with no notification.
6. GitHub's built-in notifications deliver the reminder via email — no external setup needed.
