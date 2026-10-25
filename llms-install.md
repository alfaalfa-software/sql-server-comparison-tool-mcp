# Installation notes for AI assistants

These notes help an AI assistant set up the SQL Server Comparison Tool MCP server for a user.

## What this server is

- A local stdio MCP server, `SctMcpServer.exe`, installed together with SQL Server Comparison Tool (SCT) 6.0 or later.
- Windows only (Windows 10 or 11, .NET Framework 4.8).
- It is not distributed as an npm, PyPI or Docker package. The user must install SCT first: https://www.sql-server-tool.com/sql-server-download.htm (free 30-day trial).

## Steps

1. Confirm SCT 6.0 or later is installed. The default location of the server is:
   `C:\Program Files (x86)\SQL Server Comparison Tool\SctMcpServer.exe`
   If it is not there, ask the user where SCT is installed.
2. Ask the user to open SCT and save at least one comparison (run a comparison, then click *Save Comparison*). The server can only run saved comparisons.
3. Add this entry to the MCP client configuration. For Claude Desktop the file is `%APPDATA%\Claude\claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "SQL Server Comparison Tool": {
      "command": "C:\\Program Files (x86)\\SQL Server Comparison Tool\\SctMcpServer.exe"
    }
  }
}
```

   Keep any existing entries in `mcpServers`. Use double backslashes in the path.
4. The server takes no arguments, environment variables or API keys. Database connections and credentials are managed inside SCT and are never passed through MCP.
5. Restart the MCP client fully, then test with: "List my saved database comparisons."

## Tools

- `list_saved_comparisons`: returns the comparisons saved in SCT.
- `run_comparison`: runs one saved comparison by id and returns structured JSON results. Comparisons can take a minute or more on large databases.

The server is read-only: it never changes databases.

Full guide: [docs/setup-guide.md](docs/setup-guide.md)
