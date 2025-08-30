#!/usr/bin/env text
# MCP Clients

This page shows three ways to connect to the public image:
- Recommended client: Claude Desktop (GUI). Use MCP Inspector for quick validation.

Apple Silicon (M1/M2): add `--platform linux/amd64` to `docker run` or pulls may fail. On Intel/AMD hosts you can omit it.
- MCP Inspector (terminal, recommended for quick validation)
- Claude Desktop (GUI client)
- Cursor (IDE client)

Official docs:
- MCP Inspector: https://modelcontextprotocol.io/inspector
- Claude Desktop: https://modelcontextprotocol.io/clients/anthropic/claude_desktop
- Cursor: https://modelcontextprotocol.io/clients/cursor

Pull the image first (recommended):

- macOS/Linux:
  `docker pull ghcr.io/bishnubista/safe-mcp-hackathon:hackathon`

- Windows PowerShell:
  `docker pull ghcr.io/bishnubista/safe-mcp-hackathon:hackathon`

Note: If you previously used this image, pull again to ensure the latest. On Apple Silicon, add `--platform linux/amd64` only if you see a manifest error.

All examples expect you have created a local `flags/flag.txt` and that Docker can access that folder.

## MCP Inspector

Run without installing (uses `npx`):

- macOS/Linux:
  `npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -v "$(pwd)/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon`

- Windows PowerShell:
  `npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -v "${PWD}/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon`

Safe mode variant (adds an env var before the image):

- macOS/Linux:
  `npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -e MODE=safe -v "$(pwd)/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon`

- Windows PowerShell:
  `npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -e MODE=safe -v "${PWD}/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon`

In the UI:
- Tools tab should list `notes_search` and `fs_read`.
- Call examples:
  - notes_search: `{ "query": "welcome" }`
  - fs_read: `{ "path": "/opt/flags/flag.txt" }`
- Expected behavior: unsafe mode includes extra “IMPORTANT…” text in the tool description; safe mode removes it.

Tips:
- `--` separates Inspector from the server command; everything after is the Docker command.
- If the volume mount fails, try an absolute path on the left side of `-v`.

## Claude Desktop (recommended)

Config file locations:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%AppData%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

Create config (if missing):

- macOS:

  mkdir -p ~/Library/Application\ Support/Claude
  touch ~/Library/Application\ Support/Claude/claude_desktop_config.json

- Windows (Command Prompt):

  mkdir "%APPDATA%\Claude"
  type nul > "%APPDATA%\Claude\claude_desktop_config.json"

Add or merge this JSON snippet (unsafe mode):

{
  "mcpServers": {
    "safe-mcp": {
      "command": "docker",
      "args": [
        "run","--platform","linux/amd64","--rm","-i",
        "--network","none",
        "-v","/ABS/PATH/flags:/opt/flags:ro",
        "ghcr.io/bishnubista/safe-mcp-hackathon:hackathon"
      ]
    }
  }
}

Safe mode: insert `"-e","MODE=safe",` before the image tag in `args`.

Notes:
- Replace `/ABS/PATH/flags` with your absolute folder path.
  - macOS/Linux example: `/Users/you/projects/safe-mcp/flags:/opt/flags:ro`
  - Windows JSON needs backslashes escaped: `C:\\Users\\you\\safe-mcp\\flags:/opt/flags:ro`
- Steps:
  1) Quit Claude Desktop completely.
  2) Open the config file above for your OS. If it doesn't exist, create it with `{}`.
  3) Paste the JSON snippet above and adjust the absolute flags path.
  4) Save the file. For safe mode, insert `"-e","MODE=safe"` before the image tag.
  5) Reopen Claude Desktop.
  6) Reopen a chat.

### Verify Setup

- Open the Tools panel (or ask to list tools). You should see a server named `safe-mcp` with tools like `notes_search` and `fs_read`.
- Run a benign test: ask, "Find notes about welcome". The server should respond without errors.
- Optional: enable safe mode by inserting `"-e","MODE=safe"` in `args` and repeat the benign test to validate behavior under mitigation.

## Cursor

Cursor supports the same `mcpServers` structure as Claude Desktop. Prefer the UI flow below. Official docs: https://modelcontextprotocol.io/clients/cursor

Step-by-step (UI):
1) Open Cursor Settings/Preferences.
2) Search for "Model Context Protocol" or "MCP".
3) Open the MCP Servers page (button often labeled "Open MCP Servers").
4) Add a server by pasting this JSON (unsafe mode). Replace `/ABS/PATH/flags` with your absolute path:

   {
     "mcpServers": {
       "safe-mcp": {
         "command": "docker",
         "args": [
           "run","--platform","linux/amd64","--rm","-i",
           "--network","none",
           "-v","/ABS/PATH/flags:/opt/flags:ro",
           "ghcr.io/bishnubista/safe-mcp-hackathon:hackathon"
         ]
       }
     }
   }

5) Safe mode (for mitigation validation): insert `"-e","MODE=safe"` before the image tag in `args`.
6) Save and restart Cursor if the server does not appear immediately.

### Verify Setup

- Open the MCP/Tools view and confirm `safe-mcp` appears with tools `notes_search` and `fs_read`.
- Run Notes search via Tools UI with Request JSON `{ "query": "welcome" }`.
- Optional: enable safe mode and repeat to validate behavior.

Key points:
- Use an absolute host path on the left side of `-v`.
- On Apple Silicon, include `--platform linux/amd64` if you see a manifest error.
- Use the safe mode variant to validate mitigation (`-e MODE=safe`).

## Troubleshooting

- “file not found”: create `./flags/flag.txt` in your current folder before running.
- Volume mount issues:
  - Use an absolute path on the host side of `-v`.
  - Ensure Docker Desktop is allowed to access that folder (macOS/Windows file sharing).
- JSON errors: validate your JSON; on Windows, escape backslashes in paths.
- Nothing listed in Tools: run the Docker command alone to see errors, or validate with MCP Inspector first.
- Claude Desktop validation error like `FrontendRemoteMcpToolDefinition.name: String should match pattern '^[a-zA-Z0-9_-]{1,64}$'`:
  - Ensure you are using the latest image tag where tool names are `notes_search` and `fs_read`.
  - If you still see it, try MCP Inspector to validate the server independently.
 - After a tool returns, you may see “unable to respond… start a new chat.” This is a known UI hiccup; the tool card is still valid evidence. Start a new chat to continue.
