安装插件：

```
elasticsearch-plugin install analysis-icu

```



开启多个

```
bin/elasticsearch -E node.name=node1 -E cluster.name=demo -E path.data=node1_data -d
bin/elasticsearch -E node.name=node2 -E cluster.name=demo -E path.data=node2_data -d
bin/elasticsearch -E node.name=node3 -E cluster.name=demo -E path.data=node3_data -d
```



删除进程

```
ps | grep elasticsearch
kill pid
```



安装Kibana 

直接下载安装。运行时报错——必须使用和Elastic同一个版本才能正常启动运行。



<https://www.elastic.co/cn/downloads/logstash>





使用logstash导入数据

```
E:\tyg\elastic\movielens\logstash.conf

logstash -f E:\tyg\elastic\movielens\logstash.conf   ##不行，失败了

logstash.bat -f  logstash.conf   //失败了

logstash.bat -e 'input { stdin { } } output { stdout {} }'
logstash -e 'input { stdin { } } output { stdout {} }'
```

一些查询命令

```
//查看索引相关信息
GET kibana_sample_data_ecommerce

//查看索引的文档总数
GET kibana_sample_data_ecommerce/_count

//查看前10条文档，了解文档格式
POST kibana_sample_data_ecommerce/_search
{
}

//_cat indices API
//查看indices
GET /_cat/indices/kibana*?v&s=index

//查看状态为绿的索引
GET /_cat/indices?v&health=green

//按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

//查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

//How much memory is used per index?
GET /_cat/indices?v&h=i,tm&s=tm:desc

```







