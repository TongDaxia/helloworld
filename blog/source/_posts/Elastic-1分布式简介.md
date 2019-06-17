---
title: Elastic-1分布式简介
date: 2019-05-29 15:42:00
tags: [java,分布式,Elastic]
---

Elastic集群分布式特性

- 分配文档到不同的容器 或 *分片* 中，文档可以储存在一个或多个节点中
- 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
- 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
- 将集群中任一节点的请求路由到存有相关数据的节点
- 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复

<!--more-->

集群扩容

故障转移

分布式文档存储

分布式检索

分片内部原理







比如：

三个节点组成的集群，分片（shard）数量最好是 3 ，使得每台机器上有一份数据。

elasticsearch-head中的

###### demo-a

size: 30.0Mi (30.0Mi)  原始数据的总量(原始数据+副本的总量)

docs: 209,899 (209,899)  文档的数量（原始数据+副本的总量）



索引信息：

```
{
	"uuid": "15uCDmXOQcawNFlYQqX5-w",
	"primaries": {
		"docs": {
			"count": 209899,
			"deleted": 0
		},
		"store": {
			"size_in_bytes": 31488920
		},
		"indexing": {
			"index_total": 209899,
			"index_time_in_millis": 18974,
			"index_current": 0,
			"index_failed": 0,
			"delete_total": 0,
			"delete_time_in_millis": 0,
			"delete_current": 0,
			"noop_update_total": 0,
			"is_throttled": false,
			"throttle_time_in_millis": 0
		},
		"get": {
			"total": 0,
			"time_in_millis": 0,
			"exists_total": 0,
			"exists_time_in_millis": 0,
			"missing_total": 0,
			"missing_time_in_millis": 0,
			"current": 0
		},
		"search": {
			"open_contexts": 0,
			"query_total": 15,
			"query_time_in_millis": 4,
			"query_current": 0,
			"fetch_total": 5,
			"fetch_time_in_millis": 8,
			"fetch_current": 0,
			"scroll_total": 0,
			"scroll_time_in_millis": 0,
			"scroll_current": 0,
			"suggest_total": 0,
			"suggest_time_in_millis": 0,
			"suggest_current": 0
		},
		"merges": {
			"current": 0,
			"current_docs": 0,
			"current_size_in_bytes": 0,
			"total": 0,
			"total_time_in_millis": 0,
			"total_docs": 0,
			"total_size_in_bytes": 0,
			"total_stopped_time_in_millis": 0,
			"total_throttled_time_in_millis": 0,
			"total_auto_throttle_in_bytes": 62914560
		},
		"refresh": {
			"total": 21,
			"total_time_in_millis": 4611,
			"listeners": 0
		},
		"flush": {
			"total": 3,
			"periodic": 0,
			"total_time_in_millis": 58
		},
		"warmer": {
			"current": 0,
			"total": 12,
			"total_time_in_millis": 0
		},
		"query_cache": {
			"memory_size_in_bytes": 0,
			"total_count": 12,
			"hit_count": 0,
			"miss_count": 12,
			"cache_size": 0,
			"cache_count": 0,
			"evictions": 0
		},
		"fielddata": {
			"memory_size_in_bytes": 0,
			"evictions": 0
		},
		"completion": {
			"size_in_bytes": 0
		},
		"segments": {
			"count": 3,
			"memory_in_bytes": 353343,
			"terms_memory_in_bytes": 344575,
			"stored_fields_memory_in_bytes": 7544,
			"term_vectors_memory_in_bytes": 0,
			"norms_memory_in_bytes": 0,
			"points_memory_in_bytes": 1020,
			"doc_values_memory_in_bytes": 204,
			"index_writer_memory_in_bytes": 0,
			"version_map_memory_in_bytes": 0,
			"fixed_bit_set_memory_in_bytes": 0,
			"max_unsafe_auto_id_timestamp": -1,
			"file_sizes": {}
		},
		"translog": {
			"operations": 209899,
			"size_in_bytes": 41002556,
			"uncommitted_operations": 0,
			"uncommitted_size_in_bytes": 165,
			"earliest_last_modified_age": 0
		},
		"request_cache": {
			"memory_size_in_bytes": 0,
			"evictions": 0,
			"hit_count": 0,
			"miss_count": 0
		},
		"recovery": {
			"current_as_source": 0,
			"current_as_target": 0,
			"throttle_time_in_millis": 0
		}
	},
	"total": {
		"docs": {
			"count": 209899,
			"deleted": 0
		},
		"store": {
			"size_in_bytes": 31488920
		},
		"indexing": {
			"index_total": 209899,
			"index_time_in_millis": 18974,
			"index_current": 0,
			"index_failed": 0,
			"delete_total": 0,
			"delete_time_in_millis": 0,
			"delete_current": 0,
			"noop_update_total": 0,
			"is_throttled": false,
			"throttle_time_in_millis": 0
		},
		"get": {
			"total": 0,
			"time_in_millis": 0,
			"exists_total": 0,
			"exists_time_in_millis": 0,
			"missing_total": 0,
			"missing_time_in_millis": 0,
			"current": 0
		},
		"search": {
			"open_contexts": 0,
			"query_total": 15,
			"query_time_in_millis": 4,
			"query_current": 0,
			"fetch_total": 5,
			"fetch_time_in_millis": 8,
			"fetch_current": 0,
			"scroll_total": 0,
			"scroll_time_in_millis": 0,
			"scroll_current": 0,
			"suggest_total": 0,
			"suggest_time_in_millis": 0,
			"suggest_current": 0
		},
		"merges": {
			"current": 0,
			"current_docs": 0,
			"current_size_in_bytes": 0,
			"total": 0,
			"total_time_in_millis": 0,
			"total_docs": 0,
			"total_size_in_bytes": 0,
			"total_stopped_time_in_millis": 0,
			"total_throttled_time_in_millis": 0,
			"total_auto_throttle_in_bytes": 62914560
		},
		"refresh": {
			"total": 21,
			"total_time_in_millis": 4611,
			"listeners": 0
		},
		"flush": {
			"total": 3,
			"periodic": 0,
			"total_time_in_millis": 58
		},
		"warmer": {
			"current": 0,
			"total": 12,
			"total_time_in_millis": 0
		},
		"query_cache": {
			"memory_size_in_bytes": 0,
			"total_count": 12,
			"hit_count": 0,
			"miss_count": 12,
			"cache_size": 0,
			"cache_count": 0,
			"evictions": 0
		},
		"fielddata": {
			"memory_size_in_bytes": 0,
			"evictions": 0
		},
		"completion": {
			"size_in_bytes": 0
		},
		"segments": {
			"count": 3,
			"memory_in_bytes": 353343,
			"terms_memory_in_bytes": 344575,
			"stored_fields_memory_in_bytes": 7544,
			"term_vectors_memory_in_bytes": 0,
			"norms_memory_in_bytes": 0,
			"points_memory_in_bytes": 1020,
			"doc_values_memory_in_bytes": 204,
			"index_writer_memory_in_bytes": 0,
			"version_map_memory_in_bytes": 0,
			"fixed_bit_set_memory_in_bytes": 0,
			"max_unsafe_auto_id_timestamp": -1,
			"file_sizes": {}
		},
		"translog": {
			"operations": 209899,
			"size_in_bytes": 41002556,
			"uncommitted_operations": 0,
			"uncommitted_size_in_bytes": 165,
			"earliest_last_modified_age": 0
		},
		"request_cache": {
			"memory_size_in_bytes": 0,
			"evictions": 0,
			"hit_count": 0,
			"miss_count": 0
		},
		"recovery": {
			"current_as_source": 0,
			"current_as_target": 0,
			"throttle_time_in_millis": 0
		}
	}
}
```



参考：

<https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html>