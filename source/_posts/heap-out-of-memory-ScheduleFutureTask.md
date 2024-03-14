---

title: Heap OutOfMemory - ScheduledFutureTask
date: 2020-03-10 20:50:55
tags: [OOM, Java]
excerpt: OOM 案例分析

---

## Key

```java
Hystrix.Setter.executionTimeout = Integer.MAX_VALUE
```


## Overview

Dump 文件分析发现 `java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask` 的堆内存占用约 2 G，基本确认问题


![dump file - overview](http://h.img.siblings.top/2020/03/10/29bd2dba-4bea-4778-be4f-fd88ef2da239.png)



## Analysis

尝试通过查看类 Reference 关系，确定下发生内存内泄漏的原因



### Outgoing Reference


先看下实例内部引用「Outgoing Reference」

![dump file - outgoing reference 01](http://h.img.siblings.top/2020/03/10/e1e7f1ca-0e66-4475-8c7b-6d2f7b53c2ac.png
)



> 可以看到有 20,000,000+ 的实例，我们应该看哪个呢？

通过 Retained Size 排序，找到 size 相同的出现频率最高的实例对象


![dump file - outgoing reference 02](http://h.img.siblings.top/2020/03/10/7b9c5368-d286-49e8-af48-aafa54738fb4-2.png)



这里有两个明显异常的地方

1. 调度线程的调度周期是 `Integer.MAX_VALUE`
2. 调度线程的队列长度非常大，约等于这个类实例的数量
3. 调度线程的任务已经被取消 `status=4`

### Incoming Reference

查看引用实例的地方「Incoming Reference」

按照 `Retained Size` 排序，找出同样大小，出现频率最高的对象实例

![dump file - incoming reference 02](http://h.img.siblings.top/2020/03/10/834493ba-d3b3-4923-be1e-2562b8f94e21.png
)

到这儿，我们基本可以确认是 `HystrixTimer` 相关的实现导致了 `ScheduledThreadPollExecutor` 的延迟队列（unlimit）中无限制的堆积了 `ScheduledFutureTask`


### Why


通过关键字 `HystrixTimer ScheduledFutureTask` Google。



![google-oriented programming - 01](http://h.img.siblings.top/2020/03/10/a4fac5e9-bc8c-4727-bcc1-534f7950e094.png
)


通过搜索结果，基本可以定位  问题主要和 Hystrix Timeout 相关。


我们可以发现， 已经有人在 16 年遇到过类似的问题了

![google-oriented programming - 02](http://h.img.siblings.top/2020/03/10/984a7ca3-c507-4654-b9c7-8e506c7e6dd4.png
)


原因，上面的 Google Group 解释的比较清楚了。可以对照源码看一下。

1. `Hystrix` 每次会调用 `HystrixTimer` 来生成一个 `TimeListener`
2. `TimeListener` 会使用 `ScheduledThreadPoolExecutor.scheduleAtFixRate()` 来申明一个调度队列，因为我们的 `Hystrix.Setter.executionTimeout` 设置为了 `Integer.MAX_VALUE`, 导致调度队列在初始化时，调度周期 = `Integer.MAX_VALUE`
3. Command 执行完成后，调度线程池的延迟队列中的调度任务状态会被设置为取消，但因为 JDK7 的默认策略是取消任务不会立即移除，只有在任务触发时才会移除，所以调度任务会常驻在队列中，直到任务触发
4. 调度线程池中的延迟队列属于无界（unlimit）队列，所以队列中的任务会一直堆积，直到 `JVM Restart`


## Solution


调整 `Hystrix.Setter.executionTimeout` 为合理数值 / 关闭 `Hystrix Timeout`


## Ref

[Retained Size](https://www.jianshu.com/p/851b5bb0a4d4)

[Incoming Reference、Outgoing Reference](https://blog.csdn.net/weixin_45410925/article/details/102740403)

[线程池相关](https://zhuanlan.zhihu.com/p/32867181)

[Hystrix Timeout 实现](https://segmentfault.com/a/1190000015393836)

[ScheduledThreadPoolExecutor.setRemoveOnCancelPolicy(boolean)](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ScheduledThreadPoolExecutor.html)