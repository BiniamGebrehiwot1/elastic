# Text Analysis

How Elasticsearch turns text into searchable tokens. Analysis happens at **index time** (storing docs) and **query time** (analyzing your search terms) — both must align for matches to work.

## The `_analyze` API

Your best friend for "why doesn't my search match?":

```
GET _analyze
{
  "text": "United Kingdom",
  "analyzer": "standard"
}
```

→ tokens `united` (pos 0) and `kingdom` (pos 1), lowercased and split.

```
GET /my-index/_analyze            # Test against a real field's configured analyzer
{
  "field": "title",
  "text": "United Kingdom"
}

GET _analyze                      # Build a chain ad hoc
{
  "tokenizer": "standard",
  "filter": ["lowercase", "asciifolding"],
  "text": "Café Déjà Vu"
}
```

## Anatomy of an Analyzer

```
character filters → tokenizer → token filters
```

| Stage | Examples |
|---|---|
| Char filters | `html_strip`, `mapping`, `pattern_replace` |
| Tokenizer | `standard`, `whitespace`, `keyword`, `ngram`, `edge_ngram`, `pattern` |
| Token filters | `lowercase`, `stop`, `stemmer`, `synonym`, `asciifolding`, `trim` |

## Built-in Analyzers

| Analyzer | Behavior |
|---|---|
| `standard` (default) | Unicode word splitting + lowercase |
| `simple` | Split on non-letters + lowercase |
| `whitespace` | Split on whitespace only, no lowercasing |
| `keyword` | No-op — whole input is one token |
| `stop` | Like `simple` + removes stopwords |
| `pattern` | Regex-based splitting |
| `english`, `french`, ... | Language-aware: stemming + stopwords |

Compare:

```
GET _analyze
{ "text": "The QUICK brown foxes", "analyzer": "english" }
# → the stemmed tokens: quick, brown, fox
```

## Custom Analyzers

Defined in index settings, referenced in mappings:

```
PUT /my-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonyms": {
          "type": "synonym",
          "synonyms": ["uk, united kingdom, great britain"]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "asciifolding", "my_synonyms"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": { "type": "text", "analyzer": "my_analyzer" }
    }
  }
}
```

Use `"analyzer"` for index time and optionally `"search_analyzer"` for a different query-time analyzer (common with `edge_ngram` autocomplete: ngram at index time, standard at search time).

## Autocomplete Pattern (edge_ngram)

```
"tokenizer": {
  "autocomplete_tokenizer": {
    "type": "edge_ngram",
    "min_gram": 2,
    "max_gram": 15,
    "token_chars": ["letter", "digit"]
  }
}
```

## Debugging Checklist

1. `GET /my-index/_mapping` — what analyzer does the field actually use?
2. `GET /my-index/_analyze` with `"field"` — what tokens were indexed?
3. Run the same on your **query text** — do the token sets overlap?
4. Remember: `term` queries skip analysis entirely; `match` queries analyze. Mismatched expectations here cause most "no results" mysteries.
