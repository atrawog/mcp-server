# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) configuration workspace that provides integration between various AI assistants (Cursor IDE, Claude Desktop, and Claude Code) and external tools/data sources.

## Architecture

The repository contains MCP server configurations that enable AI assistants to interact with:
- Local filesystem (restricted to `/home/atrawog/AI` directory)
- GitHub API (requires `GITHUB_TOKEN`)
- Notion API (requires `NOTION_API_KEY`)
- Memory persistence
- Web browser automation via Puppeteer
- HTTP requests with custom user agent

Configuration files:
- `.cursor/mcp.json` - MCP configuration for Cursor IDE
- `claude_desktop_config.json` - MCP configuration for Claude Desktop and Claude Code (identical format)

## Key Configuration Details

All MCP servers are configured to run via `npx` with the `-y` flag for automatic package installation. The servers are from the `@modelcontextprotocol` npm organization.

When modifying configurations:
1. Ensure JSON syntax is valid
2. Environment variables in the `env` section should be replaced with actual values
3. The `ALLOWED_DIRECTORIES` for filesystem access can be modified to include additional paths

## Adding New MCP Servers

To add a new MCP server, add an entry to the `mcpServers` object in both configuration files:

```json
"server-name": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-package-name"],
  "env": {
    "REQUIRED_ENV_VAR": "value"
  }
}
```