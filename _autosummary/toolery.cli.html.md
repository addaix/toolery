# toolery.cli

Command-line interface for toolery (`cw` dispatch).

Follows the SSOT pattern: functions in `_dispatch_funcs` become subcommands.

### Functions

| [`agents`](#toolery.cli.agents)(root[, query, limit])                    | List (or search) Claude Code subagent specs (.claude/agents/<br/><br/>```<br/>*<br/>```<br/><br/>.md) under ROOT.   |
|--------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| [`discover`](#toolery.cli.discover)(query, root, \*[, limit, kinds, ...])  | Federated semantic search across multiple asset KINDS under ROOT (needs toolery[ir]).                               |
| [`index`](#toolery.cli.index)([config, embedder])                       | Build or refresh the on-disk semantic index for your configured sources.                                            |
| [`main`](#toolery.cli.main)()                                          | Dispatch the toolery CLI subcommands.                                                                               |
| [`mine`](#toolery.cli.mine)(query, \*[, config, limit, semantic, ...]) | Search across YOUR configured ecosystem (~/.config/toolery/sources.toml) for QUERY.                                 |
| [`packages`](#toolery.cli.packages)(root[, query, limit])                  | List (or search) Python packages (dirs with pyproject.toml) under ROOT.                                             |
| [`search`](#toolery.cli.search)(query, source, \*[, kind, limit, ...])   | Search a folder corpus of documents (SOURCE) for QUERY, ranked best-first.                                          |
| [`serve`](#toolery.cli.serve)([config, http, name])                     | Serve your ecosystem's search as an MCP server exposing a single `search` tool.                                     |
| [`skills`](#toolery.cli.skills)(root[, query, limit])                    | List (or, with QUERY, search) Claude Agent Skills (SKILL.md) under ROOT.                                            |

### toolery.cli.agents(root, query='', , limit=10)

List (or search) Claude Code subagent specs (.claude/agents/

```
*
```

.md) under ROOT.

### toolery.cli.discover(query, root, , limit=10, kinds='skill,agent,doc', embedder='default', persist=False)

Federated semantic search across multiple asset KINDS under ROOT (needs toolery[ir]).

Harvests each of KINDS (comma-separated: skill,agent,doc,package,mcp), builds one ir
corpus per kind, and searches them together with RRF fusion + abstention. Example:
`toolery discover "parse a pdf" ~/proj --kinds skill,doc,package`

### toolery.cli.index(config=None, , embedder='default')

Build or refresh the on-disk semantic index for your configured sources.

Warms ir’s persistent index (incremental via the ledger) so later `toolery mine
--semantic --persist` is fast. Needs `pip install 'toolery[ir]'`.

### toolery.cli.main()

Dispatch the toolery CLI subcommands.

`cw.dispatch` *returns* the exit code (argh exited by itself), so the
`SystemExit` here is what makes `toolery no-such-command` exit 2.

### toolery.cli.mine(query, , config=None, limit=10, semantic=False, persist=False)

Search across YOUR configured ecosystem (~/.config/toolery/sources.toml) for QUERY.

Configure sources once (see toolery.contrib.from_config); with –semantic, search via
the ir federated backend (needs `pip install 'toolery[ir]'`); with –persist, reuse
an on-disk index (incremental across runs — warm it first with `toolery index`).

### toolery.cli.packages(root, query='', , limit=10)

List (or search) Python packages (dirs with pyproject.toml) under ROOT.

### toolery.cli.search(query, source, , kind='doc', limit=10, pattern='\*\*/\*.md', semantic=False)

Search a folder corpus of documents (SOURCE) for QUERY, ranked best-first.

With –semantic, use the ir embedding backend (needs `pip install 'toolery[ir]'`).

### Example

`toolery search "dedupe csv" ~/notes`

### toolery.cli.serve(config=None, , http=False, name='toolery')

Serve your ecosystem’s search as an MCP server exposing a single `search` tool.

Builds a catalog from ~/.config/toolery/sources.toml (see toolery.contrib.from_config)
and serves it over stdio (or –http). Point your agent host’s MCP config at this
command so the agent gets one search tool over everything. Needs `toolery[mcp]`.

### toolery.cli.skills(root, query='', , limit=10)

List (or, with QUERY, search) Claude Agent Skills (SKILL.md) under ROOT.
