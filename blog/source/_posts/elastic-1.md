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









