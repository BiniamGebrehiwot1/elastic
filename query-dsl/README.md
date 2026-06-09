# Query DSL

Search requests go to `GET /<index>/_search` with a JSON body.

```
GET /my-index/_search
{
  "query": { "match": { "title": "elastic search" } }
}
```

## Query Types

### Full-text (analyzed)

```
{ "match":         { "title": "quick fox" } }                  # OR of terms by default
{ "match":         { "title": { "query": "quick fox", "operator": "and" } } }
{ "match_phrase":  { "title": "quick fox" } }                  # Terms in order
{ "match_phrase":  { "title": { "query": "quick fox", "slop": 2 } } }
{ "multi_match":   { "query": "fox", "fields": ["title^3", "body"] } }
{ "query_string":  { "query": "title:(quick AND fox)" } }      # Lucene syntax (power users)
```

### Term-level (exact, not analyzed)

```
{ "term":     { "status": "active" } }                # Exact value (use keyword fields)
{ "terms":    { "status": ["active", "pending"] } }
{ "range":    { "price": { "gte": 10, "lte": 50 } } }
{ "range":    { "date": { "gte": "now-7d/d", "lt": "now/d" } } }
{ "exists":   { "field": "email" } }
{ "prefix":   { "name": "uni" } }
{ "wildcard": { "name": "uni*ed" } }                  # Avoid leading wildcards
{ "regexp":   { "name": "un.+ed" } }
{ "ids":      { "values": ["1", "2", "3"] } }
{ "fuzzy":    { "name": { "value": "kingdon", "fuzziness": "AUTO" } } }
```

> Gotcha: `term` against a `text` field rarely matches, because the indexed tokens are analyzed (lowercased, split). Use `term` on `keyword` fields, `match` on `text` fields.

## Boolean Composition

```
GET /my-index/_search
{
  "query": {
    "bool": {
      "must":     [ { "match": { "title": "elastic" } } ],
      "filter":   [ { "term":  { "status": "published" } },
                    { "range": { "date": { "gte": "now-30d/d" } } } ],
      "must_not": [ { "term":  { "tags": "draft" } } ],
      "should":   [ { "match": { "summary": "search" } } ],
      "minimum_should_match": 1
    }
  }
}
```

| Clause | Scoring | Semantics |
|---|---|---|
| `must` | Yes | AND |
| `filter` | No (cacheable, faster) | AND |
| `should` | Yes | OR / boost |
| `must_not` | No | NOT |

**Rule of thumb:** anything that's a yes/no condition (status, dates, IDs) belongs in `filter`.

## Pagination, Sorting, Source Filtering

```
GET /my-index/_search
{
  "from": 0,
  "size": 20,
  "sort": [ { "created_at": "desc" }, "_score" ],
  "_source": ["title", "price"],
  "query": { "match_all": {} }
}
```

- `from + size` is capped at 10,000 by default. For deep pagination use `search_after`:

```
GET /my-index/_search
{
  "size": 20,
  "sort": [ { "created_at": "desc" }, { "_id": "asc" } ],
  "search_after": ["2026-01-15T10:00:00Z", "doc-942"]
}
```

## Extras

```
# Highlighting
{
  "query": { "match": { "body": "console" } },
  "highlight": { "fields": { "body": {} } }
}

# Count only
GET /my-index/_count
{ "query": { "term": { "status": "active" } } }

# Multiple / wildcard indices
GET /index-a,index-b/_search
GET /logs-*/_search

# Quick URI search (debugging only)
GET /my-index/_search?q=title:elastic
```

## Date Math

| Expression | Meaning |
|---|---|
| `now` | Current time |
| `now-1d` | 24 hours ago |
| `now-7d/d` | 7 days ago, rounded to start of day |
| `now/M` | Start of current month |
| `2026-01-01\|\|+1M/d` | Anchored date math |

## Debugging Relevance

```
GET /my-index/_explain/1
{ "query": { "match": { "title": "elastic" } } }

GET /my-index/_validate/query?explain=true
{ "query": { "match": { "title": "elastic" } } }

GET /my-index/_search?profile=true
{ "query": { "match": { "title": "elastic" } } }
```
