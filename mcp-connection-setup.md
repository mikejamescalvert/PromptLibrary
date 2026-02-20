# MCP Connection Setup — SQL or API Service

## Purpose

Set up a Model Context Protocol (MCP) server connection to a SQL database or an external API service. Walk through configuration, test the connection, and persist the setup in either Claude memory or environment variables for future use.

## Prompt

```
Help me set up an MCP server connection for Claude Code.

1. Ask me what type of connection I need:
   - **SQL Database** (PostgreSQL, MySQL, MSSQL, SQLite)
   - **REST API** (generic API with base URL and auth)
   - **Other** (describe the service)

2. Based on my selection, ask for the required connection details:

   For SQL:
   - Database type (postgres, mysql, mssql, sqlite)
   - Host / connection string
   - Port (if not default)
   - Database name
   - Authentication method (user/password, Windows auth, connection string)
   - Whether credentials are in environment variables (and their names) or should be set up now

   For API:
   - Base URL
   - Authentication type (API key, Bearer token, OAuth, none)
   - Where credentials are stored (environment variable name or to be configured)
   - Any required headers

3. Ask me how I want to persist the connection configuration:
   - **Option A: Claude memory** — Store the connection details (excluding secrets) in memory so they are available in future sessions.
   - **Option B: Environment variables** — Guide me through setting up environment variables for the connection, then store the variable names in memory.
   - **Option C: Both** — Store config in memory and set up environment variables for secrets.

4. Generate the MCP server configuration for `~/.claude/claude_desktop_config.json` (or the appropriate settings file):

   Example SQL (MSSQL) config:
   ```json
   {
     "mcpServers": {
       "my-database": {
         "command": "npx",
         "args": ["-y", "@anthropic/mcp-server-mssql"],
         "env": {
           "MSSQL_CONNECTION_STRING": "${MSSQL_CONN_STR}"
         }
       }
     }
   }
   ```

   Example API config:
   ```json
   {
     "mcpServers": {
       "my-api": {
         "command": "npx",
         "args": ["-y", "@anthropic/mcp-server-fetch"],
         "env": {
           "API_BASE_URL": "https://api.example.com",
           "API_KEY": "${MY_API_KEY}"
         }
       }
     }
   }
   ```

5. Test the connection:
   - For SQL: run a simple query like `SELECT 1` or `SELECT @@VERSION`.
   - For API: make a basic health-check or list request.

6. Save the configuration to memory:
   - Connection type and name
   - Environment variable names used (NOT the values)
   - MCP server name as configured
   - Any notes about the setup
```

## Parameters

| Parameter          | Description                                          | Example                          |
| ------------------ | ---------------------------------------------------- | -------------------------------- |
| `CONNECTION_TYPE`  | SQL database or API service                          | `mssql`, `postgres`, `rest-api`  |
| `CONNECTION_NAME`  | Friendly name for this connection                    | `prod-db`, `billing-api`         |
| `PERSISTENCE`      | Where to store config (memory, env vars, or both)    | `both`                           |

## Supported MCP Server Packages

| Type       | Package                              | Notes                      |
| ---------- | ------------------------------------ | -------------------------- |
| PostgreSQL | `@anthropic/mcp-server-postgres`     | Uses `DATABASE_URL`        |
| MySQL      | `@anthropic/mcp-server-mysql`        | Uses connection string     |
| MSSQL      | `@anthropic/mcp-server-mssql`        | Uses `MSSQL_CONNECTION_STRING` |
| SQLite     | `@anthropic/mcp-server-sqlite`       | Uses file path             |
| REST API   | `@anthropic/mcp-server-fetch`        | Generic HTTP fetch         |

## Expected Outcome

1. MCP server configuration generated and added to the appropriate config file.
2. Connection tested and verified working.
3. Connection details (excluding secrets) saved to Claude memory or environment variables as chosen.
4. Future sessions can reference the connection by name without re-entering details.
