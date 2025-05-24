# mcp-server

A workspace configured for Model Context Protocol (MCP) integration with Cursor IDE.

## MCP Integration

This workspace includes MCP configuration for Cursor, enabling integration with various external tools and data sources.

### Configuration

The MCP configuration is located at `.cursor/mcp.json` and includes the following pre-configured servers:

- **filesystem**: Access to local file system (configured for `/home/atrawog/AI` directory)
- **github**: GitHub integration (requires `GITHUB_TOKEN`)
- **notion**: Notion integration (requires `NOTION_API_KEY`)
- **memory**: Persistent memory storage
- **puppeteer**: Web browser automation
- **fetch**: HTTP requests with custom user agent

### Setup

1. Open this workspace in Cursor
2. The MCP servers will be automatically detected from `.cursor/mcp.json`
3. Configure any required API tokens in the configuration file:
   - Replace `your-github-token-here` with your GitHub personal access token
   - Replace `your-notion-api-key-here` with your Notion integration token

### Usage

Once configured, the MCP tools will be available in Cursor's AI assistant. The assistant will automatically suggest relevant tools based on your requests.

### Adding Custom Servers

To add additional MCP servers, edit `.cursor/mcp.json`:

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

For more information about MCP in Cursor, visit: https://docs.cursor.com/context/model-context-protocol