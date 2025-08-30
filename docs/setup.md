# Detailed Setup Guide

Follow these steps if you want a guided setup with exact commands for macOS, Windows, and Linux.

## Prerequisites

- Docker Desktop installed and running
- A terminal (Terminal or PowerShell)

## 1) Pull the Public Image

- macOS/Linux:

  docker pull ghcr.io/bishnubista/safe-mcp-hackathon:hackathon

- Windows PowerShell:

  docker pull ghcr.io/bishnubista/safe-mcp-hackathon:hackathon

Notes:
- If you previously pulled the image, pull again to get the latest update.
- Apple Silicon (M1/M2): add `--platform linux/amd64` only if you hit a manifest error.

## 2) Create the Flags File

The server expects a read‑only volume mounted at `/opt/flags` with a file `flag.txt`.

- macOS/Linux:

  mkdir -p ./flags
  echo "hello" > ./flags/flag.txt

- Windows PowerShell:

  New-Item -ItemType Directory -Force -Path .\flags | Out-Null
  Set-Content -Path .\flags\flag.txt -Value "hello"

## 3) Configure Claude Desktop (recommended)

Config file locations:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%AppData%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

Create the config file if missing:

- macOS:

  mkdir -p ~/Library/Application\ Support/Claude
  touch ~/Library/Application\ Support/Claude/claude_desktop_config.json

- Windows (Command Prompt):

  mkdir "%APPDATA%\Claude"
  type nul > "%APPDATA%\Claude\claude_desktop_config.json"

Add or merge this JSON (unsafe mode). Replace `/ABS/PATH/flags` with your absolute path to the `flags` folder.

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

Safe mode variant (to validate mitigations): insert `"-e","MODE=safe"` before the image tag in `args`.

Apply and verify:
1) Quit Claude Desktop completely.
2) Save the config file.
3) Reopen Claude Desktop and a new chat.
4) Verify the Tools list shows `notes_search` and `fs_read`.

## 4) Configure Cursor (optional)

Cursor supports the same `mcpServers` structure as Claude Desktop. Use its MCP settings UI and paste the same JSON above, adjusting your absolute flags path. Then verify a server named `safe-mcp` appears with tools `notes_search` and `fs_read`.

Official docs: https://modelcontextprotocol.io/clients/cursor

## 5) Quick CLI Check with MCP Inspector

Run without installing (uses `npx`). Replace the left side of `-v` with your absolute path if needed.

- macOS/Linux:

  npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -v "$(pwd)/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon

- Windows PowerShell:

  npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -v "${PWD}/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon

Safe mode variant:

- macOS/Linux:

  npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -e MODE=safe -v "$(pwd)/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon

- Windows PowerShell:

  npx @modelcontextprotocol/inspector -- docker run --platform linux/amd64 --rm -i --network none -e MODE=safe -v "${PWD}/flags:/opt/flags:ro" ghcr.io/bishnubista/safe-mcp-hackathon:hackathon

## 6) Verify Behavior

- Benign search:
  - Use `notes_search` with `{ "query": "welcome" }` and observe the response.
- File read check (unsafe mode):
  - Use `fs_read` with `{ "path": "/opt/flags/flag.txt" }`.
- Mitigation check:
  - Switch to safe mode and repeat the benign search to compare behavior.

## Troubleshooting

- “file not found”: ensure `./flags/flag.txt` exists and the host path on the left of `-v` is an absolute path if needed.
- Nothing listed in Tools: run the raw Docker command to see server output, or validate with MCP Inspector first.
- On Windows JSON, escape backslashes in paths: `C\\\\Users\\\\you\\\\flags:/opt/flags:ro`.
