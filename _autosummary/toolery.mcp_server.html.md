# toolery.mcp_server

Expose a toolery catalog’s search as an MCP server — one search tool for agents.

Optional — needs `pip install 'toolery[mcp]'` (py2mcp). Instead of handing an agent a
schema per asset, this gives it a single `search` tool over your whole catalog: the
“one search tool, not fifty schemas” idea made concrete. Point any MCP host at the
`toolery serve` command (see `toolery.cli`).

### Functions

| [`make_server`](#toolery.mcp_server.make_server)(catalog, \*[, name])      | Build (but don't run) an MCP server exposing `catalog`'s search as one tool.         |
|----------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| [`search_tool`](#toolery.mcp_server.search_tool)(catalog)                  | Return a `search(query, limit=10)` function (an MCP-shaped tool) bound to `catalog`. |
| [`serve`](#toolery.mcp_server.serve)(catalog, \*[, name, transport]) | Serve `catalog`'s search as an MCP server (blocks until the host disconnects).       |

### toolery.mcp_server.make_server(catalog, , name='toolery')

Build (but don’t run) an MCP server exposing `catalog`’s search as one tool.

### toolery.mcp_server.search_tool(catalog)

Return a `search(query, limit=10)` function (an MCP-shaped tool) bound to `catalog`.

Its signature and docstring become the MCP tool schema and description.

### toolery.mcp_server.serve(catalog, , name='toolery', transport='stdio')

Serve `catalog`’s search as an MCP server (blocks until the host disconnects).

* **Return type:**
  [`None`](https://docs.python.org/3/builtins/constants.html#None)
