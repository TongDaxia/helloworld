---
title: SpringCloud+mybatis+webmagic Mapper注入报错
date: 2019-06-17 
tags: [java,Spring,Bug]
---

##### 项目背景

Springcloud+mybatis+webMagic 获取百度热搜榜、搜狗热搜榜等热搜数据并存储到数据库中。

使用Mybatis Generator自动生成Mapper后放置在Mapper文件夹，并添加了对应的注解支持，service无法自动注入Mapper对象。

##### 相关配置

<!--more-->

application.yml

```
mybatis:
  config-location: classpath:mybatis/SqlMapConfig.xml # mybatis配置文件所在路径
  type-aliases-package: com.tyg.pojo  # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/*.xml  # mapper映射文件
```



##### 相关代码

启动类

```
package com.tyg;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@MapperScan("com.tyg.mapper")
public class DeptProviderApplication8002 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderApplication8002.class, args);
    }
}
```



service

```
package com.tyg.service.impl;

import com.tyg.mapper.BangMapper;
import com.tyg.mapper.ContentMapper;
import com.tyg.pojo.Bang;
import com.tyg.pojo.Content;
import com.tyg.service.BuzzBaiduSerivce;
import com.tyg.util.IdWorker;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import us.codecraft.webmagic.Page;
import us.codecraft.webmagic.Site;
import us.codecraft.webmagic.Spider;
import us.codecraft.webmagic.processor.PageProcessor;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.Date;

@Component
public class BuzzBaiduSerivceImpl implements PageProcessor, BuzzBaiduSerivce {

    private Site site = Site.me().setRetryTimes(3).setSleepTime(100)
            .setUserAgent("Mozilla/5.0 (Windows NT 6.1; WOW64; rv:53.0) Gecko/20100101 Firefox/53.0").setTimeOut(100000);

    private static String REGEX_PAGE_URL = "http://top.baidu.com/category?c=513&fr=topbuzz";


    @Autowired
    private BangMapper bangMapper;
    @Autowired
    private ContentMapper contentMapper;

    @Override
    public void process(Page page) {
       //TODO 在这里处理相关业务
       //bangMapper.insert(bang);
	   //contentMapper.insert(content);
    }
    
    @Override
    public Site getSite() {
        return site;
    }
    
    //@Scheduled(cron = "0 0/20 * * * ? ")//每20分钟跑一次
    @Override
    public void insertBang() {
        long startTime, endTime;
        System.out.println("开始爬取...");
        startTime = System.currentTimeMillis();
        Spider.create(new BuzzSougouServiceImpl()).addUrl(REGEX_PAGE_URL).thread(5).run();
        endTime = System.currentTimeMillis();
        System.out.println("爬取结束，耗时约" + ((endTime - startTime) / 1000) + "秒");
    }
}
```

mapper

```
package com.tyg.mapper;

import com.tyg.pojo.Content;
import com.tyg.pojo.ContentExample;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;


@Mapper
public interface ContentMapper {
    long countByExample(ContentExample example);

    int deleteByExample(ContentExample example);

    int deleteByPrimaryKey(Long id);

    int insert(Content record);

    int insertSelective(Content record);

    List<Content> selectByExample(ContentExample example);

    Content selectByPrimaryKey(Long id);

    int updateByExampleSelective(@Param("record") Content record, @Param("example") ContentExample example);

    int updateByExample(@Param("record") Content record, @Param("example") ContentExample example);

    int updateByPrimaryKeySelective(Content record);

    int updateByPrimaryKey(Content record);
}
```

##### 调试过程

**怀疑是注解使用错了！！！**

我是用的是@Mapper

应该是@Component

前者是Mybatis的注解，后者是Spring框架的注解。

虽然更改过来不报错了，但是运行时还是会报NPE.

现在怀疑是maven项目没有更新导致的；改用eclipse打开项目进行update操作看看是否有反应。

结果是错误的。

**终于找到原因**：

通过反复Debug发现，一开始进入到serviceImpl的时候是有mapper对象的，但是运行

process（）的时候每次调用mapper都报空指针异常。

所以，问题的关键是使用webMagic框架的时候， new了一个新对象，并使用新对象处理相关业务，但是新对象的并没有注入mapper bean。因为Spring 框架只在第一次启动将（service）对象生成并纳入容器内部时，生成了相关对象属性（mapper对象），在后续如果使用new的方式创建，获取不到相关属性对象，只能从bean中获取的时候才能得到属性对象。

##### 解决问题

  1.使用Spring框架提供的API获取Bean;

  2.将代码中的  `new BuzzSougouServiceImpl()`更换为`this` 。



本项目源码地址：[TongDaxia/goodIDEA](https://github.com/TongDaxia/goodIDEA)