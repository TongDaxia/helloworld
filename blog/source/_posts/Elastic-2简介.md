---
title: Elastic-1集群内的原理
date: 2019-05-29 15:42:00
tags: [java,分布式,Elastic]
---

##### 集群基础

一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

<!--more-->

当一个节点被选举成为 *主* 节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到 *集群中的任何节点* ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。



##### 集群健康

Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是 *集群健康* ， 它在 `status` 字段中展示为 `green` 、 `yellow` 或者 `red` 。

```js
GET /_cluster/health
```



在一个不包含任何索引的空集群中，它将会有一个类似于如下所示的返回内容：

```js
{
   "cluster_name":          "elasticsearch",
   "status":                "green", 
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```

`status` 字段是我们最关心的。

`status` 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：

- `green`

  所有的主分片和副本分片都正常运行。

- `yellow`

  所有的主分片都正常运行，但不是所有的副本分片都正常运行。

- `red`

  有主分片没能正常运行。

##### 索引

我们往 Elasticsearch 添加数据时需要用到 *索引* —— 保存相关数据的地方。 索引实际上是指向一个或者多个物理 *分片* 的 *逻辑命名空间* 。

一个 **分片** 是一个底层的 **工作单元** ，它仅保存了 全部数据中的一部分。 在[`分片内部机制`](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/inside-a-shard.html)中，我们将详细介绍分片是如何工作的，而现在我们只需知道一个分片是**一个 Lucene 的实例**，以及它本身就是**一个完整的搜索引擎**。 我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。

Elasticsearch 是利用分片将数据分发到集群内各处的。分片是**数据的容器**，**文档保存在分片内**，分片又被分配到集群内的各个节点里。 当你的集群规模扩大或者缩小时， Elasticsearch 会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。

一个分片可以是 *主* 分片或者 *副本* 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

> 技术上来说，一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档，但是实际最大值还需要参考你的使用场景：包括你使用的硬件， 文档的大小和复杂程度，索引和查询文档的方式以及你期望的响应时长。
>

一个副本分片只是一个主分片的拷贝。 副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。

```js
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

![elas_0202](E:\tyg\study\blog\source\_posts\img\elas_0202.png)

由于只有一个主节点，备份的分片无处可在，此时的健康情况：

```js
{
  "cluster_name": "elasticsearch",
  "status": "yellow", 
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 3,
  "active_shards": 3,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 3, 
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50
}
```



##### 添加故障转移

**启动第二个节点**

为了测试第二个节点启动后的情况，你可以在同一个目录内，完全依照启动第一个节点的方式来启动一个新节点（参考[安装并运行 Elasticsearch](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/running-elasticsearch.html)）。多个节点可以共享同一个目录。

当你在同一台机器上启动了第二个节点时，只要它和第一个节点有同样的 `cluster.name` 配置，它就会自动发现集群并加入到其中。 但是在不同机器上启动节点的时候，为了加入到同一集群，你需要配置一个可连接到的单播主机列表。 详细信息请查看[最好使用单播代替组播](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/important-configuration-changes.html#unicast)

有两个节点后的储存情况：

![elas_0203](E:\tyg\study\blog\source\_posts\img\elas_0203.png)

再增加一个节点：

![elas_0204](E:\tyg\study\blog\source\_posts\img\elas_0204.png)

这样丢失一个节点还可以正常服务。如果此时把备份数提高到2：

```js
PUT /blogs/_settings
{
   "number_of_replicas" : 2
}
```

变成了以下情况：

![elas_0204](E:\tyg\study\blog\source\_posts\img\elas_0205.png)

此时丢失任意两个节点都还能正常服务。

>
> 当然，如果只是在相同节点数目的集群上增加更多的副本分片并不能提高性能，因为每个分片从节点上获得的资源会变少。 你需要增加更多的硬件资源来提升吞吐量。
>
> 但是更多的副本分片数提高了数据冗余量。























参考：