# AI Agent Integration: Setup Guide

SQL Server Comparison Tool (SCT) 6.0 can be used by AI assistants such as **Claude** through the Model Context Protocol (MCP). Once connected, you can ask the AI in plain English to run your saved comparisons and explain the differences between your databases. For example:

- "Are our staging and production databases in sync?"
- "Run comparison 4 and summarize what changed."
- "Which tables exist in production but not in the test database?"

This guide is also available on the [SCT website](https://www.sql-server-tool.com/ai-integration-setup.htm).

> **Safe by design**
> - **Only your saved comparisons.** The AI can run only comparisons that you created and saved in SCT. It cannot connect to arbitrary servers.
> - **No credentials.** Database credentials are stored locally by SCT, encrypted, and are never visible to the AI or sent to it.
> - **Read-only.** The AI can read comparison results, but nothing in this integration executes changes against your databases.

## 1. What is installed

The SCT 6.0 installer places these files in the installation folder, typically `C:\Program Files (x86)\SQL Server Comparison Tool\`:

| File | Purpose |
| --- | --- |
| `sct.exe` | SQL Server Comparison Tool itself. |
| `SctMcpServer.exe` | A small connector that lets AI assistants run SCT. It starts only when your AI application starts it, and uses no resources otherwise. |
| `SctMcpServer.exe.config` | Settings for the connector: paths, timeout and log location. |

## 2. Requirements

- **Windows 10 or 11** (.NET Framework 4.8, preinstalled on both).
- **SQL Server Comparison Tool 6.0 or later.** New users can [download the free 30-day trial](https://www.sql-server-tool.com/sql-server-download.htm).
- **[Claude Desktop](https://claude.ai/download)**, free from Anthropic, or another MCP-compatible application. The browser version of claude.ai cannot connect to programs on your computer, so the Desktop app is required.
- **At least one saved comparison in SCT.** In SCT, run a comparison, then click *Save Comparison*.

## 3. Connect Claude Desktop to SCT

1. **Quit Claude Desktop** if it is running: right-click its icon in the system tray, near the clock, and choose *Quit*.
2. **Open the folder** `%APPDATA%\Claude`: paste this text into the Windows Explorer address bar and press Enter.
3. **Open the file** `claude_desktop_config.json` in Notepad. If the file does not exist, create it with exactly the content below. If it already exists and has other settings, add only the `"SQL Server Comparison Tool"` entry inside `"mcpServers"` (create the `"mcpServers"` section if it is missing).

   ```json
   {
     "mcpServers": {
       "SQL Server Comparison Tool": {
         "command": "C:\\Program Files (x86)\\SQL Server Comparison Tool\\SctMcpServer.exe"
       }
     }
   }
   ```

   **Note the double backslashes** (`\\`) in the path. The file is in JSON format, which requires them. If you installed SCT in a different folder, adjust the path. The name in quotes is what Claude shows in its interface; you can choose any name.

4. **Start Claude Desktop again.**
5. **Check the connection:** click the **+** (or the tools icon) near the message box. You should see **SQL Server Comparison Tool** under Connectors, with the tools `list_saved_comparisons` and `run_comparison`.

## 4. Your first conversation

Open a new chat in Claude Desktop and type:

```
List my saved database comparisons.
```

The first time, Claude asks for permission to use the SCT tool: click *Allow*. You should see your saved comparisons. Then try:

```
Run comparison 1 and explain the differences in plain English.
```

A comparison can take a minute or two on larger databases. Claude shows a timer while SCT works.

## 5. Configuration reference

Settings in `SctMcpServer.exe.config`. The defaults work for a standard installation.

| Setting | Meaning |
| --- | --- |
| `SctExePath` | Full path to `sct.exe`. |
| `OutputDir` | Folder for temporary result files. Created automatically and cleaned after each run. |
| `TimeoutSeconds` | Maximum time for one comparison (default 600). Increase it for very large databases. |
| `LogPath` | Diagnostic log file. Its size is limited automatically, and it is safe to delete at any time. |

## 6. Troubleshooting

### The SCT tools do not appear in Claude Desktop

- Check `claude_desktop_config.json` for JSON mistakes. A missing comma or single backslashes are the most common causes; any online JSON validator will find them.
- Make sure the path in the file really points to `SctMcpServer.exe`.
- Quit Claude Desktop completely (system tray, then *Quit*) and start it again. The configuration is read only at startup.

### Tools stop responding after an SCT update

Restart Claude Desktop fully (system tray, then *Quit*, then start it again). Claude Desktop keeps the connector from before the update until it restarts.

### A comparison fails with a connection or provider error

- First run the same saved comparison in the SCT window. If it fails there too, the problem is the database connection (is the server reachable, is the password still valid?), not the AI integration.
- If it works in SCT but fails through Claude: AI applications may start programs with a reduced set of Windows environment variables. SctMcpServer restores the standard ones automatically, which fixes the known error *"3706: Provider cannot be found"*. If you see a similar error, please contact support and attach the logs listed below.

### A comparison times out

Increase `TimeoutSeconds` in `SctMcpServer.exe.config`, and check that the database servers in that saved comparison are reachable from this computer.

### Collecting logs for support

- The connector log: see `LogPath` in `SctMcpServer.exe.config`.
- The SCT log, in the SCT installation folder.
- Claude Desktop's connector log, in `%APPDATA%\Claude\logs\` (the file name starts with `mcp-server-` followed by the name you used in the config file).

Then use the [contact form](https://www.sql-server-tool.com/sql-server-contact.htm).

## 7. Frequently asked questions

**Can the AI modify my databases?**
No. The integration offers only two operations: listing saved comparisons and running them. SCT's comparison engine only reads schema and data, and the integration has no operation that executes changes against a database.

**Can the AI see my database passwords?**
No. Credentials are stored locally by SCT in encrypted form and are used only inside SCT. They never appear in anything sent to the AI.

**Does my database content get sent to the AI?**
Comparison results are sent to the AI so it can explain them: object names, structural differences and difference counts. For data comparisons, only row counts are included, never the row contents.

**Which AI applications work with this?**
Any application that supports MCP servers over stdio. Claude Desktop is the tested reference; the connector follows the open MCP standard.

**How do I apply the differences the AI finds?**
The same way as always: generate the update script in SCT, review it, and run it yourself. The AI helps you understand what is different; changes stay in your hands.
