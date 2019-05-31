##### 悲观并发控制

这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。

##### 乐观并发控制

Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。

###### 内部版本号

主要是利用`_version` 来确保。

```js
PUT /website/blog/1?version=1 
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```

以上操作只有之目标文档的版本为1时才会操作成功，成功后版本号会自动增加到2。

###### 通过外部系统进行版本控制

如果关系型数据库中有了版本号字段，可以增加 `version_type=external`到查询字符串中，这样就能重复使用这些版本号。 版本号必须是大于零的整数， 且小于 `9.2E+18` — 一个 Java 中 `long` 类型的正值。

如果使用了外部版本号，Elasticsearch 不是检查当前 `_version` 和请求中指定的版本号是否相同， 而是检查当前 `_version` 是否 *小于* 指定的版本号。 如果请求成功，外部的版本号作为文档的新 `_version` 进行存储。

外部版本号不仅在索引和删除请求是可以指定，而且在 *创建* 新文档时也可以指定。

例如，要创建一个新的具有外部版本号 `5` 的博客文章，我们可以按以下方法进行：

```js
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```



