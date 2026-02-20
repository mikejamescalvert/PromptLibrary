# PR Review Reminder Pipeline

## Purpose

Set up a GitHub Actions pipeline that checks a repository for open pull requests on a daily schedule and sends an email summary to the repo owner (or a specified recipient) when any exist. Follows the ETL pattern: extract open PRs from GitHub, transform into a formatted email, and load by sending the notification.

## Prompt

```
Set up a PR review reminder pipeline for this repository.

1. Ask me for:
   - **Schedule** — when should the check run? (default: weekdays at 2:00 PM UTC)
   - **Recipient email** — who should receive the reminder?
   - **SMTP configuration** — ask whether I already have SMTP secrets configured in the repo, or if I need help setting them up (server, port, username, password).

2. Create a GitHub Actions workflow (`.github/workflows/pr-reminder.yml`) that:

   **Extract:**
   - Uses `gh pr list` to fetch all open PRs with details (number, title, author, created date, URL, branch name).
   - If there are no open PRs, skip the notification and exit successfully.

   **Transform:**
   - Formats the PR data into an HTML email with a table showing each PR's number (linked), title, author, branch, and age.
   - Includes a direct link to the repo's PR page.
   - Includes the total count of open PRs in the subject line and body.

   **Load:**
   - Sends the email via SMTP using the `dawidd6/action-send-mail` action.
   - Uses GitHub repo secrets for all credentials (never hardcoded).

3. The workflow should also support `workflow_dispatch` so it can be triggered manually for testing.

4. Set up the required GitHub repo secrets:
   - `SMTP_SERVER` — SMTP host (e.g., smtp.gmail.com, smtp.office365.com)
   - `SMTP_PORT` — SMTP port (e.g., 587 for TLS)
   - `SMTP_USERNAME` — sender email address
   - `SMTP_PASSWORD` — sender email password or app password
   - `NOTIFY_EMAIL` — recipient email address
   Provide the `gh secret set` commands for each.

5. Commit the workflow and push for review.
```

## Parameters

| Parameter        | Description                                      | Example                          |
| ---------------- | ------------------------------------------------ | -------------------------------- |
| `SCHEDULE`       | Cron expression for when to check                | `0 14 * * 1-5` (weekdays 2pm UTC) |
| `NOTIFY_EMAIL`   | Email address to receive reminders               | `owner@company.com`              |
| `SMTP_SERVER`    | SMTP server hostname                             | `smtp.gmail.com`                 |
| `SMTP_PORT`      | SMTP port                                        | `587`                            |

## Reference

### Required GitHub secrets

| Secret           | Description                          | Example                     |
| ---------------- | ------------------------------------ | --------------------------- |
| `SMTP_SERVER`    | SMTP server hostname                 | `smtp.gmail.com`            |
| `SMTP_PORT`      | SMTP port (TLS)                      | `587`                       |
| `SMTP_USERNAME`  | Sender email / login                 | `noreply@company.com`       |
| `SMTP_PASSWORD`  | Email password or app-specific password | (app password)           |
| `NOTIFY_EMAIL`   | Recipient email address              | `owner@company.com`         |

### Setting secrets via CLI

```bash
gh secret set SMTP_SERVER --body "smtp.gmail.com"
gh secret set SMTP_PORT --body "587"
gh secret set SMTP_USERNAME --body "noreply@company.com"
gh secret set SMTP_PASSWORD  # prompts for value (recommended for passwords)
gh secret set NOTIFY_EMAIL --body "owner@company.com"
```

### Common SMTP servers

| Provider         | Server                    | Port | Notes                                |
| ---------------- | ------------------------- | ---- | ------------------------------------ |
| Gmail            | `smtp.gmail.com`          | 587  | Requires app password (2FA enabled)  |
| Outlook/O365     | `smtp.office365.com`      | 587  | May require admin consent             |
| SendGrid         | `smtp.sendgrid.net`       | 587  | Uses API key as password             |
| Amazon SES       | `email-smtp.us-east-1.amazonaws.com` | 587  | Uses IAM SMTP credentials  |

### Email output example

The email includes an HTML table:

| #   | Title                          | Author          | Branch              | Opened     |
| --- | ------------------------------ | --------------- | ------------------- | ---------- |
| #12 | Add user authentication        | `dev-alice`     | `feat/auth`         | 2026-02-18 |
| #10 | Fix invoice calculation bug    | `dev-bob`       | `fix/invoice-calc`  | 2026-02-15 |

### Adapting to other notification channels

The workflow can be adapted to send notifications via other channels:

**Slack (using webhook):**

```yaml
- name: Notify via Slack
  if: success()
  run: |
    curl -X POST "${{ secrets.SLACK_WEBHOOK_URL }}" \
      -H "Content-Type: application/json" \
      -d "{\"text\": \"PR Review Reminder: ${COUNT} open PR(s) in ${{ github.repository }}\"}"
```

**GitHub Issue (no external secrets needed):**

```yaml
- name: Create reminder issue
  if: success()
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    gh issue create \
      --title "PR Review Reminder: ${COUNT} open PR(s)" \
      --body "$BODY_MARKDOWN"
```

## Expected Outcome

1. A `.github/workflows/pr-reminder.yml` workflow committed to the repo.
2. Pipeline runs daily (or on the specified schedule) and on manual trigger.
3. If open PRs exist, an HTML email is sent listing each PR with links.
4. If no open PRs exist, the workflow exits silently with no notification.
5. GitHub repo secrets configured for SMTP access.
