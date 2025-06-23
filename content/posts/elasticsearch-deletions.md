+++
title = "How Elasticsearch Handles Deletions (and Why Our 30TB Purge Didn’t Break Anything)"
date = "2025-06-23T21:00:00+01:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Akshay Katyal"
authorTwitter = "mrdhat" #do not include @
cover = ""
tags = ["elasticsearch", "databases"]
keywords = ["elasticsearch", "databases"]
description = ""
showFullContent = false
readingTime = true
hideComments = true
draft=false
+++

I recently joined a new team & my onboarding task was to optimize storage in one of our Elasticsearch clusters which held 100s of terabytes of data. One of the first things we needed to figure out was whether the system could survive a massive deletion during peak traffic. We expected indexing to clog up and search latencies to spike. What actually happened was more interesting: the cluster remained stable, but there was clearly a lot happening behind the scenes.

That led me to a deeper understanding of how Elasticsearch handles deletions—and why it works as well as it does at scale.

## Deletions Aren’t What You Think

When a document is deleted, it doesn’t vanish immediately. Instead, Elasticsearch (and the Lucene engine beneath it) uses a two phase strategy:

1. **Logical deletion** — the document is marked as deleted and the change is written to the [translog](https://www.elastic.co/docs/reference/elasticsearch/index-settings/translog#index-modules-translog).
2. **Physical deletion** — the document is eventually removed during a segment merge.

This design lets the system respond quickly while deferring the expensive cleanup work to background tasks. Elasticsearch stores data in immutable segments. Since a segment can't be changed directly, deletions are handled using delete markers.

Here's what happens when a document is deleted:

1. A delete request is written to the translog.
2. A delete marker is created in memory.
3. On the next refresh, the marker is persisted to disk.
4. Search queries check these markers to filter out deleted documents.

## Why Our Cluster Didn't Break A Sweat

We deleted 30TB worth of documents and... everything held up. That’s because the initial operation wasn’t physically moving or deleting much but was just tagging documents as deleted.

That said, we did see:

- Increased memory usage from tracking millions of delete markers.
- "Merge pressure" building up as the system prepared to clean things up later.

The user facing performance stayed consistent but the system was definitely working harder under the hood.

## Real Cleanup Happens Later

The actual deletion—reclaiming disk space and removing data—happens during segment merges.

During a merge:

1. Smaller segments are combined into larger ones.
2. Deleted documents are left out.
3. Old segments are discarded.
4. Disk space is reclaimed.

These merges run in the background and they take time. That’s why deletions don’t immediately reduce the disk usage.

## What This Taught Me

This experience highlighted how Elasticsearch is built:

- Speed over instant cleanup
- Availability over simplicity
- Scale over direct control

> Deletions aren’t free, they’re just deferred.

## Appendix

### Metrics Worth Tracking

In case this is something you are doing right now, here are some metrics you can watch:

```bash
# Check deleted vs total documents
GET _cat/indices?v&h=index,docs.count,docs.deleted&s=docs.deleted:desc

# See segment layout
GET /your_index/_segments

# Monitor ongoing merges
GET _cat/nodes?v&h=name,merges.current,merges.total_time&s=merges.current:desc
```
