# ETL Pipeline Setup — GitHub Repo Based

## Purpose

Scaffold and configure an ETL (Extract, Transform, Load) pipeline in a GitHub repository. The pipeline pulls data from databases or APIs, runs transformation logic, and pushes results to a target database or API. Uses GitHub Actions for orchestration (scheduled or on-demand) with secrets management, logging, and error notifications.

## Prompt

```
Help me set up an ETL pipeline in a GitHub repository.

1. Ask me about the pipeline:
   - **Pipeline name** — a short name for this integration (used for the repo, workflow, and logging).
   - **Schedule** — how often should it run? (cron expression, on-demand via workflow_dispatch, or triggered by an event)
   - **Language** — Python (default), Node.js, or other.

2. Ask me about the data source(s) (Extract):
   - Source type: SQL database, REST API, file (CSV/JSON/Parquet), or cloud storage (S3, Azure Blob).
   - Connection details: host, database, endpoint URL, auth method.
   - What data to extract: query, API endpoint, file path/pattern.
   - Whether credentials are already in environment variables (and their names) or need to be set up.

3. Ask me about the transformation logic (Transform):
   - What transformations are needed? (filtering, mapping, aggregation, joins, data type conversions, deduplication, enrichment)
   - Are there business rules I can describe in plain language?
   - Should intermediate results be logged or saved for debugging?

4. Ask me about the data target(s) (Load):
   - Target type: SQL database, REST API, file, or cloud storage.
   - Connection details: same format as source.
   - Load strategy: full replace, upsert/merge, append-only, or incremental.
   - Error handling: skip failed records, fail entire batch, or dead-letter queue.

5. Scaffold the project with this structure:

   ```
   <pipeline-name>/
   ├── .github/
   │   └── workflows/
   │       └── etl.yml              # GitHub Actions workflow
   ├── src/
   │   ├── extract.py               # Data extraction from source(s)
   │   ├── transform.py             # Transformation logic
   │   ├── load.py                  # Data loading to target(s)
   │   ├── pipeline.py              # Orchestrator — runs extract → transform → load
   │   └── config.py                # Configuration and environment variable loading
   ├── tests/
   │   ├── test_extract.py          # Unit tests for extraction
   │   ├── test_transform.py        # Unit tests for transformations
   │   └── test_load.py             # Unit tests for loading
   ├── requirements.txt             # Python dependencies
   ├── .env.example                 # Template for required environment variables (no secrets)
   ├── .gitignore
   └── README.md                    # Pipeline documentation
   ```

6. Generate the code for each module:

   **extract.py** — connects to the source, pulls data, returns a standard format (list of dicts or DataFrame).
   **transform.py** — accepts extracted data, applies transformations, returns transformed data.
   **load.py** — accepts transformed data, connects to the target, writes data using the chosen load strategy.
   **pipeline.py** — orchestrator that:
   - Logs start time and pipeline name.
   - Calls extract → transform → load in sequence.
   - Catches exceptions at each stage with clear error messages.
   - Logs row counts at each stage (extracted, transformed, loaded).
   - Logs total duration and success/failure status.
   - Returns an exit code (0 = success, 1 = failure).
   **config.py** — loads connection details from environment variables, validates they exist.

7. Generate the GitHub Actions workflow (`etl.yml`) that:
   - Runs on the specified schedule (cron) and supports manual trigger (workflow_dispatch).
   - Checks out the repo, sets up the runtime, installs dependencies.
   - Runs `pipeline.py` with secrets injected from GitHub repo secrets.
   - Sends a notification on failure (GitHub issue, email, or Slack webhook — ask which).
   - Uploads a run log as a workflow artifact for debugging.

8. Set up secrets:
   - List the GitHub repo secrets that need to be configured (e.g., `SOURCE_DB_CONNECTION_STRING`, `TARGET_API_KEY`).
   - Provide the `gh secret set` commands to configure them.
   - Generate a `.env.example` file documenting all required variables (without values).

9. Generate unit tests for each module with sample data.

10. Commit everything and push for review.
```

## Parameters

| Parameter         | Description                                          | Example                          |
| ----------------- | ---------------------------------------------------- | -------------------------------- |
| `PIPELINE_NAME`   | Short name for the integration                       | `invoice-sync`                   |
| `SCHEDULE`        | Cron expression or trigger type                      | `0 6 * * *` (daily at 6am UTC)  |
| `SOURCE_TYPE`     | Database, API, file, or cloud storage                | `postgres`, `rest-api`, `s3`     |
| `TARGET_TYPE`     | Database, API, file, or cloud storage                | `mssql`, `rest-api`, `azure-blob`|
| `LOAD_STRATEGY`   | How to write to the target                           | `upsert`, `append`, `replace`    |
| `LANGUAGE`        | Pipeline implementation language                     | `python`                         |

## Reference

### GitHub Actions workflow example

```yaml
name: ETL — invoice-sync

on:
  schedule:
    - cron: '0 6 * * *'  # Daily at 6:00 UTC
  workflow_dispatch:       # Manual trigger

jobs:
  run-pipeline:
    name: Run ETL Pipeline
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run pipeline
        env:
          SOURCE_DB_HOST: ${{ secrets.SOURCE_DB_HOST }}
          SOURCE_DB_NAME: ${{ secrets.SOURCE_DB_NAME }}
          SOURCE_DB_USER: ${{ secrets.SOURCE_DB_USER }}
          SOURCE_DB_PASSWORD: ${{ secrets.SOURCE_DB_PASSWORD }}
          TARGET_API_URL: ${{ secrets.TARGET_API_URL }}
          TARGET_API_KEY: ${{ secrets.TARGET_API_KEY }}
        run: python src/pipeline.py 2>&1 | tee pipeline.log

      - name: Upload run log
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: pipeline-log-${{ github.run_number }}
          path: pipeline.log
          retention-days: 30

      - name: Notify on failure
        if: failure()
        run: |
          gh issue create \
            --title "ETL failure: invoice-sync — Run #${{ github.run_number }}" \
            --body "Pipeline failed at $(date -u). See [workflow run](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}) for details."
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Pipeline orchestrator pattern

```python
import sys
import time
import logging

from extract import extract
from transform import transform
from load import load

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)
log = logging.getLogger(__name__)

def run():
    pipeline = "invoice-sync"
    start = time.time()
    log.info(f"Pipeline '{pipeline}' started")

    try:
        # Extract
        raw_data = extract()
        log.info(f"Extracted {len(raw_data)} records")

        # Transform
        transformed = transform(raw_data)
        log.info(f"Transformed {len(transformed)} records")

        # Load
        loaded = load(transformed)
        log.info(f"Loaded {loaded} records")

    except Exception as e:
        log.error(f"Pipeline failed: {e}", exc_info=True)
        sys.exit(1)

    elapsed = round(time.time() - start, 2)
    log.info(f"Pipeline '{pipeline}' completed in {elapsed}s")

if __name__ == "__main__":
    run()
```

### Common source/target patterns

**SQL database (extract):**

```python
import os
import pyodbc  # or psycopg2, pymysql

def extract():
    conn_str = os.environ["SOURCE_DB_CONNECTION_STRING"]
    conn = pyodbc.connect(conn_str)
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM invoices WHERE status = 'pending'")
    columns = [desc[0] for desc in cursor.description]
    rows = [dict(zip(columns, row)) for row in cursor.fetchall()]
    conn.close()
    return rows
```

**REST API (extract):**

```python
import os
import requests

def extract():
    url = os.environ["SOURCE_API_URL"]
    headers = {"Authorization": f"Bearer {os.environ['SOURCE_API_KEY']}"}
    response = requests.get(f"{url}/invoices?status=pending", headers=headers)
    response.raise_for_status()
    return response.json()["data"]
```

**SQL database (load — upsert):**

```python
import os
import pyodbc

def load(records):
    conn_str = os.environ["TARGET_DB_CONNECTION_STRING"]
    conn = pyodbc.connect(conn_str)
    cursor = conn.cursor()
    loaded = 0
    for record in records:
        cursor.execute("""
            MERGE INTO invoices AS target
            USING (SELECT ? AS id, ? AS amount, ? AS status) AS source
            ON target.id = source.id
            WHEN MATCHED THEN UPDATE SET amount = source.amount, status = source.status
            WHEN NOT MATCHED THEN INSERT (id, amount, status) VALUES (source.id, source.amount, source.status);
        """, record["id"], record["amount"], record["status"])
        loaded += 1
    conn.commit()
    conn.close()
    return loaded
```

**REST API (load):**

```python
import os
import requests

def load(records):
    url = os.environ["TARGET_API_URL"]
    headers = {
        "Authorization": f"Bearer {os.environ['TARGET_API_KEY']}",
        "Content-Type": "application/json"
    }
    loaded = 0
    for record in records:
        response = requests.post(f"{url}/invoices", json=record, headers=headers)
        response.raise_for_status()
        loaded += 1
    return loaded
```

### Setting up GitHub secrets

```bash
# List the secrets you need
gh secret list

# Set each secret
gh secret set SOURCE_DB_CONNECTION_STRING --body "Server=...;Database=...;Uid=...;Pwd=..."
gh secret set TARGET_API_KEY --body "sk-..."

# Verify (shows names only, not values)
gh secret list
```

### Load strategies

| Strategy    | Behavior                                                    | When to use                        |
| ----------- | ----------------------------------------------------------- | ---------------------------------- |
| `replace`   | Truncate target, then insert all records                    | Small datasets, full refresh       |
| `append`    | Insert all records without checking for duplicates          | Event logs, time-series data       |
| `upsert`    | Insert new records, update existing (match on key)          | Master data, syncing entities      |
| `incremental` | Only process records changed since last run (watermark)  | Large datasets, frequent runs      |

## Expected Outcome

1. A fully scaffolded ETL project with extract, transform, load modules.
2. GitHub Actions workflow for scheduled and on-demand execution.
3. Secrets configured for source and target connections.
4. Logging at each pipeline stage with row counts and duration.
5. Failure notifications via GitHub Issues (or Slack/email).
6. Unit tests for each module.
7. Everything committed and ready for PR review.
