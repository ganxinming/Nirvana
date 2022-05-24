# RDD设计背景

在实际应用中，存在许多迭代式算法（比如机器学习、图算法等）和交互式数据挖掘工具，这些应用场景的共同之处是，不同计算阶段之间会重用中间结果，==即一个阶段的输出结果会作为下一个阶段的输入==。但是，目前的MapReduce框架都是把中间结果写入到HDFS中，带来了大量的数据复制、磁盘IO和序列化开销。(==这样的话会频繁读写HDFS==)

RDD就是为了满足这种需求而出现的，==它提供了一个抽象的数据架构==，我们不必担心底层数据的分布式特性，只需将具体的应用逻辑表达为一系列转换处理，不同RDD之间的转换操作形成依赖关系，可以实现管道化，从而避免了中间结果的存储，大大降低了数据复制、磁盘IO和序列化开销。

Spark的核心是建立在统一的抽象RDD之上，使得Spark的各个组件可以无缝进行集成，在同一个应用程序中完成大数据计算任务。



# RDD概念

1.一个RDD就是一个==分布式对象集合==，本质上是一个只读的分区记录集合，每个RDD可以分成多个分区，每个分区就是一个数据集片段，并且一个RDD的不同分区可以被保存到集群中不同的节点上，从而可以在集群中的不同节点上进行并行计算。

2.RDD提供了一种高度受限的共享内存模型，即RDD是只读的记录分区的集合，==不能直接修改==，只能基于稳定的物理存储中的数据集来创建RDD，或者通过在其他RDD上执行确定的转换操作（如map、join和groupBy）而创建得到新的RDD。RDD提供了一组丰富的操作以支持常见的数据运算，==分为“行动”（Action）和“转换”（Transformation）两种类型==，前者用于执行计算并指定输出的形式，后者指定RDD之间的相互依赖关系。两类操作的主要区别是，转换操作（比如map、filter、groupBy、join等）接受RDD并返回RDD，而行动操作（比如count、collect等）接受RDD但是返回非RDD（即输出一个值或结果）。

3.RDD提供的转换接口都非常简单，都是类似map、filter、groupBy、join等粗粒度的数据转换操作，而不是针对某个数据项的细粒度修改。因此，RDD比较适合对于数据集中元素执行相同操作的批处理式应用，而不适合用于需要异步、细粒度状态的应用，比如Web应用系统、增量式的网页爬虫等。正因为这样，这种粗粒度转换接口设计，会使人直觉上认为RDD的功能很受限、不够强大。但是，实际上RDD已经被实践证明可以很好地应用于许多并行计算应用中，可以具备很多现有计算框架（比如MapReduce、SQL、Pregel等）的表达能力，并且可以应用于这些框架处理不了的交互式数据挖掘应用。

### Spark用Scala语言实现了RDD的API，程序员可以通过调用API实现对RDD的各种操作

## RDD执行流程

1. RDD读入外部数据源（或者内存中的集合）进行创建；
2. RDD经过一系列的“转换”操作，每一次都会产生不同的RDD，供给下一个“转换”使用；
3. 最后一个RDD经“行动”操作进行处理，并输出到外部数据源（或者变成Scala集合或标量）。
需要说明的是，==RDD采用了惰性调用==，即在RDD的执行过程中（如图9-8所示），真正的计算发生在RDD的“行动”操作，对于“行动”之前的所有“转换”操作，Spark只是记录下“转换”操作应用的一些基础数据集以及RDD生成的轨迹，即相互之间的依赖关系，而不会触发真正的计算。(==只是记录RDD依赖关系轨迹，只有真正行动才会运行==)

例如，从输入中逻辑上生成A和C两个RDD，经过一系列“转换”操作，逻辑上生成了F（也是一个RDD），之所以说是逻辑上，是因为这时候计算并没有发生，Spark只是记录了RDD之间的生成和依赖关系。当F要进行输出时，也就是当F进行“行动”操作的时候，Spark才会根据RDD的依赖关系生成DAG，并从起点开始真正的计算。

![image-20220420111700661](../图片/image-20220420111700661.png)

上述这一系列处理称为一个“血缘关系（Lineage），即DAG拓扑排序的结果。

采用惰性调用，通过血缘关系连接起来的一系列RDD操作就可以实现管道化（pipeline），避免了多次转换操作之间数据同步的等待，而且不用担心有过多的中间数据，==因为这些具有血缘关系的操作都管道化了==，==一个操作得到的结果不需要保存为中间数据，而是直接管道式地流入到下一个操作进行处理==。同时，这种通过血缘关系把一系列操作进行管道化连接的设计方式，也使得管道中每次操作的计算变得相对简单，保证了每个操作在处理逻辑上的单一性。

#### 举个例子：

这里以一个“Hello World”入门级Spark程序来解释RDD执行过程，这个程序的功能是读取一个HDFS文件，计算出包含字符串“Hello World”的行数。

```
val sc= new SparkContext(“spark://localhost:7077”,”Hello World”, “YOUR_SPARK_HOME”,”YOUR_APP_JAR”)
val fileRDD = sc.textFile(“hdfs://192.168.0.103:9000/examplefile”)
val filterRDD = fileRDD.filter(_.contains(“Hello World”))
filterRDD.cache()
filterRDD.count()//执行真正的计算
```

可以看出，一个Spark应用程序，基本是基于RDD的一系列计算操作。第1行代码用于创建SparkContext对象；第2行代码从HDFS文件中读取数据创建一个RDD；第3行代码对fileRDD进行转换操作得到一个新的RDD，即filterRDD；第4行代码表示对filterRDD进行持久化，把它保存在内存或磁盘中（这里采用cache接口把数据集保存在内存中），方便后续重复使用，当数据被反复访问时（比如查询一些热点数据，或者运行迭代算法），这是非常有用的，而且通过cache()可以缓存非常大的数据集，支持跨越几十甚至上百个节点；第5行代码中的count()是一个行动操作，用于计算一个RDD集合中包含的元素个数。这个程序的执行过程如下：
\*  创建这个Spark程序的执行上下文，即创建SparkContext对象；
\*  从外部数据源（即HDFS文件）中读取数据创建fileRDD对象；
\*  构建起fileRDD和filterRDD之间的依赖关系，形成DAG图，这时候并没有发生真正的计算，只是记录转换的轨迹；
\*  执行到第5行代码时，count()是一个行动类型的操作，触发真正的计算，开始实际执行从fileRDD到filterRDD的转换操作，并把结果持久化到内存中，最后计算出filterRDD中包含的元素个数。

## RDD特性

总体而言，Spark采用RDD以后能够实现高效计算的主要原因如下：
（1）高效的容错性。现有的分布式共享内存、键值存储、内存数据库等，为了实现容错，必须在集群节点之间进行数据复制或者记录日志，也就是在节点之间会发生大量的数据传输，这对于数据密集型应用而言会带来很大的开销。==在RDD的设计中，数据只读，不可修改==，如果需要修改数据，必须从父RDD转换到子RDD，由此在不同RDD之间建立了血缘关系。所以，RDD是一种天生具有容错机制的特殊集合，不需要通过数据冗余的方式（比如检查点）实现容错，而只需通过RDD父子依赖（血缘）关系重新计算得到丢失的分区来实现容错，无需回滚整个系统，这样就避免了数据复制的高开销，而且重算过程可以在不同节点之间并行进行，实现了高效的容错。此外，RDD提供的转换操作都是一些粗粒度的操作（比如map、filter和join），RDD依赖关系只需要记录这种粗粒度的转换操作，而不需要记录具体的数据和各种细粒度操作的日志（比如对哪个数据项进行了修改），这就大大降低了数据密集型应用中的容错开销；
（2）中间结果持久化到内存。数据在内存中的多个RDD操作之间进行传递，不需要“落地”到磁盘上，避免了不必要的读写磁盘开销；
（3）存放的数据可以是Java对象，避免了不必要的对象序列化和反序列化开销。

(==1.只读不可修改，只可转换2.中间结果保存在内存==)

## RDD之间的依赖关系

RDD中不同的操作会使得不同RDD中的分区会产生不同的依赖。RDD中的依赖关系分为==窄依赖（Narrow Dependency）与宽依赖（Wide Dependency）==，图9-10展示了两种依赖之间的区别。

窄依赖表现为一个父RDD的分区对应于一个子RDD的分区，或多个父RDD的分区对应于一个子RDD的分区；比如图9-10(a)中，RDD1是RDD2的父RDD，RDD2是子RDD，RDD1的分区1，对应于RDD2的一个分区（即分区4）；再比如，RDD6和RDD7都是RDD8的父RDD，RDD6中的分区（分区15）和RDD7中的分区（分区18），两者都对应于RDD8中的一个分区（分区21）。
宽依赖则表现为存在一个父RDD的一个分区对应一个子RDD的多个分区。比如图9-10(b)中，RDD9是RDD12的父RDD，RDD9中的分区24对应了RDD12中的两个分区（即分区27和分区28）。
总体而言，如果父RDD的一个分区只被一个子RDD的一个分区所使用就是窄依赖，否则就是宽依赖。

窄依赖典型的操作包括map、filter、union等，

宽依赖典型的操作包括groupByKey、sortByKey等。对于连接（join）操作，可以分为两种情况。

<img src="../../Library/Application Support/typora-user-images/image-20220420123553623.png" alt="image-20220420123553623" style="zoom: 33%;" />

在两种依赖关系中，窄依赖的失败恢复更为高效，它只需要根据父RDD分区重新计算丢失的分区即可（不需要重新计算所有分区），而且可以并行地在不同节点进行重新计算。而对于宽依赖而言，单个节点失效通常意味着重新计算过程会涉及多个父RDD分区，开销较大。

## 阶段的划分

Spark通过分析各个RDD的依赖关系生成了DAG，再通过分析各个RDD中的分区之间的依赖关系来决定如何划分阶段，具体划分方法是：在DAG中进行反向解析，遇到宽依赖就断开，遇到窄依赖就把当前的RDD加入到当前的阶段中；将窄依赖尽量划分在同一个阶段中，可以实现流水线计算（具体的阶段划分算法请参见AMP实验室发表的论文《Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing》）。例如，如图9-11所示，假设从HDFS中读入数据生成3个不同的RDD（即A、C和E），通过一系列转换操作后再将计算结果保存回HDFS。对DAG进行解析时，在依赖图中进行反向解析，由于从RDD A到RDD B的转换以及从RDD B和F到RDD G的转换，都属于宽依赖，因此，在宽依赖处断开后可以得到三个阶段，即阶段1、阶段2和阶段3。可以看出，在阶段2中，从map到union都是窄依赖，这两步操作可以形成一个流水线操作，比如，分区7通过map操作生成的分区9，可以不用等待分区8到分区9这个转换操作的计算结束，而是继续进行union操作，转换得到分区13，这样流水线执行大大提高了计算的效率。

<img src="../../Library/Application Support/typora-user-images/image-20220420130312899.png" alt="image-20220420130312899" style="zoom:33%;" />

图9-11根据RDD分区的依赖关系划分阶段

##### 由上述论述可知，把一个DAG图(按宽窄依赖)划分成多个“阶段”以后，每个阶段都代表了一组关联的、相互之间没有Shuffle依赖关系的任务组成的任务集合。每个任务集合会被提交给任务调度器（TaskScheduler）进行处理，由任务调度器将任务分发给Executor运行。

## RDD运行过程

通过上述对RDD概念、依赖关系和阶段划分的介绍，结合之前介绍的Spark运行基本流程，这里再总结一下RDD在Spark架构中的运行过程（如图9-12所示）：
（1）创建RDD对象；
（2）SparkContext负责计算RDD之间的依赖关系，构建DAG；
（3）DAGScheduler负责把DAG图分解成多个阶段，每个阶段中包含了多个任务，每个任务会被任务调度器分发给各个工作节点（Worker Node）上的Executor去执行。

==任务发放原则：计算向数据靠拢：假如有两台机器，AB，不是随便分配任务的，如果A上有数据则A处理，否则难道将A数据传输到B进行处理吗？==

![image-20220420130521367](../图片/image-20220420130521367.png)



<configuration>
	<property>
	    <name>yarn.nodemanager.aux-services</name>
	    <value>mapreduce_shuffle</value>
	  </property>
	<property>
	    <name>yarn.resourcemanager.address</name>
	    <value>localhost:9000</value>
	</property> 
	<property>
	  <name>yarn.scheduler.capacity.maximum-am-resource-percent</name>
	  <value>100</value>
	</property>
</configuration>

/usr/local/Cellar/hadoop/3.3.1/libexec/sbin