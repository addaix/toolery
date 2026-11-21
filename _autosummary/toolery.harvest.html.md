# toolery.harvest

Harvesters: turn a source of raw assets into normalized [`Card`](toolery.base.html.md#toolery.base.Card) s.

A *harvester* is any iterable of `Card` s. Built-in harvesters cover the most
common corpora:

- [`folder()`](#toolery.harvest.folder) — any tree of markdown/text documents (the general case).
- [`skills()`](#toolery.harvest.skills) — Claude Agent Skills (`SKILL.md` with YAML frontmatter).

New asset kinds (agent specs, MCP tool schemas, packages) are added by writing
another generator that yields `Card` s — nothing else in the stack changes.

### Functions

| [`agents`](#toolery.harvest.agents)(root, \*[, kind])          | Harvest Claude Code subagent specs (`.claude/agents/*.md`) under `root`.    |
|------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| [`folder`](#toolery.harvest.folder)(root, \*[, kind, pattern]) | Harvest markdown/text documents under `root` into cards.                    |
| [`mcp`](#toolery.harvest.mcp)(source, \*[, kind])           | Harvest configured MCP servers from an MCP config (`.mcp.json` or similar). |
| [`packages`](#toolery.harvest.packages)(root, \*[, kind])        | Harvest Python packages (dirs with a `pyproject.toml`) under `root`.        |
| [`skills`](#toolery.harvest.skills)(root, \*[, kind])          | Harvest Claude Agent Skills (`SKILL.md` files) under `root` into cards.     |

### toolery.harvest.agents(root, , kind='agent')

Harvest Claude Code subagent specs (`.claude/agents/*.md`) under `root`.

Agent specs declare a YAML-frontmatter `name`; `.md` files without one are
skipped. Pass a project root (finds `**/agents/*.md`) or an `agents` dir directly.

### toolery.harvest.folder(root, , kind='doc', pattern='\*\*/\*.md')

Harvest markdown/text documents under `root` into cards.

The card `id` is the path relative to `root`; `name` and `description`
are read from frontmatter when present, else derived from the first heading
and paragraph. `pattern` is a [`pathlib.Path.glob()`](https://docs.python.org/3/library/pathlib.html#pathlib.Path.glob) pattern (recursive
by default via `**`).

* **Return type:**
  [`Iterator`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)[[`Card`](toolery.base.html.md#toolery.base.Card)]

### toolery.harvest.mcp(source, , kind='mcp')

Harvest configured MCP servers from an MCP config (`.mcp.json` or similar).

`source` may be the config file itself or a directory containing a `.mcp.json`.

### toolery.harvest.packages(root, , kind='package')

Harvest Python packages (dirs with a `pyproject.toml`) under `root`.

Scans `root/pyproject.toml` and `root/*/pyproject.toml` (the common
folder-of-packages layout); name/description come from the `[project]` table.

### toolery.harvest.skills(root, , kind='skill')

Harvest Claude Agent Skills (`SKILL.md` files) under `root` into cards.

* **Return type:**
  [`Iterator`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)[[`Card`](toolery.base.html.md#toolery.base.Card)]
