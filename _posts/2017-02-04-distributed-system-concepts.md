---
layout: post
tId: 1609071
title: "CAP理论和BASE思想"
date: 2017-02-04 13:00:00 +0800
categories: 架构,分布式
codelang: csharp
desc: "在深入研究分布式系统之前，应该首先了解分布式系统的一些基本原理，以便于更好的理解和探索分布式系统的世界，首先是CAP理论和BASE思想"
---
### CAP理论 ###
在计算机科学中, CAP定理（CAP theorem）, 又被称作 布鲁尔定理（Brewer's theorem）, 它指出对于一个分布式计算系统来说，不可能同时满足以下三点:
  * C : 一致性(Consistency) -- 所有节点在同一时间具有相同的数据
  * A : 可用性(Availability) -- 保证每个请求不管成功或者失败都有响应
  * P : 分隔容忍(Partition tolerance) -- 系统中任意信息的丢失或失败不会影响系统的继续运作

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。
因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：
* CA : 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
* CP : 满足一致性，分区容忍性的系统，通常性能不是特别高。
* AP : 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。


### BASE思想 ###
BASE：Basically Available, Soft-state, Eventually Consistent。 由 Eric Brewer 定义。
BASE是NoSQL数据库通常对可用性及一致性的弱要求原则:
* Basically Availble -- 基本可用
* Soft-state -- 软状态/柔性事务。 "Soft state" 可以理解为"无连接"的, 而 "Hard state" 是"面向连接"的
* Eventual Consistency -- 最终一致性 最终一致性， 也是是 ACID 的最终目的。

#### 与BASE思想相对的是关系型数据库中的ACID原则：####
* A : 原子性(Atomicity) -- 一个事务中所有操作都必须全部完成，要么全部不完成
* C : 一致性(Consistency) -- 在事务开始或结束时，数据库应该在一致状态
* I : 隔离性(Isolation) -- 事务将假定只有它自己在操作数据库，彼此不知晓
* D : 持久性 (Durable) -- 一旦事务完成，就不能返回
