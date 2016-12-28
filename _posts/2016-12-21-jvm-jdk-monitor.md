---
layout: post
title: 虚拟机性能监控工具介绍和使用
published: true
tags:
- java
- jvm
- jdk
categories: jvm
---

　　文章主要介绍JDK的命令行监控工具jps、jmap、jstack、jhat的介绍，以及可视化工具jconsle和visualvm的介绍和使用。

### JDK命令行工具
* jps工具

　　JVM Process Status Tool 显示制定系统中所有HotSpot虚拟机进程。
jps的格式为： jps [option] [hostid]
其中option具体取值有：

| 取值 | 描述 |
| -q | 只输出LVMID（Local Virtual Machine Identifier，本地虚拟机唯一ID）信息，不显示主类的名字 |
| -m | 显示虚拟机进程启动时传递给main函数的参数 |
| -l | 显示主类的全名，如果进程执行的jar，则显示jar的路径 |
| -v | 显示虚拟机进程启动时参数 |

* jstat工具

　　JVM Statistics Monitoring Tool，用户收集HotSpot虚拟机各方面的运行数据；可以显示本地或者远程类装载、内存、垃圾收集、JIT编译等运行数据。
jstat命令格式为： jstat [option vmid [interval[s|ms] [count]] ]

其中：如果是本地虚拟机进程那么VMID与LVIMD是一样的，如果是远程的虚拟机进程，那么格式为：

~~~~~
[protocol:][//]lvmid[@localhost[:port]/servername]
~~~~~

option对应的值有：

| 取值 | 描述 |
| -class | 监视类装载、卸载数量、总空间及类装载所耗费的时间 |
| -gc | 监视Java堆状况，包括Eden区、2个Survivor区、老年代、永久代等的容量 |
| -gccapacity | 监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大和最小空间 |
| -gcutil | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比 |
| -gccause | 与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因 |
| -gcnew | 监视新生代GC的状况 |
| -gcnewcapacity | 监视内容与-gcnew基本相同，输出主要关注使用到的最大和最小空间 |
| -gcold | 监视老年代GC的状况 |
| -gcoldcapacity | 监视内容与——gcold基本相同，输出主要关注使用到的最大和最小空间 |
| -gcpermcapacity | 输出永久代使用到的最大和最小空间 |
| -compiler | 输出JIT编译器编译过的方法、耗时等信息 |
| -printcompilation | 输出已经被JIT编译的方法 |


参数interval和count代表查询间隔和次数，如果省略这两个参数，说明只查询一次。假设需要每250毫秒查询一次进程2225垃圾收集状况，一共查询5次命令为：

~~~~~
jstat -gcutil 2225 250 5
~~~~~

运行的结果为：

~~~~~
 S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
 64.46   0.00  15.84  71.26  99.78     80    4.093     0    0.000    4.093
 64.46   0.00  15.84  71.26  99.78     80    4.093     0    0.000    4.093
 64.46   0.00  15.84  71.26  99.78     80    4.093     0    0.000    4.093
 64.46   0.00  15.84  71.26  99.78     80    4.093     0    0.000    4.093
 64.46   0.00  15.84  71.26  99.78     80    4.093     0    0.000    4.093
~~~~~

S0      年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
S1      年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
E       年轻代中Eden（伊甸园）已使用的占当前容量百分比
O       old代已使用的占当前容量百分比
P       perm代已使用的占当前容量百分比
YGC     从应用程序启动到采样时年轻代中gc次数
YGCT    从应用程序启动到采样时年轻代中gc所用时间(s)
FGC     从应用程序启动到采样时old代(全gc)gc次数
FGCT    从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT     从应用程序启动到采样时gc用的总时间(s)

* Jmap工具

　　Memory map for java， 生成虚拟机的内存转储快照（heapdump文件）;也可以设置在outofmemory的时候生成heapdump文件：-XX:+HeapDumpOutOfMemoryError；jmap不仅能生成dump文件，还阔以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。
Jmap命令格式为：

~~~~~

jmap [option] LVMID

~~~~~

option可选值为：

| 取值 | 描述 |
| -dump | 生成堆转储快照。格式为：-dump:[live,]format=b,file=<filename> pid |
| -finalizerinfo | 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象。只在linux和Solaris系统上有效 |
| -heap | 显示Java堆详细信息。只在linux和Solaris系统上有效 |
| -histo | 显示堆中对象的统计信息 |
| -permstat | 以ClassLoader为统计口径显示永久代内存信息。只在linux和Solaris系统上有效 |
| -F | 当-dump没有响应时，强制生成dump快照。只在linux和Solaris系统上有效 |

jmap例子：

~~~~~
jmap -dump:live,format=b,file=dump.hprof 28920
~~~~~

结果为：

~~~~~

[root@dev tmp]$ jmap -dump:live,format=b,file=dump.hprof 2225
Dumping heap to /tmp/dump.hprof ...
Heap dump file created
[root@dev tmp]$ ls

~~~~~

这个生成的文件需要通过下面要介绍的jhat工具来查看。


* jhat工具

　　JVM Heap Dump Browser，用户分析heapdump文件，会创建一个HTTP/HTML服务器，让用户可以在浏览器上面查看结果；
通过jhat来查看上面生成的dump.hprof文件：
~~~~~
[root@dev tmp]$ jhat dump.hprof
Reading from dump.hprof...
Dump file created Thu Dec 22 22:27:10 CST 2016
Snapshot read, resolving...
Resolving 1447482 objects...
WARNING:  Failed to resolve object id 0xe26c5bc8 for field clazz (signature L)
WARNING:  Failed to resolve object id 0xe26c53e8 for field clazz (signature L)
WARNING:  Failed to resolve object id 0xe26c4f18 for field clazz (signature L)
WARNING:  Failed to resolve object id 0xe26c4b80 for field clazz (signature L)
WARNING:  Failed to resolve object id 0xe26c4630 for field clazz (signature L)
WARNING:  Failed to resolve object id 0xe26c44f0 for field clazz (signature L)
Chasing references, expect 289 dots.................................................................................................................................................................................................................................................................................................
Eliminating duplicate references.................................................................................................................................................................................................................................................................................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
~~~~~

然后通过浏览器请求对应ip加端口就可以看到具体对象的内存使用情况，如图：

![](/img/20161215/jhat.png)


* jstack

　　Stack Trace For Java，显示虚拟机的线程快照。

jstack命令格式为：

~~~~~

jstack [option] LVMID

~~~~~

option具体选项有：
| 取值 | 描述 |
| -F | 当‘jstack [-l] pid’没有相应的时候强制打印栈信息 |
| -l | 除对账外，打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表. |
| -m | 打印java和native c/c++框架的所有栈信息. |

jstack例子：

~~~~~
jstack -l 2225
~~~~~

部分线程结果为：

~~~~~
[finance@dev tmp]$ jstack -l 2225
2016-12-24 16:14:00
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.79-b02 mixed mode):

"http-nio-8002-exec-10" daemon prio=10 tid=0x00007f11780c4800 nid=0x254d waiting on condition [0x00007f11bd1e7000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000e4bd8898> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:103)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:31)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"http-nio-8002-exec-9" daemon prio=10 tid=0x00007f117400f000 nid=0x253a waiting on condition [0x00007f11bd2e8000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000e4bd8898> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:103)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:31)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"http-nio-8002-exec-8" daemon prio=10 tid=0x00007f1178008000 nid=0x24fa waiting on condition [0x00007f11bd3e9000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000e4bd8898> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:103)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:31)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
~~~~~

### 可视化监控工具

* JConsole工具
　　基于JMX的可视化监视、管理工具，能够比较好的监控堆内存、线程等情况。


* visualvm工具
　　集成多个JDK命令行工具的可视化工具，它能为您提供强大的分析能力，对Java应用程序做性能分析和调优，并且有丰富的插件，也是笔者开发过程中用到的内存跟踪工具。
