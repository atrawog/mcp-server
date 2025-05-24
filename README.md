# mcp-server

A workspace configured for Model Context Protocol (MCP) integration with Cursor IDE, Claude Desktop, and Claude Code.

## MCP Integration

This workspace includes MCP configuration for Cursor, Claude Desktop, and Claude Code, enabling integration with various external tools and data sources.

### Configuration

MCP configurations are provided for:
- **Cursor**: Configuration at `.cursor/mcp.json`
- **Claude Desktop**: Configuration at `claude_desktop_config.json`
- **Claude Code**: Configuration at `claude_desktop_config.json` (same format)

Both include the following pre-configured servers:

- **filesystem**: Access to local file system (configured for `/home/atrawog/AI` directory)
- **github**: GitHub integration (requires `GITHUB_TOKEN`)
- **notion**: Notion integration (requires `NOTION_API_KEY`)
- **memory**: Persistent memory storage
- **puppeteer**: Web browser automation
- **fetch**: HTTP requests with custom user agent

### Setup

#### For Cursor:
1. Open this workspace in Cursor
2. The MCP servers will be automatically detected from `.cursor/mcp.json`
3. Configure any required API tokens in the configuration file

#### For Claude Desktop:
1. Copy the configuration to Claude's config directory:
   ```bash
   # On macOS
   cp claude_desktop_config.json ~/Library/Application\ Support/Claude/claude_desktop_config.json
   
   # On Windows
   # Copy to %APPDATA%\Claude\claude_desktop_config.json
   
   # On Linux
   cp claude_desktop_config.json ~/.config/Claude/claude_desktop_config.json
   ```
2. Restart Claude Desktop

#### For Claude Code:
1. Copy the configuration to Claude Code's config directory:
   ```bash
   # Create config directory if it doesn't exist
   mkdir -p ~/.config/claude-code
   
   # Copy configuration
   cp claude_desktop_config.json ~/.config/claude-code/mcp_config.json
   ```
2. The configuration will be loaded automatically on next use

#### For all Claude tools:
Configure any required API tokens in the configuration file:
   - Replace `your-github-token-here` with your GitHub personal access token
   - Replace `your-notion-api-key-here` with your Notion integration token

### Usage

Once configured, the MCP tools will be available in Cursor's AI assistant, Claude Desktop, and Claude Code. The assistants will automatically suggest relevant tools based on your requests.

### Adding Custom Servers

To add additional MCP servers:

#### For Cursor:
Edit `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "your-server-name": {
      "command": "npx",
      "args": ["-y", "your-mcp-server-package"],
      "env": {
        "YOUR_ENV_VAR": "value"
      }
    }
  }
}
```

#### For Claude Desktop and Claude Code:
Edit `claude_desktop_config.json` (or the copied config file):

```json
{
  "mcpServers": {
    "your-server-name": {
      "command": "npx",
      "args": ["-y", "your-mcp-server-package"],
      "env": {
        "YOUR_ENV_VAR": "value"
      }
    }
  }
}
```

### Documentation

- For MCP in Cursor: https://docs.cursor.com/context/model-context-protocol
- For MCP in Claude Desktop: https://docs.anthropic.com/en/docs/claude-mcp
- For Claude Code: https://docs.anthropic.com/en/docs/claude-code