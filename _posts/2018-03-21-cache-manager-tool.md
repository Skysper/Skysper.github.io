---
layout: post
tId: 1803001
title: "CacheManager缓存管理开源工具"
date: 2018-03-21 22:53:00 +0800
categories: redis,架构,cache
codelang: html
desc: "最近使用cropper完成了项目中的图片上传功能，方便强大，是图片上传处理的一大利器"
---

本文主要介绍我的开源项目CacheManager的使用说明和特性
CacheManager可以协助管理我们项目应用中使用的Redis、Memcache缓存键值对。

目前已经实现了对Redis的支持，支持的数据类型包括String、List、Set、SortedSet、Hash，可以修改、删除键值，设置过期时间等。

### 1. 创建App信息
![创建App](//img-blog.csdn.net/20180321221325471?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

创建app应用，配置连接字符串，支持Redis集群

### 2. 添加App中使用到的键值
![应用使用缓存项](//img-blog.csdn.net/20180321221302912?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


##### 键值规则
缓存Key值名称创建可以包含两种方式:

- 单项Key值 - 直接对应缓存服务器中特定的Key，如: sys:cache:maxTTL 等
- 批量Key值 - 多项Key值，一对多的映射，此Key代表一个Key值序列，该序列支持两种模式
1. 数值类型，自动+1或-1操作，如 sys:cache:news:{1-10000}
2. 字符类型, 仅支持Char单字符形式，如 sys:cache:cluster{A-Z}

两种模式可以混合使用，如 sys:cache:cluster{A-Z}:news{1-1000},则查询的Key键列表为:

```
sys:cache:clusterA:news1
system.cache:clusterA:news2
...
system.cache:clusterA:news1000
system.cache:clusterB:news1
...
cache:clusterZ:news1000
```

*注：多项Key值模式，适合针对单表Id自增等有数值或字符规则的批量缓存进行管理*

### 3.查询管理

设定缓存的值和过期时间

![管理TTL](//img-blog.csdn.net/20180321222914489?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

查看列表
![缓存结果列表](//img-blog.csdn.net/20180321223210160?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

结果列表根据批量键值规则以及分页页码生成当前页的具体Key值，列出的Key值并不一定真实存在于Redis缓存中，列表中前四项的键值类型分别为String、List、Set(Sorted Set)、Hash

![Hash的键值设置](//img-blog.csdn.net/20180321223122806?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 值Value设置规则
- String 

```
我可以输入任意内容
```

- List、Set、Sorted Set

```
value1
value2
value3
```

- Hash

```
key1$$value1
key2$$value2
key3$$value3
```
