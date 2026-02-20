# Manual Test Recording with Screenshots

## Purpose

Facilitate manual testing of websites or program functionality by guiding the user through test steps, analyzing screenshots they provide at each step, recording pass/fail results, and producing a structured test report. Optionally act on failures by filing bugs or creating work items.

## Prompt

```
Help me run and record a manual test session.

1. Ask me what I'm testing:
   - Application or website name and URL (if applicable).
   - What feature or workflow am I testing?
   - What environment is this? (dev, staging, production)

2. Help me define test cases. Either:
   - I'll describe the workflow and you break it into numbered test steps with expected outcomes.
   - Or I'll provide a list of test cases to use as-is.
   Each test step should have:
   - **Step number and name**
   - **Action** — what to do (click, navigate, enter data, etc.)
   - **Expected result** — what should happen if the step passes.

3. Walk me through each test step one at a time:
   - Tell me what action to perform.
   - Ask me to take a screenshot and share it.
   - When I share a screenshot, analyze it carefully:
     - Describe what you see on screen.
     - Compare it against the expected result.
     - Record the step as **Pass**, **Fail**, or **Blocked**.
     - If it fails, ask me for any additional context and record the failure details.
   - Move to the next step.

4. If a step fails or is blocked:
   - Record the failure with: step number, expected vs. actual result, and screenshot reference.
   - Ask if I want to continue testing remaining steps or stop.
   - Ask if I want to file a bug (see "Acting on failures" below).

5. After all steps are complete (or testing is stopped), produce a test report:

   **Test Report**
   - Application: [name]
   - Environment: [env]
   - Date: [today's date]
   - Tester: [name if provided]

   | Step | Name | Result | Notes |
   |------|------|--------|-------|
   | 1    | ...  | Pass   |       |
   | 2    | ...  | Fail   | Expected X, got Y (screenshot #2) |

   **Summary**: X passed, Y failed, Z blocked out of N total steps.
   **Overall result**: Pass / Fail

6. Acting on failures (ask which, if any, to use):
   - **Save report to file** — write the test report as a markdown file in the current directory.
   - **Create GitHub issue** — file a bug for each failure using `gh issue create`.
   - **Create Azure DevOps work item** — file a bug via the DevOps REST API (uses PAT from memory if previously configured).
   - **Do nothing** — just display the report.
```

## Parameters

| Parameter         | Description                                          | Example                          |
| ----------------- | ---------------------------------------------------- | -------------------------------- |
| `APPLICATION`     | Name of the application or website being tested      | `Invoice Portal`                 |
| `ENVIRONMENT`     | Test environment                                     | `staging`                        |
| `WORKFLOW`        | Feature or workflow under test                       | `User login and password reset`  |

## Reference

### Screenshot analysis tips

When analyzing a screenshot, check for:
- Page title, URL bar, or breadcrumbs to confirm correct navigation.
- Form fields — are they populated correctly? Are there validation errors?
- Success/error messages — banners, toasts, modals.
- UI state — buttons enabled/disabled, loading spinners, empty states.
- Data correctness — do displayed values match what was entered or expected?
- Console errors or network failures if dev tools are visible.

### Filing a GitHub issue from a failure

```bash
gh issue create --title "Bug: [Step Name] — [short description]" --body "$(cat <<'EOF'
## Steps to reproduce

1. [action from the test step]

## Expected result

[expected result from the test plan]

## Actual result

[what was observed in the screenshot]

## Environment

- Application: [name]
- Environment: [env]
- Date: [date]

## Screenshot

[reference to screenshot file or embed]
EOF
)"
```

### Filing an Azure DevOps bug from a failure

```bash
curl -s -u ":{PAT}" "https://dev.azure.com/{ORG}/{PROJECT}/_apis/wit/workitems/$Bug?api-version=7.1" \
  -H "Content-Type: application/json-patch+json" \
  -d '[
    {"op": "add", "path": "/fields/System.Title", "value": "Bug: [Step Name] — [short description]"},
    {"op": "add", "path": "/fields/Microsoft.VSTS.TCM.ReproSteps", "value": "<p>Steps to reproduce: ...</p><p>Expected: ...</p><p>Actual: ...</p>"}
  ]'
```

### Saving a screenshot for reference

If the user shares a screenshot as a file path, it can be copied into the test output directory:

```bash
mkdir -p test-results
cp /path/to/screenshot.png test-results/step-01-login-page.png
```

## Expected Outcome

1. Structured test steps defined before testing begins.
2. Each screenshot analyzed and compared against expected results in real time.
3. Pass/fail recorded per step with failure details captured.
4. A complete test report produced at the end — displayable, saveable, or actionable.
5. Optionally, bugs filed in GitHub Issues or Azure DevOps for any failures.
