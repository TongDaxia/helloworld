---
title: Elastic 启动出现bug
date: 2019-05-23  
tags: [java,Elastic,分布式] 
---



创建多个Elastic节点的时候，需要拷贝多份，注意端口号不需要配置，会自动占用新的端口号。

但是如果拷贝时携带了  `data`文件夹，就会有冲突，报错如下：

> with the same id but is a different node instance

解决办法：

删除 data 文件夹即可。





