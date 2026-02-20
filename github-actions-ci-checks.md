# GitHub Actions — CI Checks for PR Gating

## Purpose

Generate a GitHub Actions workflow that runs common tests and checks on every pull request. Configure the checks as required status checks so PRs cannot be merged until they pass.

## Prompt

```
Set up a GitHub Actions CI workflow for this repository that runs on every pull request.

1. Ask me about the project:
   - What language/framework is this project using? (or infer from the repo contents)
   - What checks should run? Choose from:
     - **Lint** — static analysis / code style (e.g., ESLint, Pylint, Ruff, RuboCop)
     - **Type check** — type validation (e.g., TypeScript tsc, mypy, Pyright)
     - **Unit tests** — test suite execution (e.g., Jest, pytest, Go test, JUnit)
     - **Build** — verify the project compiles/builds successfully
     - **Security scan** — dependency vulnerability check (e.g., npm audit, pip-audit, Trivy)
     - **Formatting** — verify code is properly formatted (e.g., Prettier, Black, gofmt)
   - Any specific versions or configurations to use? (Node version, Python version, etc.)

2. Generate a `.github/workflows/ci.yml` file that:
   - Triggers on `pull_request` targeting `main` (and any other protected branches specified).
   - Runs the selected checks as separate jobs (so failures are easy to identify).
   - Uses caching where appropriate (node_modules, pip cache, Go modules, etc.).
   - Installs dependencies before running checks.
   - Uses pinned action versions (e.g., `actions/checkout@v4`, not `@latest`).
   - Fails fast — if any check fails, the PR is blocked.

3. Provide instructions to enable branch protection:
   - How to set the workflow's jobs as required status checks in GitHub repo settings.
   - Recommend enabling "Require branches to be up to date before merging".

4. Commit the workflow file and push to the current branch.
```

## Parameters

| Parameter          | Description                                          | Example                          |
| ------------------ | ---------------------------------------------------- | -------------------------------- |
| `LANGUAGE`         | Primary language/framework                           | `typescript`, `python`, `go`     |
| `CHECKS`           | Which checks to include                              | `lint, test, build`              |
| `NODE_VERSION`     | Runtime version (if applicable)                      | `20`, `3.12`, `1.22`            |
| `PROTECTED_BRANCH` | Branch(es) that require passing checks               | `main`                           |

## Reference

### Workflow structure

Each check runs as a separate job so individual failures are clearly visible in the PR status:

```yaml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps: ...
  test:
    runs-on: ubuntu-latest
    steps: ...
  build:
    runs-on: ubuntu-latest
    steps: ...
```

### Common check configurations by language

**Node / TypeScript:**

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx tsc --noEmit

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
```

**Python:**

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'
      - run: pip install -r requirements.txt
      - run: pip install ruff
      - run: ruff check .

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'
      - run: pip install -r requirements.txt
      - run: pip install mypy
      - run: mypy .

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'
      - run: pip install -r requirements.txt
      - run: pytest

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install pip-audit
      - run: pip-audit -r requirements.txt
```

**Go:**

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      - uses: golangci/golangci-lint-action@v6

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      - run: go test ./...

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      - run: go build ./...
```

### Enabling required status checks

After the workflow runs at least once on a PR:

1. Go to **Settings > Branches > Branch protection rules**.
2. Click **Add rule** (or edit the existing rule for `main`).
3. Enable **Require status checks to pass before merging**.
4. Search for and select each job name (e.g., `lint`, `test`, `build`).
5. Enable **Require branches to be up to date before merging**.
6. Save changes.

Alternatively, use the `gh` CLI:

```bash
gh api repos/{OWNER}/{REPO}/branches/main/protection -X PUT \
  -f "required_status_checks[strict]=true" \
  -f "required_status_checks[contexts][]=lint" \
  -f "required_status_checks[contexts][]=test" \
  -f "required_status_checks[contexts][]=build" \
  -f "enforce_admins=false" \
  -f "required_pull_request_reviews=null" \
  -f "restrictions=null"
```

## Expected Outcome

1. A `.github/workflows/ci.yml` file committed to the repo.
2. Selected checks run automatically on every PR targeting protected branches.
3. Each check is a separate job with clear pass/fail status visible in the PR.
4. Branch protection configured so PRs cannot merge until all checks pass.
