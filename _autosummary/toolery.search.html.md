# toolery.search

Search backends for toolery.

A *search backend* is any callable `(query, cards, *, limit) -> [(card, score), ...]`.
The default, [`lexical_search()`](#toolery.search.lexical_search), is a zero-dependency token-overlap scorer so
that `toolery` works out of the box with no models, services, or API keys. A
semantic backend built on the `ir` retrieval substrate plugs into the same seam
(see the `search_backend` argument of [`toolery.catalog.Catalog`](toolery.html.md#toolery.Catalog)).

### Functions

| [`lexical_search`](#toolery.search.lexical_search)(query, cards, \*[, limit])   | Rank `cards` against `query` by token overlap, with name/phrase boosts.   |
|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|

### Classes

| [`SearchBackend`](#toolery.search.SearchBackend)(\*args, \*\*kwargs)   | Callable protocol every search backend satisfies.   |
|--------------------------------------------------------------------------------------|-----------------------------------------------------|

### *class* toolery.search.SearchBackend(\*args, \*\*kwargs)

Bases: [`Protocol`](https://docs.python.org/3/library/typing.html#typing.Protocol)

Callable protocol every search backend satisfies.

### toolery.search.lexical_search(query, cards, , limit=10)

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
