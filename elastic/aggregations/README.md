# Aggregations

Analytics over your documents. Set `"size": 0` when you only want agg results, not hits.

```
GET /my-index/_search
{
  "size": 0,
  "aggs": {
    "by_status": { "terms": { "field": "status", "size": 10 } }
  }
}
```

> Aggregate on `keyword`, numeric, or `date` fields — not raw `text` fields.

## Metric Aggregations

```
"avg_price":    { "avg":         { "field": "price" } }
"total":        { "sum":         { "field": "price" } }
"min_price":    { "min":         { "field": "price" } }
"max_price":    { "max":         { "field": "price" } }
"price_stats":  { "stats":       { "field": "price" } }      # min/max/avg/sum/count in one
"unique_users": { "cardinality": { "field": "user_id" } }    # Approximate distinct count
"latency_pcts": { "percentiles": { "field": "duration", "percents": [50, 95, 99] } }
"n_values":     { "value_count": { "field": "price" } }
"newest":       { "top_hits":    { "size": 1, "sort": [{ "date": "desc" }] } }
```

## Bucket Aggregations

```
# Group by value
"by_category": { "terms": { "field": "category", "size": 20 } }

# Time series
"per_month": {
  "date_histogram": {
    "field": "created_at",
    "calendar_interval": "month"        # or fixed_interval: "30d", "1h"
  }
}

# Numeric ranges
"price_bands": {
  "range": {
    "field": "price",
    "ranges": [
      { "to": 50 },
      { "from": 50, "to": 200 },
      { "from": 200 }
    ]
  }
}

# Date ranges
"recency": {
  "date_range": {
    "field": "created_at",
    "ranges": [
      { "from": "now-7d/d", "key": "this_week" },
      { "to": "now-7d/d",   "key": "older" }
    ]
  }
}

# Numeric histogram
"by_size": { "histogram": { "field": "bytes", "interval": 1024 } }

# Arbitrary filters as buckets
"groups": {
  "filters": {
    "filters": {
      "errors":   { "range": { "status": { "gte": 500 } } },
      "warnings": { "range": { "status": { "gte": 400, "lt": 500 } } }
    }
  }
}
```

## Nesting (sub-aggregations)

Metrics inside buckets, buckets inside buckets:

```
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "per_month": {
      "date_histogram": { "field": "date", "calendar_interval": "month" },
      "aggs": {
        "by_region": {
          "terms": { "field": "region" },
          "aggs": {
            "revenue": { "sum": { "field": "amount" } }
          }
        }
      }
    }
  }
}
```

## Pipeline Aggregations

Operate on the output of other aggs:

```
"aggs": {
  "per_month": {
    "date_histogram": { "field": "date", "calendar_interval": "month" },
    "aggs": {
      "revenue": { "sum": { "field": "amount" } },
      "revenue_change": { "derivative":    { "buckets_path": "revenue" } },
      "rolling_avg":    { "moving_fn":     { "buckets_path": "revenue", "window": 3,
                                             "script": "MovingFunctions.unweightedAvg(values)" } }
    }
  },
  "best_month": { "max_bucket": { "buckets_path": "per_month>revenue" } }
}
```

## Filter + Aggregate Together

`query` filters the doc set first; aggs run on what's left:

```
GET /sales/_search
{
  "size": 0,
  "query": { "range": { "date": { "gte": "now-90d/d" } } },
  "aggs":  { "by_region": { "terms": { "field": "region" } } }
}
```

## Common Pitfalls

- `terms` agg counts are approximate across shards; raise `"shard_size"` if accuracy matters.
- `cardinality` is approximate by design (HyperLogLog++).
- Aggregating on a `text` field throws an error — use its `.keyword` sub-field if the mapping has one: `"field": "category.keyword"`.
