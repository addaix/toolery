# toolery.base

Core data model for toolery: the normalized asset `Card`.

Every asset the catalog knows about — a Claude skill, an agent spec, an MCP tool,
a research report, a package — is projected onto one uniform record, the
[`Card`](#toolery.base.Card), so that a single search surface can range over all of them.

### Classes

| [`Card`](#toolery.base.Card)(id, kind, name[, description, tags, ...])   | A normalized record for one catalogued asset.   |
|---------------------------------------------------------------------------------------------------|-------------------------------------------------|

### *class* toolery.base.Card(id, kind, name, description='', tags=(), source_uri=None, content_ref=None, extra=<factory>)

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
