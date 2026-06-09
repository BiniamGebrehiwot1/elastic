# Cluster Operations & Diagnostics

## Health & Status

```
GET /                              # Version, cluster name — am I connected?
GET _cluster/health                # green / yellow / red
GET _cluster/health?level=indices  # Per-index health
GET _cluster/stats                 # Cluster-wide stats
GET _cluster/settings              # Persistent + transient settings
GET _nodes/stats                   # Per-node stats (heap, GC, threads)
GET _nodes/hot_threads             # What's burning CPU right now?
```

**Health colors:**

| Color | Meaning |
|---|---|
| green | All primary and replica shards allocated |
| yellow | All primaries allocated, some replicas aren't (single-node clusters are always yellow if replicas > 0) |
| red | At least one primary shard unallocated — data unavailable |

## `_cat` APIs (human-readable)

```
GET _cat/health?v
GET _cat/nodes?v&h=name,heap.percent,cpu,load_1m,disk.used_percent
GET _cat/indices?v&s=store.size:desc        # Biggest indices first
GET _cat/shards?v&s=index
GET _cat/allocation?v                        # Disk usage per node
GET _cat/pending_tasks?v
GET _cat/thread_pool?v&h=node_name,name,active,queue,rejected
GET _cat/recovery?v&active_only=true         # Shard recoveries in flight
GET _cat/aliases?v
GET _cat/templates?v
```

Modifiers: `?v` headers · `&s=col` sort · `&h=col1,col2` choose columns · `&format=json`.

## Why Won't My Shard Allocate?

```
GET _cluster/allocation/explain              # Explains the first unassigned shard
GET _cluster/allocation/explain
{
  "index": "my-index",
  "shard": 0,
  "primary": true
}
```

Common causes: disk watermarks exceeded, allocation filtering rules, max shards per node, node left the cluster.

## Tasks

```
GET _tasks                                    # All running tasks
GET _tasks?actions=*reindex*&detailed=true    # Find that long reindex
GET _tasks/<task_id>                          # Status of one task
POST _tasks/<task_id>/_cancel                 # Cancel it
```

## Snapshots (backup/restore)

```
GET _snapshot                                 # Registered repositories
GET _snapshot/my_repo/_all                    # All snapshots in repo
PUT _snapshot/my_repo/snap_1?wait_for_completion=false
{ "indices": "my-index", "include_global_state": false }
POST _snapshot/my_repo/snap_1/_restore
{ "indices": "my-index", "rename_pattern": "(.+)", "rename_replacement": "restored_$1" }
GET _snapshot/my_repo/snap_1/_status
```

## Quick Triage Runbook

1. `GET _cluster/health` — what color, how many unassigned shards?
2. `GET _cat/nodes?v` — are all expected nodes present? Heap/CPU sane?
3. `GET _cat/allocation?v` — anyone near disk watermarks (~85%/90%)?
4. `GET _cluster/allocation/explain` — why are shards unassigned?
5. `GET _cat/thread_pool?v` — rejections climbing? (write/search pressure)
6. `GET _nodes/hot_threads` — what's actually consuming CPU?

## Useful Settings Checks

```
GET _cluster/settings?include_defaults=true&filter_path=**.watermark*
GET /my-index/_settings
PUT _cluster/settings                         # e.g., temporarily disable allocation
{ "persistent": { "cluster.routing.allocation.enable": "primaries" } }
```

> Re-enable with `"all"` after maintenance — forgetting this is a classic outage cause.
