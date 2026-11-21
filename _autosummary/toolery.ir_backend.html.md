# toolery.ir_backend

An `ir`-backed semantic search backend for toolery.

Optional — requires the `ir` package (`pip install 'toolery[ir]'`). It embeds each
card’s text with `ir`’s retrieval substrate and answers queries by vector similarity,
rebuilding the index only when the catalog’s cards change (content-fingerprinted).

The default embedder (a MiniLM sentence-transformer) gives real semantic matching;
`embedder="light"` uses `ir`’s pure-numpy hashing embedder — no model download, no
network — which is what the tests use.

Drop it into any [`toolery.Catalog`](toolery.html.md#toolery.Catalog) in place of the default lexical backend:

```default
from toolery import Catalog
from toolery.ir_backend import IrBackend

cat = Catalog(cards, search_backend=IrBackend())        # MiniLM semantics
cat = Catalog(cards, search_backend=IrBackend(embedder="light"))  # hermetic
```

### Classes

| [`IrBackend`](#toolery.ir_backend.IrBackend)(\*[, name, embedder, mode, ...])    | Semantic search backend delegating to the `ir` retrieval substrate.               |
|------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| [`IrFederatedBackend`](#toolery.ir_backend.IrFederatedBackend)(\*[, name, embedder, ...]) | Federated semantic backend: one `ir` corpus **per card kind**, searched together. |

### *class* toolery.ir_backend.IrBackend(, name='toolery', embedder='default', mode='dense', persist=False, min_score=0.0)

Bases: [`object`](https://docs.python.org/3/builtins/functions.html#object)

Semantic search backend delegating to the `ir` retrieval substrate.

Satisfies the `(query, cards, *, limit) -> [(card, score), ...]` search-backend
contract, so it is a drop-in for [`toolery.Catalog`](toolery.html.md#toolery.Catalog)’s `search_backend`. The
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

### *class* toolery.ir_backend.IrFederatedBackend(, name='toolery', embedder='default', mode='dense', min_score=None, max_k=None, persist=False)

Bases: [`object`](https://docs.python.org/3/builtins/functions.html#object)

Federated semantic backend: one `ir` corpus **per card kind**, searched together.

Delegates to `ir.discover([...])`, which gates each per-kind corpus on its own
(raw-score) abstention floor, fuses across corpora with Reciprocal Rank Fusion, then
applies distractor-robust selection — so heterogeneous kinds (skills vs. packages vs.
docs), whose similarity scores live on different scales, compare fairly. This is the
multi-corpus discovery the catalog is designed around.

Same drop-in contract as [`IrBackend`](#toolery.ir_backend.IrBackend). Args (keyword-only):
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
