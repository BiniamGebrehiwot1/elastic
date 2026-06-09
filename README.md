# Elastic Cheat Sheets

Quick references for working with Elasticsearch and Kibana, organized by topic. Most examples are meant to be run from the **Kibana Dev Tools Console** (`Management → Dev Tools → Console`).

## Contents

| Section | What's inside |
|---|---|
| [console/](console/) | Kibana Dev Tools Console — shortcuts, tips, UI features |
| [esql/](esql/) | ES\|QL — the piped query language |
| [query-dsl/](query-dsl/) | Search queries — match, term, bool, ranges, pagination |
| [aggregations/](aggregations/) | Metrics, buckets, date histograms, nested aggs |
| [documents/](documents/) | Document CRUD, bulk operations, update/delete by query |
| [index-management/](index-management/) | Indices, mappings, aliases, templates, reindex |
| [analysis/](analysis/) | Text analysis — `_analyze`, analyzers, tokenizers |
| [cluster-ops/](cluster-ops/) | Cluster health, `_cat` APIs, diagnostics, debugging |

## Conventions

- Examples target **Elasticsearch 8.x**; most also work on 7.x.
- `my-index` is a placeholder — replace with your index name.
- `?v` on `_cat` APIs adds column headers; `&s=` sorts; `&h=` selects columns.

## Quick start

```
GET /                    # Am I connected? Version info
GET _cluster/health      # Is the cluster green?
GET _cat/indices?v       # What indices exist?
```
