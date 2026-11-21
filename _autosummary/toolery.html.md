# toolery

toolery — a searchable catalog over any corpus of tools, skills, agents, and components.

Point `toolery` at a collection of heterogeneous assets — Claude skills, agent
specs, MCP tools, docs, or packages — and get one searchable catalog: ask  *“what do
I already have for X?”* and get a ranked answer, not fifty schemas.

Everything is projected onto a uniform [`Card`](#toolery.Card); a pluggable `search_backend`
answers queries (the default is a zero-dependency lexical scorer, so it works out of
the box; a semantic backend built on the `ir` retrieval substrate drops into the
same seam).

```pycon
>>> import toolery
>>> cat = toolery.catalog([
...     toolery.Card('a', 'tool', 'csvdedupe', 'remove duplicate rows from a csv'),
...     toolery.Card('b', 'tool', 'pdfread', 'extract text from pdf files'),
... ])
>>> [c.name for c, score in cat.search('deduplicate csv rows')]
['csvdedupe']
```

### Functions

| [`catalog`](#toolery.catalog)(\*sources[, search_backend])      | Build a [`Catalog`](#toolery.Catalog) from one or more sources.   |
|--------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| [`folder`](#toolery.folder)(root, \*[, kind, pattern])         | Harvest markdown/text documents under `root` into cards.                                     |
| [`skills`](#toolery.skills)(root, \*[, kind])                  | Harvest Claude Agent Skills (`SKILL.md` files) under `root` into cards.                      |
| [`agents`](#toolery.agents)(root, \*[, kind])                  | Harvest Claude Code subagent specs (`.claude/agents/*.md`) under `root`.                     |
| [`packages`](#toolery.packages)(root, \*[, kind])                | Harvest Python packages (dirs with a `pyproject.toml`) under `root`.                         |
| [`mcp`](#toolery.mcp)(source, \*[, kind])                   | Harvest configured MCP servers from an MCP config (`.mcp.json` or similar).                  |
| [`lexical_search`](#toolery.lexical_search)(query, cards, \*[, limit]) | Rank `cards` against `query` by token overlap, with name/phrase boosts.                      |
| [`make_server`](#toolery.make_server)(catalog, \*[, name])          | Build (but don't run) an MCP server exposing `catalog`'s search as one tool.                 |
| [`search_tool`](#toolery.search_tool)(catalog)                      | Return a `search(query, limit=10)` function (an MCP-shaped tool) bound to `catalog`.         |

### Classes

| [`Card`](#toolery.Card)(id, kind, name[, description, tags, ...])   | A normalized record for one catalogued asset.                                                      |
|---------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| [`Catalog`](#toolery.Catalog)([cards, search_backend])                 | A searchable catalog of [`Card`](toolery.base.html.md#toolery.base.Card) s. |
| [`SearchBackend`](#toolery.SearchBackend)(\*args, \*\*kwargs)                | Callable protocol every search backend satisfies.                                                  |
| [`IrBackend`](#toolery.IrBackend)(\*[, name, embedder, mode, ...])       | Semantic search backend delegating to the `ir` retrieval substrate.                                |
| [`IrFederatedBackend`](#toolery.IrFederatedBackend)(\*[, name, embedder, ...])    | Federated semantic backend: one `ir` corpus **per card kind**, searched together.                  |

### *class* toolery.Card(id, kind, name, description='', tags=(), source_uri=None, content_ref=None, extra=<factory>)

Bases: [`object`](https://docs.python.org/3/builtins/functions.html#object)

A normalized record for one catalogued asset.

The card is the *cheap* unit of discovery: small, uniform, and searchable.
The full asset (a file, a package, a tool schema) is referenced by
`content_ref`/`source_uri` and loaded on demand.

```pycon
>>> c = Card(id='x', kind='skill', name='CSV deduper',
...          description='remove duplicate rows from a csv', tags=('data',))
>>> c.text
'CSV deduper\nremove duplicate rows from a csv\ndata'
>>> c.kind
'skill'
```

#### *property* text *: [str](https://docs.python.org/3/builtins/stdtypes.html#str)*

The searchable text surface of the card (name, description, tags).

#### to_dict()

A plain-dict view of the card (JSON-friendly, `extra` flattened out).

* **Return type:**
  [`dict`](https://docs.python.org/3/builtins/stdtypes.html#dict)

### *class* toolery.Catalog(cards=(), \*, search_backend=<function lexical_search>)

Bases: [`object`](https://docs.python.org/3/builtins/functions.html#object)

A searchable catalog of [`Card`](toolery.base.html.md#toolery.base.Card) s.

```pycon
>>> cat = Catalog([
...     Card('a', 'skill', 'CSV deduper', 'removes duplicate rows from csv'),
...     Card('b', 'skill', 'PDF reader', 'extract text from pdf files'),
... ])
>>> len(cat)
2
>>> [c.name for c, _ in cat.search('deduplicate a csv')]
['CSV deduper']
>>> [c.name for c in cat.by_kind('skill')]
['CSV deduper', 'PDF reader']
```

#### by_kind(kind)

All cards of a given `kind`.

* **Return type:**
  [`list`](https://docs.python.org/3/builtins/stdtypes.html#list)[[`Card`](toolery.base.html.md#toolery.base.Card)]

#### *property* cards *: [list](https://docs.python.org/3/builtins/stdtypes.html#list)[[Card](toolery.base.html.md#toolery.base.Card)]*

All cards in the catalog.

#### *property* kinds *: [dict](https://docs.python.org/3/builtins/stdtypes.html#dict)[[str](https://docs.python.org/3/builtins/stdtypes.html#str), [int](https://docs.python.org/3/builtins/functions.html#int)]*

Mapping of `kind` to the number of cards of that kind.

#### search(query, , limit=10)

Ranked `(card, score)` results for `query` via the search backend.

* **Return type:**
  [`list`](https://docs.python.org/3/builtins/stdtypes.html#list)[[`tuple`](https://docs.python.org/3/builtins/stdtypes.html#tuple)[[`Card`](toolery.base.html.md#toolery.base.Card), [`float`](https://docs.python.org/3/builtins/functions.html#float)]]

### *class* toolery.IrBackend(, name='toolery', embedder='default', mode='dense', persist=False, min_score=0.0)

Bases: [`object`](https://docs.python.org/3/builtins/functions.html#object)

Semantic search backend delegating to the `ir` retrieval substrate.

Satisfies the `(query, cards, *, limit) -> [(card, score), ...]` search-backend
contract, so it is a drop-in for [`toolery.Catalog`](#toolery.Catalog)’s `search_backend`. The
`ir` corpus is built lazily on first search and cached, and rebuilt only when the
cards change. Only hits above `min_score` are returned (precision-favoring).

Args (keyword-only):
: name: corpus name handed to `ir`.
  embedder: `"default"` (MiniLM) for real semantics, `"light"` for a
  <br/>
  > hermetic pure-numpy hashing embedder (no download/network).
  <br/>
  mode: `ir` retrieval mode — `"dense"`, `"lexical"`, or `"hybrid"`.
  persist: if True, use `ir`’s file-backed store (incremental across runs)
  <br/>
  > instead of the default in-memory store.
  <br/>
  min_score: drop hits at or below this similarity (default 0.0).

### *class* toolery.IrFederatedBackend(, name='toolery', embedder='default', mode='dense', min_score=None, max_k=None, persist=False)

Bases: [`object`](https://docs.python.org/3/builtins/functions.html#object)

Federated semantic backend: one `ir` corpus **per card kind**, searched together.

Delegates to `ir.discover([...])`, which gates each per-kind corpus on its own
(raw-score) abstention floor, fuses across corpora with Reciprocal Rank Fusion, then
applies distractor-robust selection — so heterogeneous kinds (skills vs. packages vs.
docs), whose similarity scores live on different scales, compare fairly. This is the
multi-corpus discovery the catalog is designed around.

Same drop-in contract as [`IrBackend`](#toolery.IrBackend). Args (keyword-only):
: name: prefix for the per-kind corpus names.
  embedder: `"default"` (MiniLM) or `"light"` (hermetic hashing).
  mode: `ir` mode — `"dense"` (warning-free), `"lexical"`, or `"hybrid"`.
  min_score: federated abstention floor — `None`, `"auto"`, or a
  <br/>
  > `{corpus_name: float|"auto"|None}` mapping (a bare float is invalid here).
  <br/>
  max_k: cap on committed results (defaults to the per-call `limit`).
  persist: if True, use ir’s file-backed store (incremental across runs via the
  <br/>
  > ledger) instead of the in-memory store.

### *class* toolery.SearchBackend(\*args, \*\*kwargs)

Bases: [`Protocol`](https://docs.python.org/3/library/typing.html#typing.Protocol)

Callable protocol every search backend satisfies.

### toolery.agents(root, , kind='agent')

Harvest Claude Code subagent specs (`.claude/agents/*.md`) under `root`.

Agent specs declare a YAML-frontmatter `name`; `.md` files without one are
skipped. Pass a project root (finds `**/agents/*.md`) or an `agents` dir directly.

### toolery.catalog(\*sources, search_backend=<function lexical_search>)

Build a [`Catalog`](#toolery.Catalog) from one or more sources.

A *source* is a folder path (harvested as markdown), a harvester iterable of
cards (e.g. [`toolery.skills()`](#toolery.skills)), or bare [`Card`](toolery.base.html.md#toolery.base.Card) s.

```pycon
>>> cat = catalog([Card('x', 'tool', 'grep', 'search text with patterns')])
>>> cat.search('pattern search')[0][0].name
'grep'
```

* **Return type:**
  [`Catalog`](#toolery.Catalog)

### toolery.folder(root, , kind='doc', pattern='\*\*/\*.md')

Harvest markdown/text documents under `root` into cards.

The card `id` is the path relative to `root`; `name` and `description`
are read from frontmatter when present, else derived from the first heading
and paragraph. `pattern` is a [`pathlib.Path.glob()`](https://docs.python.org/3/library/pathlib.html#pathlib.Path.glob) pattern (recursive
by default via `**`).

* **Return type:**
  [`Iterator`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)[[`Card`](toolery.base.html.md#toolery.base.Card)]

### toolery.lexical_search(query, cards, , limit=10)

Rank `cards` against `query` by token overlap, with name/phrase boosts.

Precision-favoring: a card is only returned if it shares a query token or
contains the whole query as a substring — so an empty or irrelevant query
yields nothing rather than noise.

```pycon
>>> cards = [
...     Card('a', 'skill', 'CSV deduper', 'removes duplicate rows from csv'),
...     Card('b', 'skill', 'PDF reader', 'extract text from pdf files'),
... ]
>>> [(c.name, round(s, 2)) for c, s in lexical_search('dedupe csv', cards)]
[('CSV deduper', 0.75)]
```

* **Return type:**
  [`list`](https://docs.python.org/3/builtins/stdtypes.html#list)[[`tuple`](https://docs.python.org/3/builtins/stdtypes.html#tuple)[[`Card`](toolery.base.html.md#toolery.base.Card), [`float`](https://docs.python.org/3/builtins/functions.html#float)]]

### toolery.make_server(catalog, , name='toolery')

Build (but don’t run) an MCP server exposing `catalog`’s search as one tool.

### toolery.mcp(source, , kind='mcp')

Harvest configured MCP servers from an MCP config (`.mcp.json` or similar).

`source` may be the config file itself or a directory containing a `.mcp.json`.

### toolery.packages(root, , kind='package')

Harvest Python packages (dirs with a `pyproject.toml`) under `root`.

Scans `root/pyproject.toml` and `root/*/pyproject.toml` (the common
folder-of-packages layout); name/description come from the `[project]` table.

### toolery.search_tool(catalog)

Return a `search(query, limit=10)` function (an MCP-shaped tool) bound to `catalog`.

Its signature and docstring become the MCP tool schema and description.

### toolery.skills(root, , kind='skill')

Harvest Claude Agent Skills (`SKILL.md` files) under `root` into cards.

* **Return type:**
  [`Iterator`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)[[`Card`](toolery.base.html.md#toolery.base.Card)]

### Modules

| [`contrib`](toolery.contrib.html.md#module-toolery.contrib)       | Convenience presets: build a catalog over your usual asset locations in one call.                                                  |
|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [`base`](toolery.base.html.md#module-toolery.base)             | Core data model for toolery: the normalized asset `Card`.                                                                          |
| [`cli`](toolery.cli.html.md#module-toolery.cli)               | Command-line interface for toolery (`cw` dispatch).                                                                                |
| [`harvest`](toolery.harvest.html.md#module-toolery.harvest)       | Harvesters: turn a source of raw assets into normalized [`Card`](toolery.base.html.md#toolery.base.Card) s. |
| [`ir_backend`](toolery.ir_backend.html.md#module-toolery.ir_backend) | An `ir`-backed semantic search backend for toolery.                                                                                |
| [`mcp_server`](toolery.mcp_server.html.md#module-toolery.mcp_server) | Expose a toolery catalog's search as an MCP server — one search tool for agents.                                                   |
| [`search`](toolery.search.html.md#module-toolery.search)         | Search backends for toolery.                                                                                                       |
