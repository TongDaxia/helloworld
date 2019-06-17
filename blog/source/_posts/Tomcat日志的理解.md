---
title: Tomcat日志的理解
date: 2019-06-11 
tags: [java,Tomcat,日志] 
---

##### 几种log

Tomcat默认使用JULI日志系统

![1560220401868](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1560220401868.png)

Server、Tomcat Localhost Log 和Tomcat Catalina Log:

1. catalina.out 是tomcat的标准输出(stdout)和标准出错(stderr).是在tomcat的启动脚本里指定的；一般包括控制台输出。
2. Localhost log 对应 CATALINA_BASE指向目录的localhost.{yyyy-MM-dd}.log 文件。记录的是应用初始化(listener, filter, servlet)未处理的异常最后被tomcat捕获而输出的日志。
3. Catalina log 对应 CATALINA_BASE指向目录的localhost.{yyyy-MM-dd}.log 文件。 是tomcat自己运行的一些日志。应用向console输出的日志不会输出到catalina.{yyyy-MM-dd}.log。
4. Tomcat 默认manager应用日志，文件名manager.日期.log，如果配置了其他的项目应用，如下配置了Demo ，会出现demo.{yyyy-MM-dd}.log日志。

<!--more-->



![1560219576716](E:\tyg\study\blog\source\_posts\img\1560220401868.png)

##### 日志的级别

分为如下 7 种：

SEVERE (highest value) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value)

```
SEVERE(highest)﻿ Captures exception and Error
WARNING﻿ Warning messages
INFO﻿ Informational message, related to the server activity
CONFIG﻿ Configuration message
FINE﻿ Detailed activity of the server transaction (similar to debug)
FINER﻿ More detailed logs than FINE
FINEST(least)﻿ Entire flow of events (similar to trace)
```



##### 常见的日志配置 

apache-tomcat-8.5.31	logging.properties

```properties
#配置tomcat的日志输出方式，这里表示文件输出和控制台输出
handlers = 1catalina.org.apache.juli.AsyncFileHandler, 2localhost.org.apache.juli.AsyncFileHandler, 3manager.org.apache.juli.AsyncFileHandler, 4host-manager.org.apache.juli.AsyncFileHandler, java.util.logging.ConsoleHandler

.handlers = 1catalina.org.apache.juli.AsyncFileHandler, java.util.logging.ConsoleHandler

############################################################
# Handler specific properties.
# Describes specific configuration info for Handlers.
############################################################

1catalina.org.apache.juli.AsyncFileHandler.level = FINE
1catalina.org.apache.juli.AsyncFileHandler.directory = ${catalina.base}/logs
1catalina.org.apache.juli.AsyncFileHandler.prefix = catalina.

2localhost.org.apache.juli.AsyncFileHandler.level = FINE
2localhost.org.apache.juli.AsyncFileHandler.directory = ${catalina.base}/logs
2localhost.org.apache.juli.AsyncFileHandler.prefix = localhost.

3manager.org.apache.juli.AsyncFileHandler.level = FINE
3manager.org.apache.juli.AsyncFileHandler.directory = ${catalina.base}/logs
3manager.org.apache.juli.AsyncFileHandler.prefix = manager.

4host-manager.org.apache.juli.AsyncFileHandler.level = FINE
4host-manager.org.apache.juli.AsyncFileHandler.directory = ${catalina.base}/logs
4host-manager.org.apache.juli.AsyncFileHandler.prefix = host-manager.

java.util.logging.ConsoleHandler.level = FINE
java.util.logging.ConsoleHandler.formatter = org.apache.juli.OneLineFormatter


############################################################
# Facility specific properties.
# Provides extra control for each logger.
############################################################

org.apache.catalina.core.ContainerBase.[Catalina].[localhost].level = INFO
org.apache.catalina.core.ContainerBase.[Catalina].[localhost].handlers = 2localhost.org.apache.juli.AsyncFileHandler

org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/manager].level = INFO
org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/manager].handlers = 3manager.org.apache.juli.AsyncFileHandler

org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/host-manager].level = INFO
org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/host-manager].handlers = 4host-manager.org.apache.juli.AsyncFileHandler

# For example, set the org.apache.catalina.util.LifecycleBase logger to log
# each component that extends LifecycleBase changing state:
#org.apache.catalina.util.LifecycleBase.level = FINE

# To see debug messages in TldLocationsCache, uncomment the following line:
#org.apache.jasper.compiler.TldLocationsCache.level = FINE

# To see debug messages for HTTP/2 handling, uncomment the following line:
#org.apache.coyote.http2.level = FINE

# To see debug messages for WebSocket handling, uncomment the following line:
#org.apache.tomcat.websocket.level = FINE

```

其中的 `CATALINA_BASE` 在windows中是在 catalina.bat 文件在配置的。

在idea中启动Tomcat可根据控制台输出寻找对应地址。

![1560220291955](E:\tyg\study\blog\source\_posts\img\1560220291955.png)比如我的会默认指向：

```
C:\Users\Administrator\.IntelliJIdea2017.3（IDEA版本）\system\tomcat\项目名\logs
```

如果想要在IDEA中启动Tomcat修改其日志（或者其他）配置，在以下目录中修改：

```
C:\Users\Administrator\.IntelliJIdea2017.3（IDEA版本）\system\tomcat\项目名\conf
```











