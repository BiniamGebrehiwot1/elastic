# Index Management

## Index Lifecycle Basics

```
PUT /my-index                          # Create (default settings)
DELETE /my-index                       # Delete — irreversible!
GET /my-index                          # Settings + mappings + aliases
HEAD /my-index                         # Exists? (200/404)
POST /my-index/_close                  # Close (frees resources, blocks ops)
POST /my-index/_open                   # Reopen
GET /my-index/_count                   # Doc count
GET /my-index/_stats                   # Detailed index stats
```

## Create with Settings & Mappings

```
PUT /my-index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "refresh_interval": "1s"
  },
  "mappings": {
    "properties": {
      "title":      { "type": "text",
                      "fields": { "keyword": { "type": "keyword" } } },
      "tags":       { "type": "keyword" },
      "price":      { "type": "double" },
      "in_stock":   { "type": "boolean" },
      "created_at": { "type": "date" },
      "location":   { "type": "geo_point" }
    }
  }
}
```

> The `text` + `.keyword` multi-field pattern gives you full-text search **and** exact matching/aggregation on the same field.

## Common Field Types

| Type | Use |
|---|---|
| `text` | Full-text search (analyzed) |
| `keyword` | Exact match, sorting, aggregations |
| `long`, `integer`, `double`, `float` | Numbers |
| `date` | Dates (configurable `format`) |
| `boolean` | true/false |
| `ip` | IP addresses (supports CIDR queries) |
| `geo_point` | Lat/lon |
| `object` | Nested JSON (flattened internally) |
| `nested` | Arrays of objects queried independently |
| `dense_vector` | Vectors for kNN/semantic search |

## Mappings

```
GET /my-index/_mapping

PUT /my-index/_mapping                 # Add NEW fields only
{
  "properties": {
    "subtitle": { "type": "text" }
  }
}
```

**Existing field types cannot be changed.** To change one: create a new index with the right mapping, then `_reindex`, then swap aliases.

## Aliases

```
GET _alias                             # All aliases
GET /my-index/_alias                   # Aliases for one index

POST /_aliases                         # Atomic swap — zero-downtime reindex pattern
{
  "actions": [
    { "add":    { "index": "my-index-v2", "alias": "my-index" } },
    { "remove": { "index": "my-index-v1", "alias": "my-index" } }
  ]
}
```

> Point your application at the **alias**, never the physical index. Reindexing then becomes invisible to clients.

## Reindex

```
POST _reindex
{
  "source": { "index": "old-index" },
  "dest":   { "index": "new-index" }
}

POST _reindex                          # Subset + transform
{
  "source": {
    "index": "old-index",
    "query": { "range": { "date": { "gte": "2025-01-01" } } }
  },
  "dest": { "index": "new-index" },
  "script": { "source": "ctx._source.migrated = true" }
}
```

Add `?wait_for_completion=false` for big reindexes; monitor with `GET _tasks/<id>`.

## Index Templates

Auto-apply settings/mappings to new indices matching a pattern:

```
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "priority": 100,
  "template": {
    "settings": { "number_of_shards": 1 },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "message":    { "type": "text" }
      }
    }
  }
}

GET _index_template/logs-template
DELETE _index_template/logs-template
```

## ILM (Index Lifecycle Management)

```
GET _ilm/policy                        # All policies
GET /my-index/_ilm/explain             # Where is this index in its lifecycle?

PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot":    { "actions": { "rollover": { "max_size": "50gb", "max_age": "7d" } } },
      "warm":   { "min_age": "30d", "actions": { "shrink": { "number_of_shards": 1 } } },
      "delete": { "min_age": "90d", "actions": { "delete": {} } }
    }
  }
}
```
