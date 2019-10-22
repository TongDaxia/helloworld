---
title: Hadoop集群搭建详解
date: 2019-09-11 22:04:56
tags: [Hadoop,数据处理] 
description: Hadoop集群的搭建demo,详细到每一个命令。
---







# 闲话时间

最近在学习数据处理这一块的知识，毕竟现在每个公司都会海量数据的问题。虽然现在流行的技术是什么spark,flink,等等之类的东西，说是Hadoop渐渐的不太流行了，但是我作为初学者还是可以从Hadoop开始的。在这里记录一些结合网上的一些资料，搭建Hadoop集群的经历。



# 准备工作

我是在Windows环境使用VMware Workstation虚拟机平台，搭建CentOS7的系统，然后在CentOS7上搭建Hadoop集群的。Hadoop的版本是 2.7.7 。

由于是开发demo，意思到了就OK，我只打算搞三个虚拟机玩一下，集群中节点的名字的ip对应关系如下：



| IP             | 名称   | 工作职责          |
| -------------- | ------ | ----------------- |
| 192.168.60.129 | master | namenode/datanode |
| 192.168.60.130 | hdp01  | datanode          |
| 192.168.60.131 | hdp02  | datanode          |

CentOS7的搭建相对比较简单，其中需要注意的一点是他的网络设置，我是用的是 NAT连接模式，并且使用的是固定IP。





## 网络配置

比如使用NAT模式的话，首先是设置Windows中的网卡



然后设置VMware的网络设置



最后是设置CentOS 的网络配置文件

```
cd /etc/sysconfig/network-scripts

vi ifcfg-ens33 //这个名字每个电脑可能会不一样
```

主要把下面的配置做修改或者放进去，其他的可以不用动

```
BOOTPROTO=static
IPADDR=192.168.60.129
NETMASK=255.255.255.0
GATEWAY=192.168.60.2
DNS1=8.8.8.8
DNS2=114.114.114.114
```

然后使网络配置生效

```
service network restart
```

防火墙关闭

关闭防火墙：

```
service iptables stop  
```

关闭防火墙自启：

```
 chkconfig iptables off
```



## 免密登录

就是设置集群之间的相互信任

```
rpm -qa | grep ssh

ssh localhost  
yes
```



## JDK安装

由于Hadoop的各个软件都是JAVA写的，所以JDK环境必不可少。







Hadoop的下载官网会有链接，选择适合的tar版本就行了，比如我下载的是 `hadoop-2.7.7.tar.gz`。



# 正式开工

首先要明确，文件上传的linux上并且修改相关的配置文件只需要在一台服务器上完成即可，后面可以使用`scp`把相关资源一股脑复制到其他的机器上。

上传

移动

配置文件修改

主要需要修改的配置文件并不多，只要理清要做的目的是什么就行了。

确定默认

配置文件移动至其他集群







