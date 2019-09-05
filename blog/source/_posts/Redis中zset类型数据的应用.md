---
title: Redis中zset类型数据的应用（实例+原理）
date: 2019-05-27 
tags: [java,Redis]
---

# 项目需求

公司APP页面需要展示一个横轴为时间，纵轴为指定基金和沪深300指数(或者其他指数)的折线图。

折线图的范围是可选的（比如一个月内，三个月内，六个月内等等），并且由于每一支基金的净值公布节奏不同，同一个时间范围的实际首尾时间，以及具体哪些日期是有值也是不一样的。

还有一个特点是沪深300的值是相对固定的，每天只会更新一次。但是很多基金都会以沪深300作为比较基准，使得需要查询一个数据多次。

比如A基金（每天公布净值）的三个月范围需要查0201，0202，0203…… 0430这些日期对应的沪深300；但是B基金（每周公布净值）的三个月范围只需要查0202，0209，0216，0223……这些日期对应的沪深300。如果使用上面的步骤，势必会使得程序运行效率降低。

# 解决思路

对于这种类似于静态数据，或者说修改一次查询很多次的数据，使用缓存是最好的选择了。下面以Redis来说明解决方案。

Redis有五种数据存储格式，分别是String，List，Hash，Set，Zset。

除了存储常用的数据，Redis还可以用Set数据类型来进行去重工作，（后面有了布隆过滤器）；还可以使用Redis 共享分布式系统中的session；使用Redis解决分布式锁的问题；Redis的List还能做消息队列和发布订阅模式，实现消息传递和进程间的通信。

扯远了，具体到眼前的实例，实现起来非常简单，只需要使用带分数的集合类型就能行了。具体是使用沪深300的时间作为 `score` 分数，将每天的收盘价作为`value`，当确定时间范围之后取出对应时间范围的沪深300净值信息，即可拿到想要的数据。

# 代码实例

主要步骤

```
Set<Tuple> tuplesCsi300Set = RedisUtils.zrevrangeByScoreWithScores(csi300Key, dateParamMax, dateParamMin, 0, 1000);
    if (tuplesCsi300Set != null && tuplesCsi300Set.size() > 0) {
        compareGrowthList = getCompaFromRedis(param_csi300_arr, tuplesCsi300Set);
        System.out.println("compareGrowthList:" + compareGrowthList);
    } else {//先查，再存
        //查询数据库全部沪深300的数据存入Redis中
        JSONArray apiDataCsiRedis = getData();
        Map<Double, String> csi300Map = new HashMap<>();
        for (int i = 0; i < apiDataCsiRedis.size(); i++) {
            JSONObject objecti = (JSONObject) apiDataCsiRedis.get(i);
            csi300Map.put(Double.valueOf(objecti.getString("tDate")), objecti.getString("tClose"));
        }
        RedisUtils.addSortSetByScoreMap(csi300Key, csi300Map);
        Set<Tuple> tuplesCsi300 = RedisUtils.zrevrangeByScoreWithScores(csi300Key, dateParamMax, dateParamMin, 0, 1000);
        compareGrowthList = getCompaFromRedis(param_csi300_arr, tuplesCsi300);
        System.out.println("compareGrowthList:" + compareGrowthList);
    }

```

   取数据：

```
 /**
     * 处理从redis中获取到的set
     *
     * @param param_csi300_arr
     * @param csi3003YMap
     * @return
     */
    private LinkedList<Object> getCompaFromRedis(String[] param_csi300_arr, Set<Tuple> csi3003YMap) {

        LinkedList<Object> compareGrowthList = new LinkedList<>();
        compareGrowthList.addFirst(0.00);
        TreeMap<String, String> treeMap = new TreeMap<>();
        for (Iterator<Tuple> iterator = csi3003YMap.iterator(); iterator.hasNext(); ) {
            Tuple next = iterator.next();
            String score = Double.valueOf(next.getScore()).intValue() + "";
            treeMap.put(score, next.getElement());
        }
        //最早时间的指数值 逻辑：(当前值/最早值) -1 *100
        String s0 = treeMap.get(treeMap.firstKey());
        for (int i = 1; i <= param_csi300_arr.length - 1; i++) {
            //最早时间的指数值 逻辑：(当前值/最早值) -1 *100
            if (treeMap.keySet().contains(param_csi300_arr[i])) { //判断第i个时间参数对应的取值结果是否存在
                Double num = (Double.parseDouble(treeMap.get(param_csi300_arr[i])) / Double.parseDouble(s0) - 1) * 100;
                compareGrowthList.add(Double.parseDouble(df_No_comma.format(num)));
            } else {
                compareGrowthList.add(BLANKVALUE);
            }
        }
        return compareGrowthList;
    }
```



主要代码逻辑还是比较简单的，拿到开始和结束时间后查询缓存，查到了就拿出区间内的数据进行后续处理，否则就从数据库（或者其他接口）查询并处理。

# Redis有序集合实现原理

Redis的有序集合是使用跳表实现的。

相比红黑树，跳表的代码实现还是比较简单的，并且他也是一种支持动态扩容的数据结构。跳表的介绍不做多展开，具体参见数据结构章节的文章。

跳表这种结构实现类的类似二分查找的速度，查询的时间复杂度是O(logn)，很高效。