### 什么是 parallelism？(并行度)

如翻译这样，parallelism 是并行的意思，在 Flink 里面代表每个任务的并行度，适当的提高并行度可以大大提高 job 的执行效率，比如你的 job 消费 kafka 数据过慢，适当调大可能就消费正常了。

#### 如何设置 parallelism？

如上图，在 flink 配置文件中可以查看到默认并行度是 1，

```
cat flink-conf.yaml | grep parallelism

# The parallelism used for programs that did not specify and other parallelism.
parallelism.default: 1
```

#### 命令启动

如果你是用命令行启动你的 Flink job，那么你也可以这样设置并行度(使用 -p 并行度)：

```
./bin/flink run -p 10 ../word-count.jar
```

#### 代码设置

你也可以通过这样来设置你整个程序的并行度：注意：这样设置的并行度是你整个程序的并行度，那么后面如果你的每个算子不单独设置并行度覆盖的话，那么后面每个算子的并行度就都是这里设置的并行度的值了。

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(10);
```

如何给每个算子单独设置并行度呢？

```
data.keyBy(new xxxKey())
    .flatMap(new XxxFlatMapFunction()).setParallelism(5)
    .map(new XxxMapFunction).setParallelism(5)
    .addSink(new XxxSink()).setParallelism(1)
```

如上，就是在每个算子后面单独的设置并行度，这样的话，就算你前面设置了 env.setParallelism(10) 也是会被覆盖的。

==这也说明优先级是：算子设置并行度 > env 设置并行度 > 配置文件默认并行度==

# 什么是 slot？

![image-20220125153409898](../../Desktop/TyporaBlogMAC/图/image-20220125153409898.png)

图中 Task Manager 是从 Job Manager 处接收需要部署的 Task，任务的并行性由每个 Task Manager 上可用的 slot 决定。每个任务代表分配给任务槽的一组资源，slot 在 Flink 里面可以认为是资源组，Flink 将每个任务分成子任务并且将这些子任务分配到 slot 来并行执行程序。

例如，如果 Task Manager 有四个 slot，那么它将为每个 slot 分配 25％ 的内存。 可以在一个 slot 中运行一个或多个线程。 同一 slot 中的线程共享相同的 JVM。 同一 JVM 中的任务共享 TCP 连接和心跳消息。Task Manager 的一个 Slot 代表一个可用线程，该线程具有固定的内存，注意 Slot 只对内存隔离，没有对 CPU 隔离。默认情况下，Flink 允许子任务共享 Slot，即使它们是不同 task 的 subtask，只要它们来自相同的 job。这种共享可以有更好的资源利用率。

#### 总结：

1.一个solt分配一个线程或多个线程。(因为cpu分配不隔离)

2.每个solt平均分配内存(内存分配隔离)

3.同一 slot 中的线程共享相同的 JVM

4.Flink 允许子任务(Task Manager)共享 Slot，即使它们是不同 task 的 subtask，只要它们来自相同的 job。这种共享可以有更好的资源利用率。



![image-20220125154254183](../../Desktop/TyporaBlogMAC/图/image-20220125154254183.png)

上面图片中有两个 Task Manager，每个 Task Manager 有三个 slot，这样我们的算子最大并行度那么就可以达到 6 个，在同一个 slot 里面可以执行 1 至多个子任务。

那么再看上面的图片，source/map/keyby/window/apply 最大可以有 6 个并行度，sink 只用了 1 个并行。

每个 Flink TaskManager 在集群中提供 slot。 slot 的数量通常与每个 TaskManager 的可用 CPU 内核数成比例。

==一般情况下你的 slot 数是你每个 TaskManager 的 cpu 的核数。==

#### 下面给出官方的图片来更加深刻的理解下 slot：

1、slot 是指 taskmanager 的并发执行能力

![image-20220125155601774](../../Desktop/TyporaBlogMAC/图/image-20220125155601774.png)

每一个 taskmanager 中的分配 3 个 TaskSlot, 3 个 taskmanager 一共有 9 个 TaskSlot。

2、parallelism 是指 taskmanager 实际使用的并发能力

![image-20220125155643427](../../Desktop/TyporaBlogMAC/图/image-20220125155643427.png)

parallelism.default:1

运行程序默认的并行度为 1，9 个 TaskSlot 只用了 1 个，有 8 个空闲。设置合适的并行度才能提高效率。



