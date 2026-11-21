# toolery.contrib

Convenience presets: build a catalog over your usual asset locations in one call.

General for any user — pass roots explicitly, or keep them in a local config file. To pull
in a custom source (for example a private skill index), list a `"module:function"`
reference in the config’s `[harvesters] refs`; toolery imports it lazily. Nothing here
hardcodes a personal path, so it stays useful for everyone while you dogfood it on your own
ecosystem: your paths live in `~/.config/toolery/sources.toml` on your machine, not in code.

### Functions

| [`claude_agents`](#toolery.contrib.claude_agents)(\*roots[, home, include_home])   | Subagent cards from each of `roots` and (by default) `~/.claude`.         |
|-------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| [`claude_skills`](#toolery.contrib.claude_skills)(\*roots[, home, include_home])   | Skill cards from each of `roots` and (by default) `~/.claude`.            |
| [`everything`](#toolery.contrib.everything)(\*[, home, claude_roots, ...])      | One catalog over your Claude skills + subagents and your Python packages. |
| [`from_config`](#toolery.contrib.from_config)([path, search_backend])            | Build a catalog from a TOML config of sources.                            |
| [`python_packages`](#toolery.contrib.python_packages)(\*roots)                       | Package cards from each root (a folder containing packages).              |

### toolery.contrib.claude_agents(\*roots, home='~', include_home=True)

Subagent cards from each of `roots` and (by default) `~/.claude`.

* **Return type:**
  [`Iterator`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)[[`Card`](toolery.base.html.md#toolery.base.Card)]

### toolery.contrib.claude_skills(\*roots, home='~', include_home=True)

Skill cards from each of `roots` and (by default) `~/.claude`.

* **Return type:**
  [`Iterator`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)[[`Card`](toolery.base.html.md#toolery.base.Card)]

### toolery.contrib.everything(, home='~', claude_roots=(), package_roots=(), search_backend=None)

One catalog over your Claude skills + subagents and your Python packages.

`claude_roots`/`package_roots` are extra locations to scan; `~/.claude` is
always included for Claude assets (pass `home` to relocate). Supply a
`search_backend` (e.g. `IrBackend()` / `IrFederatedBackend()`) for semantics.

* **Return type:**
  [`Catalog`](toolery.html.md#toolery.Catalog)

### toolery.contrib.from_config(path=None, , search_backend=None)

Build a catalog from a TOML config of sources.

Schema (every section optional):

```default
[claude]
roots = ["~/.claude", "~/work/project"]
[packages]
roots = ["~/proj/mine", "~/proj/theirs"]
[harvesters]
refs = ["mymod:my_cards"]   # "module:function" -> iterable of Card or card dicts
```

Your personal paths live in this file on your machine, not in any committed code.
Reading the config needs Python 3.11+ (`tomllib`).

* **Return type:**
  [`Catalog`](toolery.html.md#toolery.Catalog)

### toolery.contrib.python_packages(\*roots)

Package cards from each root (a folder containing packages).

* **Return type:**
  [`Iterator`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)[[`Card`](toolery.base.html.md#toolery.base.Card)]
