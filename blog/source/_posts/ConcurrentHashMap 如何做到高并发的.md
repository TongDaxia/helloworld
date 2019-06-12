使用场景

使用webmagic爬取百度榜单的时候，出现超时错误

报错内容

```
09:15:23.811 [pool-1-thread-1] DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection released: [id: 0][route: {}->http://top.baidu.com:80][total kept alive: 0; route allocated: 0 of 100; total allocated: 0 of 5]
09:15:23.814 [pool-1-thread-1] WARN us.codecraft.webmagic.downloader.HttpClientDownloader - download page http://top.baidu.com/category?c=513&fr=topbuzz error
java.net.SocketTimeoutException: Read timed out
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
```

问题分析

因为爬虫框架使用了SocketInputStream 读取网页内容，但是下载的网速很慢，使得超时，设置超时时间如下：

```java
 private Site site = Site.me().setRetryTimes(3).setSleepTime(100).
     setUserAgent("Mozilla/5.0 (Windows NT 6.1; WOW64; rv:53.0) Gecko/20100101 Firefox/53.0").setTimeOut(100000);

```



