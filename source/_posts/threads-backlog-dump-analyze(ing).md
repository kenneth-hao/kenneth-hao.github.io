---

title: Analyze Thread Backlog via Thread-dump
date: 2019-09-16 19:50:22
tags: [Java, thread-dump]
excerpt: 线程积压案例分析

---


原因：JVM 线程积压，导致服务不可用


排查过程：

总体思路是，找到异常线程，排查下异常线程中不正常的地方。

具体步骤如下:

`top -Hp {java-pid}` 发现只有几个线程处于 RUNNABLE， 300+ 的线程在 Sleeping

![线程积压](http://h.img.siblings.top/2018/11/07/threads_backlog_top_summary.png)

`top -b -Hp {java-pid} > threads.top`  把所有 JVM 进程内的线程导出到文件

将文件按照 TIME+ （线程累计使用的 CPU 时间）排序

![](http://h.img.siblings.top/2018/11/07/threads_backlog_sort_by_cpu_hold_time.png)



找到其中占用 CPU 时间较长的某个线程 ID，如 `23647` , 通过转化为 16 进制 `0x5c5f`。

到这里定位到可能出现问题的线程编号  `0x5c5f`.

通过 `kill -3 {java-pid}` 捕捉当前系统运行的线程堆栈。

然后分析 `thread_dump.out` 

![](http://h.img.siblings.top/2018/11/07/thread_dump_grep_info_001.png)


发现这个线程被阻塞，阻塞的原因，是因为在等待锁的释放。

怀疑是因为这个锁导致的线程积压。

![](http://h.img.siblings.top/2018/11/07/thead_dump_blocked_info_002.png)



```zsh
➜  cat dump1.out | grep 0x00000006c4f27c70 | wc -l
     159
```



可以看到有一大片的僵死进程，可以确认问题基本是在这个地方。



那我们在找一下死锁的线程堆栈

![](http://h.img.siblings.top/2018/11/07/thread_dump_locked_info_003.png)



通过线程堆栈，大致可以分析出，是 Apache Tiles 在解析 tiles 模板 `{name}.xml` 时的 DTD 文件时，因为 Socket 一直没有返回，导致锁一直没有被释放。

> 科普：`HttpURLConnection` 是基于 `HTTP` 协议的，其底层通过 `Socket` 通信实现。如果不设置超时（timeout），在网络异常的情况下，可能会导致程序僵死而不继续往下执行。



> charles 调试， 本地 DTD 引用 等