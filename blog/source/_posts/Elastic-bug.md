





创建多个Elastic节点的时候，需要拷贝多份，注意端口号不需要配置，会自动占用新的端口号。

但是如果拷贝时携带了  `data`文件夹，就会有冲突，报错如下：

> with the same id but is a different node instance

解决办法：

删除 data 文件夹即可。



做复杂查询的时候

查询范围不是自己想要的

```
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
```

想要查30岁以上的员工，实际上所有数据都返回了！

why?

解决：使用其他链接工具：kibana试试！







