# SAFE MCP Hackathon

What this is about:
- There is an intentional vulnerability in a demo MCP server. Your job is to find it, map it to one of the techniques from https://github.com/fkautz/safe-mcp/tree/main/techniques, and propose a mitigation. Treat it like a real investigation: gather evidence, reason about impact, map, then validate mitigation.

## Quickstart (2–3 minutes)

- Pull the image: `docker pull ghcr.io/bishnubista/safe-mcp-hackathon:hackathon`
  - Apple Silicon: add `--platform linux/amd64` only if you see a manifest error.
- Prepare a flag file:
  - macOS/Linux: `mkdir -p ./flags && echo "hello" > ./flags/flag.txt`
  - Windows PowerShell: `New-Item -Type Directory -Force -Path .\flags | Out-Null; Set-Content .\flags\flag.txt "hello"`
- Add the server to Claude Desktop (recommended): paste the JSON from “Detailed Guide”, then restart Claude and verify tools `notes_search`, `fs_read` appear.
- Try it: ask “Find notes about welcome”. Then explicitly call tools:
  - notes_search: `{ "query": "welcome" }`
  - fs_read: `{ "path": "/opt/flags/flag.txt" }`

Need step-by-step? See the Detailed Guide:
- Detailed setup (Claude Desktop, Cursor, Inspector): [Detailed Guide](setup.md)

## Deliverables (what to submit)

- Technique mapping: list applicable safe‑mcp ID(s) (e.g., SAFE‑T####) and why.
- Evidence: relevant tool metadata and a transcript/screenshot showing impact.
- Mitigation: proposal (server/client/policy) and trade‑offs.
- Validation: show the same flow with mitigation enabled and explain the outcome.
- Keep it 1–2 pages. Template: [`SUBMISSION_TEMPLATE.md`](SUBMISSION_TEMPLATE.md).

## Rules & Safety

- Only mount `flags/`; do not mount sensitive host paths.
- Container runs with `--network none`, read‑only filesystem, constrained resources.
- Internet allowed for docs/reference; no external scanning.
- Keep transcripts local; do not share real secrets.

## References

- MCP Inspector: https://modelcontextprotocol.io/inspector
- Claude Desktop: https://modelcontextprotocol.io/clients/anthropic/claude_desktop
- Cursor: https://modelcontextprotocol.io/clients/cursor
