---

title: 架构演进 at ArchSummit2016
date: 2016-12-07 21:03:48
tags: [architecture]
excerpt: 记录链家网架构演进的过程

---

## 架构演进 at ArchSummit2016 

今天为大家介绍一下链家网的架构演进历史.

### 架构之痛

在 2014-2016 两年间，B 端用户由 3W 增加到 15 W, 网站点击数量由 80,000,000 提升到 1,600,000,000。 

从业务图的最上面可以看出来, 整个系统的交易链路很长
整个平台主要分为 2B、2C 两大块。

对于 C 端用户, 如果系统出了问题, 用户可能就会丢失.

对于 B 端用户, 他们是非常敏感的, 如果系统出了问题会直接影响到他们的绩效, 奖金.

所以系统的稳定性对于链家而言是很重要的概念. 链家之前也很重视这块, 所以和一些知名的厂商进行了很深入的合作. 什么叫深入呢? 

就是花了好几个亿, 订做了一整套完整的系统, 当时的系统非常的高大上. 支撑了过去很多年的业务发展.

C 端项目主要用 PHP 实现。
B 端项目主要用 Java 实现，运行在 商业 的 Websphere 容器中。
前端用 F5 做硬件负载。
B 端存储主要用 Oracle。 C 端存储主要用 MySQL

这么强大的一套系统, 运行的怎么样呢?

有时候很有规律, 每个月总有那么几天会挂掉;

有时候也不是那么有规律, 每次挂掉的原因都不一样.

- 有时候没法登陆
- 有时候登录了, 更可怕, 串号了
- 还有时候, 数据库发生了死锁
- 总之, 出现了各种各样奇葩的问题

当时在刚刚接手这套系统的时候, 就是在救火, 同事给我起的外号, 叫消防队长. 

当时的情况就是拿着消防队长的钱, 操着卖白粉的心. 一个不小心就可能被十几万经纪人在背后默默的慰问了无数次.

总结一下过去遇到的技术问题

#### 现有问题

- 可用性差：99%
- 性能差：单机 QPS 不到50, 去做横向扩展是没有什么意义的
- 强耦合: 两个多月都没搞清楚系统之间  到底是什么样的关系
- 扩展性差、数据一致性差、容错能力差
- 可维护性差：上线难、迭代难
> QPS - req/sec 每秒请求服务器成功的次数, 通俗点说就是指, 服务器在一秒的时间内处理了多少个请求

> 独立访客 (UV) ：指一定时间范围内相同访客多次访问网站，只计算为1个独立访客。
> 页面浏览量（PV）：指一定时间范围内页面浏览量或点击量，用户每次刷新即被计算一次。

理论值计算: 
> 实际情况需要根据具体的场景、业务需求、服务配置、运营商带宽等来决定.
> 比如 新闻类, 因为交互较小, 通过把内容页缓存, 或者提前生成静态页面模板的方式, 然后由单台 Nginx 来处理静态页面 或者页面缓存. 并发量就可以达到很高的数值, 在不考虑带宽等情况, 理论值能达到上万的 QPS

公式: ( PV * 80%) / ( 24*60*60 * 20%) = 峰值时间 QPS
原理: 每日 80% 的访问集中在 20% 的时间里, 这 20% 叫做峰值时间

50 * 24 * 60 * 60 * 0.2 / 0.8 = 108,000
1,600,000,000  / 365 ≈ 4,000,000
4,000,000 / 108,000 ≈ 370


链家网在成立的时候, 就已经背上了沉重的历史包袱. 怎么才能甩掉过去的历史包袱, 搭建起自己的系统? 

### 高可用架构的演进与实践

#### 会话层

过去的会话层
B端使用 Session 复制的方式
C端使用 Session 绑定的方式

这两种方案都有明显的缺陷, 所以会出现像 单机容量不足, 串号, 恢复困难的问题.

Session Replication: 
- 每台机器都要保存 整个 Session数据, 很多人同时访问会带来很大的内存占用
- Session 数据有任何变化, 就需要通知到其他所有机器, 机器集群数量越多, 所消耗的网络带宽资源就越大
  Session Sticky:
- 如果Web 服务器重启, 会话会丢失.
- 负载服务器会保存 会话和具体服务器的映射,变成了一个有状态的节点, 在内存消耗和容灾上会更难处理

D-I-D 原则

Design: 采用 CAS 单点登录, 改造 CAS 协议来支持移动端登录 换掉了 黑盒的 Websphere.

Implement: 
有一次新系统上线, Passport 流量莫名增长了好几倍, 很多系统发生了严重的超时. 
Passport  下游系统很多. 很难直接排查到出现问题的下游系统, 当时解决这个问题只用了两分钟.

有PC、M端两个 Passport 服务器, 分开独力部署 有自己独立的 K-V 集群, 存放CAS自身协议的 Ticket.	不存储业务 Session.  
右边是业务系统集群, 都存储了自身的业务 Session 集群. 
有核心业务系统, 有非核心业务系统. 分别独立部署来实现业务系统之间的隔离.

左侧的红色开关做入口层的限流
右侧开关用来做业务系统的限流

这是一个去中心化的架构, 可以很方便的进行扩容.

Deploy: 根据自己系统实际的业务场景和流量的增长来部署.

Session 和 业务 Cache 要分开, 业务 Cache 过多, 会挤掉会话中的 Session 缓存, 用户会莫名奇妙掉线

#### 服务层

过去的服务之间 有很多循环依赖, 数据一致性差
当时有个哥们, 他每天必做的一件事儿,  就是接受派单, 写 SQL, 修数据, 他每个月总共修的数据能达到几百万条.

Why?
康威定律 - 一个组织它产生的设计, 是由这个组织内和这个组织之外的一些沟通关系、沟通结构决定的. 
换句话说, 就是我们的软件架构一定会受到公司部门人事组织架构的影响, 受到部门利益之间的影响.

在过去, B 端和 C  端是完全割裂的两个部门. 虽然他们访问的同一套源数据. 但是并没有做真正的服务化, 而且是用 ETL 工具做数据同步的方式. 所以就导致了很多数据不一致的情况.

在 B 端有很多系统, 这些部门之间的关系有得特别好, 好到可以穿一条裤子. 好到什么情况呢, 你的应用可以改我的数据库, 我的应该也可以改你的数据库.  

新系统在设计时 就需要避免这些问题.  那么如果来避免这些问题呢. 我们就需要几个原则.

首先是 KISS 原则,  -  Keep It Simple And Stupid. 
KISS 原则告诉我们要保持系统的简单, 因为越简单的系统越可靠. 
要把过去的系统变简单, 我们必须要把过去的业务做处理做分层.

这里就用到另一个原则叫分层依赖原则. 
我们要把系统分为底层服务、上层服务, 要求上层服务调用底层服务, 但底层服务不能调用上层服务.

在我们过去的系统里出现了大量的耦合, 还有大量数据接口同步调用的一些情况.
举个例子, 假如我们要新做一套房源. 我们需要在房源系统里添加一套房源, 同时还需要更新索引.
过去的做法是, 开启一个事务, 在事务里面做数据库的插入操作, 同时去写 ElasticSearch 去更新索引.  这个做法看起来实现了数据的一致性. 但实际上带来了很大的问题. 当 ElasticSearch 变慢的时候, 我们的数据库会被拖慢. 

在系统里面还有很多其他同步调用的案例, 这就导致其中一个系统出问题, 其他所有与之关联的系统可能都会出现问题. 牵一发而动全身.

这里就用到了著名的 CAP 原理, CAP原则又称CAP定理，指的是在一个分布式系统中， Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼.
从 CP -> AP.