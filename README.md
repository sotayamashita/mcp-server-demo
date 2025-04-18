# MCP Server Demo

## Prerequisites

- [uv](https://docs.astral.sh/uv/getting-started/installation/) - _An extremely fast Python package and project manager, written in Rust._

## Overview

This guide walks through creating a simple Model Context Protocol (MCP) server and testing it with both the MCP Inspector and Claude Desktop. The content is based on the [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk/blob/c2ca8e03e046908935d089a2ceed4e80b0c29a24/README.md) README, with additional notes for clarity.

## Steps to Create an MCP Server

### 1. Environment Setup

```bash
uv init mcp-server-demo
cd mcp-server-demo

# Specify Python version
echo "3.12" > .python-version

# Install Python before creating a new virtual environment
uv venv 

# Activate virtual environment
source .venv/bin/activate.fish

# Verify virtual environment (check if using the correct Python environment)
which python
path/to/mcp-server-demo/.venv/bin/python

# Install MCP dependencies (added to pyproject.toml)
uv add "mcp[cli]"

# Verify MCP dependencies were installed correctly
uv tree

Resolved 28 packages in 3ms
mcp-server-demo v0.1.0
└── mcp[cli] v1.6.0
    ├── ...
```

### 2. Creating a Simple MCP Server

```python
# server.py
from mcp.server.fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP("Demo")

# Add an addition tool
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b


# Add a dynamic greeting resource
@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """Get a personalized greeting"""
    return f"Hello, {name}!"
```

### 3. Testing with MCP Inspector

#### Launch MCP Inspector

```bash
uv run mcp dev server.py

Starting MCP inspector...
⚙️ Proxy server listening on port 6277
```

#### Testing the `add` Tool

1. Open the MCP Inspector
2. Click "Connect"
3. Click the "Tools" tab
4. Click "List Tools" inside the "Tools" tab

#### Testing the `get_greeting` Resource

1. Open the MCP Inspector
2. Click "Connect"
3. Click "Resources" Tab
4. Click "List Templates"
5. Find "get_greeting"
6. Type a name (e.g., "John")
7. Click "Read Resources"

Example response:
```json
{
  "contents": [
    {
      "uri": "greeting://John",
      "mimeType": "text/plain",
      "text": "Hello, John!"
    }
  ]
}
```

### 4. Testing with Claude Desktop

#### Install the MCP Server in Claude Desktop

```bash
uv run mcp install server.py

[/dd/MM/YY HH:MM:SS] INFO     Added server 'Demo' to Claude config            claude.py:129
                     INFO     Successfully installed Demo in Claude app          cli.py:467
```

#### Verify Configuration

```bash
# For macOS/Linux
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

Example configuration:

```json
{
    "mcpServers": {
        "Demo": {
            "command": "/absolute/path/to/uv",
            "args": [
                "run",
                "--with",
                "mcp[cli]",
                "mcp",
                "run",
                "/absolute/path/to/mcp-server-demo/server.py"
            ]
        }
    }
}
```

**Note**: After executing `uv run mcp install server.py`, the command is set as just "uv" in the configuration, but it requires an absolute path to work properly.

#### Using Claude Desktop with MCP

##### Checking Server Status

> The Claude.app interface provides basic server status information:
> 1. Click the connect icon to view:
>     - Connected servers
>     - Available prompts and resources
> 2. Click the hammer icon to view:
>     - Tools made available to the mode

- [Debugging in Claude Desktop](https://modelcontextprotocol.io/docs/tools/debugging#debugging-in-claude-desktop)

Type in Claude Desktop:
> Please use the add tool to sum 10 and 20

##### Using the `get_greeting` Resource

**Note**: There appears to be an issue retrieving resources from Claude Desktop. This section needs further investigation.

## FAQ

### How does Claude Desktop select which MCP server tools to use?

- Claude connects to all configured MCP servers but selects tools based on the content of the user's query
- If the question content is not suitable for using tools, the tools will not be used

## References

- [Model Context Protocol > Building MCP with LLMs](https://modelcontextprotocol.io/tutorials/building-mcp-with-llms)
- [Model Context Protocol > Debugging](https://modelcontextprotocol.io/docs/tools/debugging)
- [Model Context Protocol > Inspector](https://modelcontextprotocol.io/docs/tools/inspector)
