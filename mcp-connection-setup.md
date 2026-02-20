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

4. Register the MCP server using the Claude Code CLI or by editing the appropriate config file:
   - Preferred: use `claude mcp add` to register the server.
   - Manual: add to `.mcp.json` (project-level, shared via source control) or `~/.claude.json` (user-level, private).
   - Do NOT use `claude_desktop_config.json` — that is for the Claude Desktop app, not Claude Code.
   - On Windows (not WSL), wrap npx commands with `cmd /c`.
   - See the Reference section below for config examples and correct package names.

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

## Reference

### MCP Server Packages

The recommended multi-database MCP server is `@bytebase/dbhub`, which supports PostgreSQL, MySQL, MSSQL, and SQLite through a single package using DSN connection strings.

| Type       | Package                                  | Notes                                          |
| ---------- | ---------------------------------------- | ---------------------------------------------- |
| Multi-DB   | `@bytebase/dbhub`                        | Recommended. Supports Postgres, MySQL, MSSQL, SQLite via `--dsn` |
| PostgreSQL | `@modelcontextprotocol/server-postgres`  | Deprecated/archived. Use dbhub instead         |
| SQLite     | `@modelcontextprotocol/server-sqlite`    | Deprecated/archived. Use dbhub instead         |
| REST API   | `mcp-server-fetch` (Python/PyPI)         | Run via `uvx mcp-server-fetch`. Not an npm package |

### Config File Locations

| Scope   | File                  | Purpose                                    |
| ------- | --------------------- | ------------------------------------------ |
| Project | `.mcp.json`           | Shared via source control with the team    |
| User    | `~/.claude.json`      | Private to you, available in all projects  |

### CLI Registration (preferred)

```bash
# SQL via dbhub
claude mcp add my-db -- npx -y @bytebase/dbhub@latest --dsn "postgresql://user:pass@localhost:5432/mydb"

# Fetch (Python-based)
claude mcp add fetch -- uvx mcp-server-fetch

# List configured servers
claude mcp list
```

### Manual Config Examples

**SQL database via dbhub** (`.mcp.json`):

```json
{
  "mcpServers": {
    "my-database": {
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub@latest", "--dsn", "postgresql://user:pass@localhost:5432/mydb"]
    }
  }
}
```

**SQL database via dbhub on Windows** (`.mcp.json`):

```json
{
  "mcpServers": {
    "my-database": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@bytebase/dbhub@latest", "--dsn", "postgresql://user:pass@localhost:5432/mydb"]
    }
  }
}
```

**REST API fetch** (`.mcp.json`):

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

### DSN Formats for dbhub

| Database   | DSN Format                                                  |
| ---------- | ----------------------------------------------------------- |
| PostgreSQL | `postgresql://user:pass@host:5432/dbname`                   |
| MySQL      | `mysql://user:pass@host:3306/dbname`                        |
| MSSQL      | `sqlserver://user:pass@host:1433/dbname`                    |
| SQLite     | `sqlite:///path/to/database.db`                             |

## Expected Outcome

1. MCP server registered via CLI or config file.
2. Connection tested and verified working.
3. Connection details (excluding secrets) saved to Claude memory or environment variables as chosen.
4. Future sessions can reference the connection by name without re-entering details.
