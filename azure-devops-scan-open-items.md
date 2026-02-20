# Azure DevOps — Scan Open Work Items

## Purpose

Connect to an Azure DevOps repository using a local PAT stored in an environment variable, retrieve all open work items (bugs, tasks, user stories, etc.), and provide a structured analysis. Store the environment variable name in memory for future operations.

## Prompt

```
I have a Personal Access Token for Azure DevOps stored in a local environment variable.

1. Ask me for:
   - The environment variable name that holds my Azure DevOps PAT.
   - The Azure DevOps organization name.
   - The project name.
2. Verify the PAT is accessible by reading the environment variable (do NOT print the token).
3. Use the Azure DevOps REST API to fetch all open work items across the project:
   - Bugs, Tasks, User Stories, Features, Epics, and any other work item types present.
   - Use WIQL query: `SELECT [System.Id] FROM WorkItems WHERE [System.State] <> 'Closed' AND [System.State] <> 'Removed' AND [System.State] <> 'Done' ORDER BY [System.CreatedDate] DESC`
   - Fetch full details for each work item (title, state, type, assigned to, iteration, area path, tags, created date).
4. Provide an analysis covering:
   - **Summary** — Total open item count, broken down by work item type and state.
   - **Ownership** — Who has the most items assigned? Are there unassigned items?
   - **Age analysis** — Flag items that have been open for an unusually long time.
   - **Iteration/sprint status** — Items in the current vs. past iterations (potential carryover).
   - **Risks & observations** — Blocked items, items with no recent updates, or patterns worth attention.
5. Store the environment variable name, organization, and project in memory so future prompts can reuse them without asking again.
```

## Parameters

| Parameter           | Description                                              | Example                  |
| ------------------- | -------------------------------------------------------- | ------------------------ |
| `ENV_VAR_NAME`      | Name of the environment variable holding the DevOps PAT  | `AZURE_DEVOPS_PAT`      |
| `ORG_NAME`          | Azure DevOps organization name                           | `mycompany`              |
| `PROJECT_NAME`      | Azure DevOps project name                                | `BackendServices`        |

## API Reference

Base URL: `https://dev.azure.com/{ORG_NAME}/{PROJECT_NAME}/_apis`

Key endpoints used:

| Endpoint                            | Purpose                        |
| ----------------------------------- | ------------------------------ |
| `POST _apis/wit/wiql?api-version=7.1` | Execute a WIQL query           |
| `GET _apis/wit/workitems?ids={ids}&$expand=all&api-version=7.1` | Fetch work item details |

Authentication: Basic auth with empty username and the PAT as password.

```
curl -s -u ":{PAT}" "https://dev.azure.com/{ORG}/{PROJECT}/_apis/wit/wiql?api-version=7.1" \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT [System.Id] FROM WorkItems WHERE [System.State] <> '\''Closed'\'' AND [System.State] <> '\''Removed'\'' AND [System.State] <> '\''Done'\'' ORDER BY [System.CreatedDate] DESC"}'
```

## Expected Outcome

1. Environment variable validated (token exists, not empty).
2. Full list of open work items retrieved and displayed in a table.
3. Structured analysis with summary, ownership, age, iteration, and risk sections.
4. Environment variable name, org, and project saved to Claude memory for future sessions.
