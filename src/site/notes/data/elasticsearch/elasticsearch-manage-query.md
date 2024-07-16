---
{"dg-publish":true,"permalink":"/data/elasticsearch/elasticsearch-manage-query/","noteIcon":"","created":"2024-06-30T00:39:32.602+09:00"}
---



```
// cat
GET _cat/allocation?v
GET _cat/shards?v&index=kubelog-2*
GET _cat/indices?v&index=kubelog-2*
GET /_cat/indices?h=index,creation.date

// nodes
GET /_nodes/stats/indices/search

// cluster
GET /_cluster/settings?include_defaults=true

```
