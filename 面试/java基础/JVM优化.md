#### 影响一个系统性能的因素非常多(代码，数据结构+算法，web容器)，JVM是最后优化的手段。

## 常用性能测试指标

### 响应时间

常用组件的响应时间：

操作|响应时间 ---|--- 打开一个站点 |几秒 

数据库查询一条记录（有索引）| 十几毫秒 

机械磁盘一次寻址定位| 4毫秒 

从机械磁盘顺序读取1M数据 |2毫秒 

从SSD磁盘顺序读取1M数据| 0.3毫秒 (使用SSD与使用机械硬盘相比性能可以提高将近10倍)

从远程分布式缓存Redis读取一个数据|0.5毫秒 

从内存读取1M数据 |十几微妙 

Java程序本地方法调用 |几微妙(数据如果直接在本地内存，那么它的读取效率比数据库快将近1000倍，比redis快30倍左右；如果数据在redis缓存，那么它的读取速度比数据库快30倍左右；这也是为什么使用缓存是提升系统性能的“银弹”的原因)

### 并发数

并发数是指同一时刻，对服务器有实际交互的请求数。一般并发数是在**在线用户数**的5%-15%之间，如在线用户数是1000，那么可以预估并发数在50-150之间。

### 吞吐量

吞吐量是单位时间内完成的工作量(请求)的数量。如：每分钟的数据库事务，每秒传送的文件千字节数，每分钟的 Web 服务器命中数。

# GC目标

#### GC优化的终极目的

- GC的时间够小
- GC的次数够少，发生Full GC的周期足够的长，时间合理，最好是不发生。

#### GC运行指标

##### 如果满足则一般不需要调优：

- Minor GC执行时间不到50ms；- Minor GC执行不频繁，约10秒一次；
- Full GC执行时间不到1s；
- Full GC执行频率不算频繁，不低于10分钟1次；

#### 调优的原则

1. 大多数的java应用不需要GC调优
2. 大部分需要GC调优的的，不是参数问题，是代码问题
3. 在实际使用中，分析GC情况优化代码比优化GC参数要多得多；
4. GC调优是最后的手段

#### GC调优的最重要的三个选项

1. 选择合适的GC回收器
2. 选择合适的堆大小
3. 选择年轻代在堆中的比重



# GC日志

JVM参数配置：其中 -X 这个字母代表它是 JVM 运行时参数

```
堆设置
-Xms:初始堆大小
-Xmx:最大堆大小(通常将-Xms和-Xmx设置成一样的，因为当堆不够用而发生扩容时，会发生内存抖动影响程序运行时的稳定性)
-Xmn:新生代大小
-Xss：规定了每个线程虚拟机栈及堆栈的大小，一般情况下，256k是足够的，此配置将会影响此进程中并发线程数的大小。
-XX:NewRatio:设置新生代和老年代的比值。如：为3，表示年轻代与老年代比值为1：3
-XX:SurvivorRatio:新生代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：为3，表示Eden：Survivor=3：2，一个Survivor区占整个新生代的1/5  
-XX:MaxTenuringThreshold:设置转入老年代的存活次数。如果是0，则直接跳过新生代进入老年代，默认为15。
-XX:PermSize、-XX:MaxPermSize:分别设置永久代最小大小与最大大小（Java8以前）
-XX:MetaspaceSize、-XX:MaxMetaspaceSize:分别设置元空间最小大小与最大大小（Java8以后）
收集器设置
Serial收集器
-XX:+UseSerialGC->指定年轻代为Serial收集器
-XX:+UseSerialOldGC->指定老年代为Serial收集器
ParNew收集器
-XX:+UseParNewG->指定年轻代为ParNew收集器
Parallel Scavenge收集器
-XX:+UseParallelGC->指定年轻代为Parallel收集器
-XX:+UseParallelOldGC->指定老年代为Parallel收集器
-XX:ParallelGCThreads->指定GC工作的线程数量
CMS收集器
-XX:+UseConcMarkSweepGC->指定指定老年代为CMS收集器
-XX:ConcGCThreads->并发的GC线程数
-XX:+UseCMSCompactAtFullCollection->FullGC之后是否做压缩整理(减少碎片)
-XX:CMSFullGCsBeforeCompaction->多少次FullGC之后压缩一次，默认是0，代表每次FullGC后都会压缩一次，比如-XX:CMSFullGCsBeforeCompaction=0 
-XX:CMSInitiatingOccupancyFraction->当老年代使用达到该比例时会触发FullGC(默认是92，这是百分比)，比如-XX:CMSInitiatingOccupancyFaction=92
-XX:+UseCMSInitiatingOccupancyOnly->只使用设定的回收阈值(-XX:CMSInitiatingOccupancyFraction设定的值)，如果不指定，JVM仅在第一次使用设定值，后续则会自动调整
-XX:+CMSScavengeBeforeRemark->在CMSGC前启动一次minor gc，目的在于减少老年代对年轻代的引用，降低CMS GC的标记阶段时的开销，一般CMS的GC耗时80%都在 remark阶段 
G1收集器
-XX:+UseG1GC->开启G1收集器
-XX:G1HeapRegionSize->指定分区大小(1MB~32MB，且必须是2的幂)，默认将整堆划分为2048个分区
-XX:MaxGCPauseMillis->目标暂停时间(默认200ms)
-XX:G1NewSizePercent->新生代内存初始空间(默认整堆5%)
-XX:G1MaxNewSizePercent->新生代内存最大空间
-XX:-DoEscapeAnalysis表示关闭逃逸分析(可以用来分析逃逸分析能够将未方法逃逸的对象分配栈上)
-XX:+DoEscapeAnalysis 打开逃逸分析
垃圾回收统计信息
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:filename
并行收集器设置
-XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数。并行收集线程数。
-XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间
-XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)
并发收集器设置
-XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况。
-XX:ParallelGCThreads=n:设置并发收集器新生代收集方式为并行收集时，使用的CPU数。并行收集线程数。
```



以参数-Xms5m -Xmx5m -XX:+PrintGCDetails -XX:+UseSerialGC为例：(测试发现，无论配5还是6，租后都是6，无论配7还是8，都是8，说明默认是偶数)

```
[GC (Allocation Failure) [DefNew: 2431K->255K(2432K), 0.0009521 secs] 7494K->5412K(7936K), 0.0009700 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

DefNew：新生代发生收集
[DefNew: 2431K->255K(2432K), 0.0009521 secs]  表示空间回收前 新生代占用1855K，回收后占用1855K。(2432K目前总大小)
7494K->5412K(7936K)： 表示堆的回收情况，7936K表示当前整个堆的大小

[Full GC (Allocation Failure) [Tenured: 5503K->5503K(5504K), 0.0243292 secs] 7935K->6166K(7936K), [Metaspace: 21629K->21629K(1069056K)], 0.0243776 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 

Tenured：老年代发生收集
7935K->6166K(7936K)：表示堆的回收
[Metaspace: 21629K->21629K(1069056K)]：元空间回收


Heap
 def new generation   total 2432K, used 2380K [0x00000007bf800000, 0x00000007bfaa0000, 0x00000007bfaa0000)
  eden space 2176K,  99% used [0x00000007bf800000, 0x00000007bfa1fff8, 0x00000007bfa20000)
  from space 256K,  81% used [0x00000007bfa20000, 0x00000007bfa540e8, 0x00000007bfa60000)
  to   space 256K,   0% used [0x00000007bfa60000, 0x00000007bfa60000, 0x00000007bfaa0000)
 tenured generation   total 5504K, used 5503K [0x00000007bfaa0000, 0x00000007c0000000, 0x00000007c0000000)
   the space 5504K,  99% used [0x00000007bfaa0000, 0x00000007bffffff0, 0x00000007c0000000, 0x00000007c0000000)
 Metaspace       used 28696K, capacity 30316K, committed 30464K, reserved 1075200K
  class space    used 3933K, capacity 4239K, committed 4352K, reserved 1048576K

新生代 = 2176K+256K+256K =2688K
老年代 = 5504K
元空间 = 30316K
堆空间 = 新生代+老年代=2688K+ 5504K=8192K=8M

新生代占比 = 2688/8192=0.33
老年代占比 = 5504/8192=0.66
```

==当然每种垃圾收集器的日志都不同==

```
G1收集器：
[GC pause (G1 Evacuation Pause) (young), 0.0022889 secs]
   [Parallel Time: 1.5 ms, GC Workers: 8]
      [GC Worker Start (ms): Min: 5246.3, Avg: 5246.4, Max: 5246.5, Diff: 0.2]
      [Ext Root Scanning (ms): Min: 0.2, Avg: 0.4, Max: 0.7, Diff: 0.5, Sum: 3.1]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.1]
      [Object Copy (ms): Min: 0.4, Avg: 0.7, Max: 0.9, Diff: 0.5, Sum: 6.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
         [Termination Attempts: Min: 1, Avg: 7.8, Max: 13, Diff: 12, Sum: 62]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.2]
      [GC Worker Total (ms): Min: 1.1, Avg: 1.2, Max: 1.3, Diff: 0.2, Sum: 9.5]
      [GC Worker End (ms): Min: 5247.6, Avg: 5247.6, Max: 5247.6, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 0.7 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.5 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 3072.0K(3072.0K)->0.0B(2048.0K) Survivors: 0.0B->1024.0K Heap: 3072.0K(6144.0K)->928.7K(6144.0K)]

```

# 查看堆栈情况

1.jconsole 本地直连程序，可以简单明了地查看到内存的使用情况, 线程的状态, 当前加载的类的总量等.

## 2.jps  查看运行的java进程号

jps主要用来输出JVM中运行的进程状态信息。语法格式如下：

```
jps [options] [hostid]
```

如果不指定hostid就默认为当前主机或服务器。

命令行参数选项说明如下：

```
-q 不输出类名、Jar名和传入main方法的参数

-m 输出传入main方法的参数

-l 输出main类或Jar的全限名（常用）

-v 输出传入JVM的参数（常用）
```









# 优化实战：

##### 1.大对象(讲到大对象主要指字符串和数组)直接进入老年代( 目的是避免在Eden区和两个Survivor区之间发生大量的内存复制)，但是这些大对象，又很频繁创建，导致老年代空间不足，通过设置参数来设置大对象()的定义：

```
-XX:PretenureSizeThreshold=6M  //设置超过6M才被分配到大对象
```



### 内存泄露排查

#### 什么是OutOfMemoryError

- java.lang.OutOfMemoryError：是指程序在申请内存时，没有足够的内存空间供其使用，出现OutOfMemoryError。产生原因产生该错误的原因主要包括：
  - JVM内存过小。
  - 程序不严密，产生了过多的垃圾。

#### 什么是内存泄漏

​		Memory Leak是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存，迟早会被占光。在Java中，内存泄漏就是存在一些被分配的对象，这些对象有下面两个特点：

- 首先，这些对象是可达的，即在有向图中，存在通路可以与其相连；
- 其次，这些对象是无用的，即程序以后不会再使用这些对象。

### ==内存泄露会最终会导致内存溢出。==

#### 怎么排查问题

1.top 看下哪个应用占CPU和内存高

2.jps 查看应用的PID

3.jmap 生成报告，jmap -dump:live,format=b,file=/Users/ganxinming/heap1.hprof 28744

或者 查看占用排名前几的   jmap -histo pidöhead -n 10   

==到这里基本上能定位原因了，如果还不能，可以把堆文件放在MAT上进行分析==

**以下几种情况可能会导致内存泄漏：**

- **静态集合类**：如 HashMap，Vector，静态容器的生命周期与应用程序一致。如果容器内的对象不再使用，则存在内存泄漏。

  **预防：**尽量不使用静态变量。

- **各种连接，流**：如 数据库连接，网络连接，IO连接，流等。例如，创建的连接不再使用时，需要调用 **close** 方法关闭连接，只有连接被关闭后，GC 才会回收对应的对象（Connection，Statement，ResultSet，Session）。忘记关闭这些资源会导致持续占有内存，无法被 GC 回收。

  **预防：**使用 `finally`块关闭资源。

- **监听器**：应用可能会用到多个监听器，但在释放目标对象的同时往往没有删除相应的监听器。

- **变量不合理的作用域**：一个变量的定义作用域大于其使用范围，很可能存在内存泄漏；或不再使用对象没有及时将对象设置为 null，很可能导致内存泄漏的发生。

  **例如**，只在方法内使用的对象，定义为了成员变量，方法内使用完后因被外部对象导致不能及时地被回收，就存在内存泄漏。

- **引用了外部类的非静态内部类**：非静态内部类（或匿名类）的初始化总是需要依赖外部类的实例。默认情况下，每个非静态内部类都包含对其**包含类**的隐式引用，若在程序中使用这个内部类对象，那么**即使在包含类对象超出范围之后，也不会被回收**（内部类对象隐式地持有外部类对象的引用，使其成不能被回收）。

  **预防：**如果内部类不需要访问对其包含类的成员，应将其转换为静态类。

- **单例模式**：单例对象在初始化后会以静态变量的方式在 JVM 的整个生命周期中存在。如果单例对象持有外部的引用，那么这个外部对象将不能被 GC 回收，导致内存泄漏。

  **预防：**尽可能使用懒加载，而不是立即加载。

- **改变对象哈希值**：当对象存储在一个 Hash 容器中（HashSet，HashMap），就不能修改这个对象中那些参与计算哈希值的字段。

  否则，对象修改后的哈希值与之前存储进容器的哈希值就不同，在此情况下，即使在`contains`方法使用该对象的当前引用作为的参数去容器中检索对象也是找不到的，这会导致无法从容器中**单独删除**当前对象，造成内存泄露。

- **子类重写finalize()**：每当重写类的 `finalize()`方法，该类的对象不会立即被垃圾收集。相反，GC 将它们排队等待最终确定，回收将在稍后的时间点发生。如果用 `finalize()`方法编写的代码不是最佳的，并且**终结器队列**无法跟上Java垃圾收集器，那么迟早将产生 `OutOfMemoryError`。

  **预防：**应该总是避免重写 `finalize`。

- **ThreadLocal** 造成的内存泄漏：ThreadLocal 可以实现变量的线程隔离，但若使用不当，就可能会引入内存泄漏问题。

  一旦线程结束不再存在，ThreadLocals 应该被垃圾回收。但是当 ThreadLocals 与 Tomcat 一起使用则可能出现问题。

  Tomcat 使用线程池中的线程来处理请求，而不是每个请求都创建新的线程。线程复用情况下，线程中的 ThreadLocals 并不会被回收，即 ThreadLocal 仍被线程对象持有。ThreadLocal 的生命周期不等于 Request 的生命周期，而是与线程生命周期绑定。

  **预防：**在不使用 ThreadLocals 中的变量时调用 remove() 方法删除当前线程的变量值。

  不能使用 `ThreadLocal.set(null)` 来清除该值。此操作实际是查找当前线程的 `ThreadLocalMap`，键是当前线程对象，将值设置 `null`（这段内容需要看源码比较好量解）。

  最好将 `ThreadLocal` 视为 `finally` 块中关闭的资源，以确保它始终会被关闭。

  ```java
  try {
      threadLocal.set("value");
      //...some processing......
  } finally {
      threadLocal.remove();
  }
  ```

### 排查

处理内存泄漏没有一个通用的或标准的解决方案，但可以根据应用的一些症状表现来分析定位内存泄漏，使用一些方法以最大限度地减少内存泄漏。

**内存泄漏可能的症状表现：**

- 应用程序长时间连续运行时性能严重下降
- CPU 使用率飙升，甚至到 100%
- 频繁 Full GC，各种报警，例如接口超时报警等
- 应用程序抛出 `OutOfMemoryError` 错误
- 应用程序偶尔会耗尽连接对象

严重**内存泄漏**往往伴随频繁的 **Full GC**，所以分析排查内存泄漏问题首先还得从查看 Full GC 入手。主要有以下操作步骤：

1. 使用 `jps` 查看运行的 Java 进程 ID

2. 使用`top -p [pid]` 查看进程使用 CPU 和 MEM 的情况

3. 使用 `top -Hp [pid]` 查看进程下的所有线程占 CPU 和 MEM 的情况

4. 将线程 ID 转换为 16 进制：`printf "%x\n" [pid]`，输出的值就是线程栈信息中的 **nid**。

   例如：`# printf "%x\n" 29471`，换行输出 **731f**。

5. 抓取线程栈：`# jstack 29452 > 29452.txt`，可以多抓几次做个对比。

   在线程栈信息中找到对应线程号的 16 进制值，如下是 **731f** 线程的信息。线程栈分析可使用 Visualvm 插件 **TDA**。

   ```properties
   "Service Thread" #7 daemon prio=9 os_prio=0 tid=0x00007fbe2c164000 nid=0x731f runnable [0x0000000000000000]
      java.lang.Thread.State: RUNNABLE
   ```

6. 使用`jstat -gcutil [pid] 5000 10` 每隔 5 秒输出 GC 信息，输出 10 次，查看 **YGC** 和 **Full GC** 次数。通常会出现 YGC 不增加或增加缓慢，而 Full GC 增加很快。

   或使用 `jstat -gccause [pid] 5000` ，同样是输出 GC 摘要信息。

   或使用 `jmap -heap [pid]` 查看堆的摘要信息，关注老年代内存使用是否达到阀值，若达到阀值就会执行 Full GC。

7. ==如果发现 `Full GC` 次数太多，就很大概率存在内存泄漏了【重要的判断标准，因为年轻代gc少，老年代gc多说明老年代存在无法回收对象】==

8. 使用 `jmap -histo:live [pid]` 输出每个类的对象数量，内存大小(字节单位)及全限定类名。

9. 生成 `dump` 文件，借助工具分析哪 个对象非常多，基本就能定位到问题在那了

   使用 jmap 生成 dump 文件：

   ```properties
   # jmap -dump:live,format=b,file=29471.dump 29471
   Dumping heap to /root/dump ...
   Heap dump file created
   ```

   或使用 JDK 自带的 `jvisualvm.exe`生成 dump 文件。

10. **dump** 文件分析

    可以使用 **jhat** 命令分析：`jhat -port 8000 29471.dump`，浏览器访问 jhat 服务，端口是 8000。

    通常使用图形化工具分析，如 JDK 自带的 **jvisualvm**，从菜单 > 文件 > 装入 dump 文件。

    或使用第三方式具分析的，如 **JProfiler** 也是个图形化工具，**GCViewer** 工具。Eclipse 或以使用 MAT 工具查看。或使用在线分析平台 **GCEasy**。

    **注意：**如果 dump 文件较大的话，分析会占比较大的内存。

11. 在 dump 文析结果中查找存在大量的对象，再查对其的引用。

    基本上就可以定位到代码层的逻辑了。
