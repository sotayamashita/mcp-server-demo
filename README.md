# mcp-server-demo

## Prerequisites

- [uv](https://docs.astral.sh/uv/getting-started/installation/) - _An extremely fast Python package and project manager, written in Rust._

## MCP Server Creation Guide

This content is the almost same as the [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk/blob/c2ca8e03e046908935d089a2ceed4e80b0c29a24/README.md) README, but I've added supplementary notes for parts where I got stuck or details that were omitted in the original.

### 1. Environment Setup

```bash
uv init mcp-server-demo
cd mcp-server-demo

# Specify Python version
echo "3.12" > .python-version

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

### 3. Debugging the MCP Server with MCP Inspector

#### Launch MCP Inspector

```bash
uv run mcp dev server.py

Starting MCP inspector...
⚙️ Proxy server listening on port 6277
```

#### Verify the `add` tool is working

1. Open the MCP Inspector
2. Click "Connect"
3. Click the "Tools" tab
4. Click "List Tools" inside the "Tools" tab

#### Verify the `get_greeting` resource is working

1. Open the MCP Inspecter
2. Click "Connect"
2. Click "Resources" Tab
3. Click "List Templates"
4. Find "get_greeting"
5. Type the name e.g. John
6. Click "Read Resources"

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

### 4. Debugging the MCP Server with Claude Desktop

```bash
# Install the Demo in Claude App
uv run mcp install server.py

[/dd/MM/YY HH:MM:SS] INFO     Added server 'Demo' to Claude config            claude.py:129
                     INFO     Successfully installed Demo in Claude app          cli.py:467

# For macOS/Linux
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Example:
# Note: After exectuting `uv run mcp install server.py`, the command is just "uv" directly, but it won't work without an absolute path
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

#### [Debugging in Claude Desktop](https://modelcontextprotocol.io/docs/tools/debugging#debugging-in-claude-desktop)

> The Claude.app interface provides basic server status information:
> 1. Click the connect icon to view:
>     - Connected servers
>     - Available prompts and resources
> 2. Click the hammer icon to view:
>     - Tools made available to the mode

#### Use `add` tool on Claude Desktop

Type the following text:

> Please use the add tool to sum 10 and 20

#### Use `get_greeting` resource on Claude Desktop

Type the following text:

TODO: I cannot get the resource from Claude Desktop

## FAQ

### How does Claude Desktop select which MCP server tools to use?

- The MCP client basically connects to all provided servers, but which tools it uses depends on the content of the question
- If the question content is not suitable for using tools, the tools will not be used


## References

- [Model Context Protocol > Building MCP with LLMs](https://modelcontextprotocol.io/tutorials/building-mcp-with-llms)
- [Model Context Protocol > Debugging](https://modelcontextprotocol.io/docs/tools/debugging)
- [Model Context Protocol > Inspector](https://modelcontextprotocol.io/docs/tools/inspector)
