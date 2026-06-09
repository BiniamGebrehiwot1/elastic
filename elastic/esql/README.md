# ES|QL

Elasticsearch's piped query language. Run it in **Discover** (switch to ES|QL mode), or from the Console:

```
POST _query?format=txt
{
  "query": """
    FROM logs-*
    | WHERE status >= 500
    | STATS errors = COUNT(*) BY host
    | SORT errors DESC
    | LIMIT 10
  """
}
```

`format` can be `txt`, `csv`, `json`, `tsv`, and others.

## Anatomy

A source command, then zero or more processing commands chained with `|`:

```
FROM index-pattern
| <processing command>
| <processing command>
```

## Source Commands

| Command | Example |
|---|---|
| `FROM` | `FROM logs-*` — query indices |
| `ROW` | `ROW a = 1, b = "two"` — produce a literal row (testing) |
| `SHOW INFO` | Deployment version info |

## Core Processing Commands

| Command | Purpose | Example |
|---|---|---|
| `WHERE` | Filter rows | `WHERE status == 404 AND bytes > 1000` |
| `KEEP` | Select columns | `KEEP @timestamp, host, message` |
| `DROP` | Remove columns | `DROP _id` |
| `EVAL` | Compute new columns | `EVAL kb = bytes / 1024` |
| `RENAME` | Rename columns | `RENAME host AS server` |
| `SORT` | Order rows | `SORT @timestamp DESC` |
| `LIMIT` | Cap row count | `LIMIT 100` |
| `STATS ... BY` | Aggregate | `STATS avg(bytes) BY host` |
| `DISSECT` | Parse structured text | `DISSECT message "%{ip} %{verb} %{url}"` |
| `GROK` | Parse with grok patterns | `GROK message "%{IP:ip} %{WORD:verb}"` |
| `ENRICH` | Join enrich policy data | `ENRICH geo_policy ON ip` |
| `MV_EXPAND` | Explode multi-value fields | `MV_EXPAND tags` |

## STATS Examples

```
FROM web-logs
| STATS
    hits = COUNT(*),
    avg_bytes = AVG(bytes),
    p95 = PERCENTILE(duration, 95)
  BY host, status

FROM web-logs
| STATS hits = COUNT(*) BY bucket = BUCKET(@timestamp, 1 hour)
| SORT bucket
```

Common aggregate functions: `COUNT`, `COUNT_DISTINCT`, `AVG`, `SUM`, `MIN`, `MAX`, `MEDIAN`, `PERCENTILE`, `VALUES`, `TOP`.

## Useful Functions

| Category | Functions |
|---|---|
| String | `CONCAT`, `SUBSTRING`, `TO_LOWER`, `TO_UPPER`, `TRIM`, `SPLIT`, `LENGTH`, `STARTS_WITH`, `LIKE` / `RLIKE` (operators) |
| Date | `NOW()`, `DATE_TRUNC`, `DATE_FORMAT`, `DATE_DIFF`, `DATE_EXTRACT` |
| Math | `ROUND`, `ABS`, `CEIL`, `FLOOR`, `LOG`, `POW` |
| Conditional | `CASE(cond1, val1, cond2, val2, default)`, `COALESCE`, `GREATEST`, `LEAST` |
| Type conversion | `TO_STRING`, `TO_INT`, `TO_LONG`, `TO_DOUBLE`, `TO_DATETIME`, `TO_IP` |
| Null handling | `IS NULL`, `IS NOT NULL` |

## Filtering Patterns

```
| WHERE message LIKE "*timeout*"          # Wildcard match
| WHERE message RLIKE ".*error [0-9]+.*"  # Regex match
| WHERE host IN ("web-1", "web-2")
| WHERE @timestamp >= NOW() - 1 hour
| WHERE CIDR_MATCH(client_ip, "10.0.0.0/8")
```

## Full Example

```
FROM web-logs
| WHERE @timestamp >= NOW() - 24 hours AND status >= 400
| EVAL error_class = CASE(status >= 500, "server", "client")
| STATS
    count = COUNT(*),
    unique_ips = COUNT_DISTINCT(client_ip)
  BY error_class, url
| SORT count DESC
| LIMIT 20
```

## ES|QL vs Query DSL

| | ES\|QL | Query DSL |
|---|---|---|
| Style | Piped, SQL-like, readable | Nested JSON |
| Best for | Exploration, analytics, transformations | App search, relevance scoring, complex agg trees |
| Transformations | `EVAL`, `DISSECT`, `GROK` inline | Limited (runtime fields, scripts) |
| Relevance scoring | Limited | Full scoring control |
