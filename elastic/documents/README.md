# Documents — CRUD & Bulk

## Create / Index

```
POST /my-index/_doc                    # Auto-generated ID
{ "title": "Hello" }

PUT /my-index/_doc/1                   # Explicit ID — creates or overwrites
{ "title": "Hello" }

PUT /my-index/_create/1                # Create only — 409 Conflict if ID exists
{ "title": "Hello" }
```

## Read

```
GET /my-index/_doc/1                   # Full doc with metadata
GET /my-index/_source/1                # Just the source
HEAD /my-index/_doc/1                  # Exists? (200/404, no body)

GET /my-index/_mget                    # Multiple by ID
{ "ids": ["1", "2", "3"] }
```

## Update

```
POST /my-index/_update/1               # Partial update (merge fields)
{ "doc": { "title": "Updated" } }

POST /my-index/_update/1               # Upsert: update if exists, else create
{
  "doc": { "views": 1 },
  "doc_as_upsert": true
}

POST /my-index/_update/1               # Scripted update
{
  "script": {
    "source": "ctx._source.views += params.n",
    "params": { "n": 1 }
  }
}
```

## Delete

```
DELETE /my-index/_doc/1
```

## Bulk API

Action line, then (for index/create/update) a source line. **Newline-delimited — no trailing commas, no pretty-printing across lines.**

```
POST _bulk
{ "index":  { "_index": "my-index", "_id": "1" } }
{ "title": "Doc one" }
{ "create": { "_index": "my-index", "_id": "2" } }
{ "title": "Doc two" }
{ "update": { "_index": "my-index", "_id": "1" } }
{ "doc": { "title": "Doc one, revised" } }
{ "delete": { "_index": "my-index", "_id": "3" } }
```

- Check `"errors": true` in the response — bulk is **not** all-or-nothing; inspect per-item results.
- Typical sweet spot: 1,000–5,000 docs or ~5–15 MB per bulk request. Benchmark for your cluster.

## Update / Delete by Query

```
POST /my-index/_update_by_query
{
  "script": { "source": "ctx._source.archived = true" },
  "query":  { "range": { "created_at": { "lt": "now-1y" } } }
}

POST /my-index/_delete_by_query
{ "query": { "term": { "status": "expired" } } }
```

- Add `?conflicts=proceed` to skip version conflicts instead of aborting.
- Long-running? Add `?wait_for_completion=false` and track via `GET _tasks/<task_id>`.

## Refresh & Visibility

New/updated docs become searchable after a **refresh** (default: every 1s).

```
PUT /my-index/_doc/1?refresh=wait_for   # Wait until searchable (good for tests)
PUT /my-index/_doc/1?refresh=true       # Force immediate refresh (avoid in production hot paths)
POST /my-index/_refresh                 # Manual refresh
```

## Optimistic Concurrency

```
GET /my-index/_doc/1                    # Note _seq_no and _primary_term

PUT /my-index/_doc/1?if_seq_no=42&if_primary_term=1
{ "title": "Safe concurrent write" }    # 409 Conflict if doc changed since read
```
