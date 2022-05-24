# Windows

目前有许多数据分析的场景从批处理到流处理的演变， 虽然可以将批处理作为流处理的特殊情况来处理，但是分析无穷集的流数据通常需要思维方式的转变并且具有其自己的术语（例如，“windowing（窗口化）”、“at-least-once（至少一次）”、“exactly-once（只有一次）” ）。

### 它有什么作用？

通常来讲，Window 就是用来对一个无限的流设置一个有限的集合，在有界的数据集上进行操作的一种机制。window 又可以分为基于时间（Time-based）的 window 以及基于数量（Count-based）的 window。

(基于时间和数量划分窗口)

### Flink 自带的 window

Flink DataStream API 提供了 Time 和 ==Count 的 window==，同时增加了基于 Session 的 window。同时，由于某些特殊的需要，DataStream API 也提供了定制化的 window 操作，供用户自定义 window。

下面，主要介绍 Time-Based window 以及 Count-Based window，以及自定义的 window 操作，Session-Based Window 操作将会在后续的文章中讲到。

#### Time Windows

正如命名那样，Time Windows 根据时间来聚合流数据。例如：一分钟的 tumbling time window 收集一分钟的元素，并在一分钟过后对窗口中的所有元素应用于一个函数。

在 Flink 中定义 tumbling time windows(翻滚时间窗口) 和 sliding time windows(滑动时间窗口) 非常简单：

**tumbling time windows(翻滚时间窗口)**

输入一个时间参数

```
data.keyBy(1)
	.timeWindow(Time.minutes(1)) //tumbling time window 每分钟统计一次数量和
	.sum(1);
```

**sliding time windows(滑动时间窗口)**

输入两个时间参数

```
data.keyBy(1)
	.timeWindow(Time.minutes(1), Time.seconds(30)) //sliding time window 每隔 30s 统计过去一分钟的数量和
	.sum(1);
```

==区别：滚动窗口：只在窗口结束空档期执行一次操作，滑动窗口：执行时间小于滑动窗口，这样的话每次执行，窗口之间都可能有重叠==

有一点我们还没有讨论，即“收集一分钟的元素”的确切含义，它可以归结为一个问题，“流处理器如何解释时间?”

Apache Flink 具有三个不同的时间概念，即 processing time, event time 和 ingestion time。

#### Count Windows

Apache Flink 还提供计数窗口功能。如果计数窗口设置的为 100 ，那么将会在窗口中收集 100 个事件，并在添加第 100 个元素时计算窗口的值。

在 Flink 的 DataStream API 中，tumbling count window 和 sliding count window 的定义如下:

**tumbling count window**

输入一个时间参数

```
data.keyBy(1)
	.countWindow(100) //统计每 100 个元素的数量之和
	.sum(1);
```

**sliding count window**

输入两个时间参数

```
data.keyBy(1) 
	.countWindow(100, 10) //每 10 个元素统计过去 100 个元素的数量之和
	.sum(1);
```

### 如何自定义 Window？

1、Window Assigner

负责将元素分配到不同的 window。

Window API 提供了自定义的 WindowAssigner 接口，我们可以实现 WindowAssigner 的

```
public abstract Collection<W> assignWindows(T element, long timestamp)
```

方法。同时，对于基于 Count 的 window 而言，默认采用了 GlobalWindow 的 window assigner，例如：

```
keyBy.window(GlobalWindows.create())
```

2、Trigger

Trigger 即触发器，定义何时或什么情况下移除 window

我们可以指定触发器来覆盖 WindowAssigner 提供的默认触发器。 请注意，指定的触发器不会添加其他触发条件，但会替换当前触发器。

3、Evictor（可选）

驱逐者，即保留上一 window 留下的某些元素

4、通过 apply WindowFunction 来返回 DataStream 类型数据。

利用 Flink 的内部窗口机制和 DataStream API 可以实现自定义的窗口逻辑，例如 session window。