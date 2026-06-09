# Kibana Dev Tools Console

The interactive REPL for the Elasticsearch REST API: **Management → Dev Tools → Console**.

## Keyboard Shortcuts

| Action | Shortcut |
|---|---|
| Send request | `Ctrl/Cmd + Enter` |
| Auto-indent / format request | `Ctrl/Cmd + I` |
| Open autocomplete | `Ctrl + Space` |
| Jump to next/previous request | `Ctrl/Cmd + ↑ / ↓` |
| Open Console from anywhere in Kibana | `Ctrl/Cmd + /` |

## Editor Tips

- Write **multiple requests** in one editor; the ▶ button runs the request your cursor is in. Select several requests to run them in sequence.
- `#` starts a **comment** line.
- Triple quotes `"""..."""` embed multi-line strings — ideal for Painless scripts:

  ```
  POST /my-index/_update/1
  {
    "script": {
      "source": """
        ctx._source.views += params.n;
        ctx._source.updated = true;
      """,
      "params": { "n": 1 }
    }
  }
  ```

- Click the gutter arrows to **collapse/expand** JSON blocks.
- The wrench icon next to a request offers **Copy as cURL** — handy for moving a request into scripts.

## Console Features

| Feature | Where | Use |
|---|---|---|
| **History** | History tab | Replay past requests |
| **Config** | Config tab | Editor settings, autocomplete behavior |
| **Variables** | Bottom-left `</> Variables` | Define `${myVar}` and reuse across requests |
| **Export/Import requests** | Top-right | Share request collections with teammates |
| **Search Profiler** | Dev Tools tab | Visualize where query time is spent |
| **Grok Debugger** | Dev Tools tab | Test grok patterns against sample logs |
| **Painless Lab** | Dev Tools tab (beta) | Prototype Painless scripts interactively |

## Response Pane

- Status badge (e.g., `200 - OK`) and round-trip time appear bottom-right.
- Multiple executed requests produce stacked responses in order.

## Status Codes

| Code | Meaning |
|---|---|
| `200 OK` | Success |
| `201 Created` | Document created |
| `400 Bad Request` | Malformed JSON or query |
| `404 Not Found` | Index/document doesn't exist |
| `409 Conflict` | Version conflict, or `_create` on an existing ID |
| `429 Too Many Requests` | Cluster under pressure — back off and retry |
