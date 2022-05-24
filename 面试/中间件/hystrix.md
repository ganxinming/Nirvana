## Hystrix

##### Hystrix是一个库，在分布式环境下不可避免的会发生服务依赖失败。Hystrix可以通过添加等待时间容限和容错的逻辑来提高系统的弹性。从而对延迟和故障提供更强大的容错能力，提供了熔断、隔离、Fallback、cache（hystrix支持将一个请求结果缓存起来，下一个具有相同key的请求将直接从缓存中取出结果，减少请求开销）、监控，请求合并(足够短的时间内，合并多个相同请求)等功能。

看上去没有必要，但是仔细算一算发现还是很有必要的。

一个服务依赖其他的30个服务，假设失败率99.99%，那么一个服务的成功率是99.99%^30=99.7%。

那么意味着，1亿次有3000的失败。



## 使用过程：

#### 1.自定义抽象类AbstractIsolationThreadPool

```
public abstract class AbstractIsolationThreadPool<T> extends HystrixCommand<T>
```

#### 2.初始化的时候设置这五个参数，这五个参数是Setter基本参数。

1.设置GroupKey(代表了某一个下游依赖服务，一个服务有多个接口)，

2.CommandKey(是作为依赖命名,代表了一类 command，一般来说，代表了下游依赖服务的某个接口。)，

3.ThreadPoolKey(保证不同的服务之间，线程池隔离，代表了一个 HystrixThreadPool)。默认的 ThreadPoolKey 就是 command group 的名称。

(**推荐根据一个服务区划分出一个线程池，command key 默认都是属于同一个线程池的。即某个服务划分，而不是某个服务的各个接口换分**)

4.设置CommandProperties

5.设置ThreadPoolProperties

(其实这三个参数就是保证服务标识唯一，并进行分组管理)

```
//name:可以是我们自定义服务类型的名字，主要用于保证唯一(比如：Hbase，ES啊)
//HystrixThreadPoolConfig：自定义配置类，自定义的配置。
public AbstractIsolationThreadPool(String name, HystrixThreadPoolConfig poolConfig) {
    super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey(name + SUFFIX_GROUP_KEY))
            .andCommandKey(HystrixCommandKey.Factory.asKey(name + SUFFIX_COMMAND_KEY))
            .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey(name + SUFFIX_THREAD_POOL_KEY))
            .andCommandPropertiesDefaults(
                   
//设置配置CommandProperties,主要是开启熔断，超时时间等         HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(poolConfig.getExecutionTimeOut())
                            .withF        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency> allbackEnabled(true)
                            //fallBack最大并发数
                            .withFallbackIsolationSemaphoreMaxConcurrentRequests(poolConfig.getFallBackMaxConRequest())
                            //线程隔离
                            .withExecutionIsolationStrategy(HystrixCommandProperties.ExecutionIsolationStrategy.THREAD)
                            //熔断器配置
                            .withCircuitBreakerEnabled(true)
                            .withCircuitBreakerRequestVolumeThreshold(poolConfig.getCircuitBreakerRequestVolumeThreshold())
                            .withCircuitBreakerErrorThresholdPercentage(poolConfig.getCircuitBreakerErrorThresholdPercent())
                            .withCircuitBreakerSleepWindowInMilliseconds(poolConfig.getCircuitBreakerSleepWindowTime())
                            .withMetricsRollingStatisticalWindowInMilliseconds(poolConfig.getMetricsRollingStatisticWindowTime())
            )
            //设置线程池的配置
            .andThreadPoolPropertiesDefaults(
                    //线程池配置
                    HystrixThreadPoolProperties.Setter()
                            .withCoreSize(poolConfig.getCorePoolSize())
                            .withMaximumSize(poolConfig.getMaxPoolSize())
                            .withMaxQueueSize(poolConfig.getMaxQueueSize())
                            .withKeepAliveTimeMinutes(poolConfig.getKeepAliveTime())
                            .withQueueSizeRejectionThreshold(poolConfig.getQueueRejectSize())
                            .withAllowMaximumSizeToDivergeFromCoreSize(true))
    );
```

3.重写run方法。这里执行run方法前后算法骨架定义完成。所有子类继承该类后，重写doExecute()。(只有run是规定执行方法，before，after，doExecute都是人为定义的，因为每次执行的前置，后置处理一样，所以抽象出来了)

```
@Override
protected T run() throws Exception {
    Exception error = null;
    beforeExecute();
    try {
        return doExecute();
    } catch (Exception e) {
        error = e;
        throw e;
    } finally {
        monitorLogger.info(Joiner.on("|").join("risk-mihuan",
                AbstractIsolationThreadPool.this.getClass().getName(), "get",
                System.currentTimeMillis() - EXECUTE_TIME_WATCHER.get(),
                Objects.nonNull(error) ? ResultEnum.EXCEPTION.code : ResultEnum.NORMAL.code));
        afterExecute();
    }
}

/**
 * 具体任务执行
 *
 * @return 执行结果
 * @throws Exception
 */
protected abstract T doExecute() throws Exception;

protected void beforeExecute() {
    TraceContextUtils.setLocalTraceContext(traceContext);

    EXECUTE_TIME_WATCHER.set(System.currentTimeMillis());
}

protected void afterExecute() {
    EXECUTE_TIME_WATCHER.remove();
}

/**
 * 降级回调
 *
 * @return 降级返回
 */
@Override
protected abstract T getFallback();
```

4.实体类继承抽象类，重写doExecute()，fallback()方法。

```
@Override
    protected Map<String, Object> doExecute() throws Exception {

        Map<String, Object> resultMap;
        //直接执行codis取数据服务
        BaseStorageDataObtainService storageDataObtainService = FeatureRequestUtils.getStorageDataObtainService(StorageConstants.CODIS);
        resultMap = storageDataObtainService.batchObtainFeatureData(dataRequestList);

        long timeConsume = System.currentTimeMillis() - EXECUTE_TIME_WATCHER.get();
        logger.info("CodisThreadPool run consume:{}", timeConsume);
        if (timeConsume > ConfigUtils.getCodisSlowRunLogThreshold()) {
            logger.info("CodisThreadPool run consume over threshold,{},{}", timeConsume, dataRequestList);
        }
        return resultMap;
    }

    @Override
    protected Map<String, Object> getFallback() {
        logger.warn("CodisThreadPool fall back,dataRequest:{},circuitBreaker is open:{},failedException:{},executionException:{}",
                dataRequestList, this.isCircuitBreakerOpen(), this.getFailedExecutionException(), this.getExecutionException());
        return FeatureRequestUtils.failMap(dataRequestList);
    }
}
```

## 原理：

Hystrix 通过将依赖服务(被调用的那些服务)进行**资源隔离**，进而阻止某个依赖服务出现故障时在整个系统所有的依赖服务调用中进行蔓延；同时 Hystrix 还提供故障时的 fallback 降级机制(限流)。

(作用：通过资源隔离+限流，服务降级和熔断**提升分布式系统的可用性和稳定性**)

### Hystrix 更加细节的设计原则

- 阻止任何一个依赖服务耗尽所有的资源，比如 tomcat 中的所有线程资源。
- 避免请求排队和积压，采用限流和 `fail fast` 来控制故障。(限流)
- 提供 fallback 降级机制来应对故障。
- 使用资源隔离技术，比如 `bulkhead`（舱壁隔离技术）、`swimlane`（泳道技术）、`circuit breaker`（断路技术）来限制任何一个依赖服务的故障的影响。
- 通过近实时的统计/监控/报警功能，来提高故障发现的速度。
- 通过近实时的属性和配置**热修改**功能，来提高故障处理和恢复的速度。
- 保护依赖服务调用的所有故障情况，而不仅仅只是网络故障情况。

### 资源隔离实现：

#### 线程池模式(默认)

继承HystrixCommand类后，实现run方法调用服务。那么每次调用服务，就只会用该线程池中的资源，不会再去用其它线程资源了。缓存服务默认的线程大小是 10 个，最多就只有 10 个线程去调用商品服务的接口。即使商品服务接口故障了，最多就只有 10 个线程会 hang 死在调用商品服务接口的路上，缓存服务的 Tomcat 内其它的线程还是可以用来调用其它的服务，干其它的事情。(不会出现大量线程调用某个服务，没有结果，而导致其他线程资源(如：tomcat)枯竭)

(因为Hystrix默认使用了线程池模式，所以对于每个Command，在初始化的时候，会创建一个对应的线程池，不了解配置的话，按照默认配置直接使用，可能就会造成线程资源的大量浪费)

##### Hystrix有两个请求命令 HystrixCommand、HystrixObservableCommand。

　　HystrixCommand用在依赖服务返回单个操作结果的时候。又两种执行方式

　　  -execute():同步执行。从依赖的服务返回一个单一的结果对象，或是在发生错误的时候抛出异常。

　　  -queue();异步执行。直接返回一个Future对象，其中包含了服务执行结束时要返回的单一结果对象。

　　HystrixObservableCommand 用在依赖服务返回多个操作结果的时候。它也实现了两种执行方式

　　  -observe():返回Obervable对象，他代表了操作的多个结果，他是一个HotObservable

　　  -toObservable():同样返回Observable对象，也代表了操作多个结果，但它返回的是一个Cold Observable。

##### 限流：如果线程池的所有线池都满了，请求进入队列积压。如果队列满了，再有请求过来直接fallback降级。

原理:采用了 舱壁隔离技术，来将外部依赖进行资源隔离，进而避免任何外部依赖的故障导致本服务崩溃。对每个外部依赖用一个单独的线程池，这样的话，如果对那个外部依赖调用延迟很严重，最多就是耗尽那个依赖自己的线程池而已，不会影响其他的依赖调用。

#### 信号量模式

接收请求和执行下游依赖在同一个线程内完成，**不存在线程上下文切换**所带来的性能开销。每次调用线程，当前请求通过计数信号量进行限制，当信号大于了最大请求数（maxConcurrentRequests）时，进行限制，调用fallback接口快速返回。信号量的调用是**同步的**，也就是说，**每次调用都得阻塞调用方的线程**，直到结果返回。这样就导致了无法对访问做超时（只能依靠调用协议超时，无法主动释放）

信号量的资源隔离只是起到一个开关的作用，比如，服务 A 的信号量大小为 10，那么就是说它同时只允许有 10 个 tomcat 线程来访问服务 A，其它的请求都会被拒绝，从而达到资源隔离和限流保护的作用。

(不适用于服务中调用多个下游服务，因为是同步的，所以耗时会很长。)

##### 限流：隔离策略的时候允许访问的最大并发量，超过这个最大并发量，请求直接被 reject，fallback。

##### 线程池隔离技术，是用 Hystrix 自己的线程去执行调用(就是我们定义的默认10个线程)；而信号量隔离技术，是直接让 tomcat 线程去调用依赖服务(使用接受请求时tomcat请求线程)。

| 隔离方式   | 是否支持超时                                                 | 是否支持熔断                                                 | 隔离原理             | 是否是异步调用                        | 资源消耗                                    |
| :--------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------- | :------------------------------------ | :------------------------------------------ |
| 线程池隔离 | 支持,可直接返回                                              | 支持,当线程池到达maxSize后,再请求会触发fallback接口进行熔断  | 每个服务单独用线程池 | 可以是异步,也可以是同步。看调用的方法 | 大,大量线程的上下文切换，容易造成机器负载高 |
| 信号量隔离 | 不支持,如果阻塞，只能通过调用协议（如:socket超时才能返回，这就相当于不支持自定义调用超时,一旦block住，就很容易导致后面限流） | 支持，当信号量达到maxConcurrentRequests后。再请求会触发fallback | 通过信号量的计数器   | 同步调用,不支持异步                   | 小,只是个计数器                             |



## 执行原理

![E2606A52-ECA7-4239-94DB-A4E52F9CFDBE](../../../Documents/TyporaBlogMAC/图/E2606A52-ECA7-4239-94DB-A4E52F9CFDBE.png)

#### 1.创建command。

#### 2.调用执行方法(),根据继承的对象，同步和异步可以使用以下四种方法。

- execute()：调用后直接 block 住，属于同步调用，直到依赖服务返回单条结果，或者抛出异常。
- queue()：返回一个 Future，属于异步调用，后面可以通过 Future 获取单条结果。
- observe()：订阅一个 Observable 对象，Observable 代表的是依赖服务返回的结果，获取到一个那个代表结果的 Observable 对象的拷贝对象。
- toObservable()：返回一个 Observable 对象，如果我们订阅这个对象，就会执行 command 并且获取返回结果。

#### 3.检查缓存是否开启(常用)

#### 4.检查断路器，如果开启直接走fallback。

#### 5.检查线程池队列，信号是否满了，满了执行fallback降级机制，同时发送reject 信息给断路器统计。

#### 6.执行command方法(有两个执行方法)

 HystrixObservableCommand 对象的 construct() 方法，或者 HystrixCommand 的 run() 

#### 断路器检查

Hystrix 会把每一个依赖服务的调用成功、失败、Reject、Timeout 等事件发送给 circuit breaker 断路器。断路器就会对这些事件的次数进行统计，根据异常事件发生的比例来决定是否要进行断路（熔断）。如果打开了断路器，那么在接下来一段时间内，会直接断路，返回降级结果。

#### 调用fallback

大部分步骤都有可能进行fallback，综合如下：

##### 1.线程池队列满了。2.信号量满了3.断路器打开了4.超时5.异常



## 断路器(触发熔断)

Hystrix 断路器有三种状态，分别是关闭（Closed）、打开（Open）与半开（Half-Open）

1. `Closed` 断路器关闭：调用下游的请求正常通过
2. `Open` 断路器打开：阻断对下游服务的调用，直接走 Fallback 逻辑
3. `Half-Open` 断路器处于半开状态：SleepWindowInMilliseconds(如果有一条请求成功通过，则关闭断路器。[说明此事服务恢复正常]，否则断路器继续关闭)

### Enabled

```java
HystrixCommandProperties.Setter()
    .withCircuitBreakerEnabled(boolean)
```

控制断路器是否工作，包括跟踪依赖服务调用的健康状况，以及对异常情况过多时是否允许触发断路。默认值 `true`。

### circuitBreaker.requestVolumeThreshold

```java
HystrixCommandProperties.Setter()
    .withCircuitBreakerRequestVolumeThreshold(int)
```

表示在一次统计的**时间滑动窗口中（这个参数也很重要，下面有说到）**，至少经过多少个请求，才可能触发断路，默认值 20。**经过 Hystrix 断路器的流量只有在超过了一定阈值后，才有可能触发断路。**比如说，要求在 10s 内经过断路器的流量必须达到 20 个，而实际经过断路器的请求有 19 个，即使这 19 个请求全都失败，也不会去判断要不要断路。

### circuitBreaker.errorThresholdPercentage

```java
HystrixCommandProperties.Setter()
    .withCircuitBreakerErrorThresholdPercentage(int)
```

表示异常比例达到多少，才会触发断路，默认值是 50(%)。

#### circuitBreaker.sleepWindowInMilliseconds

```java
HystrixCommandProperties.Setter()
    .withCircuitBreakerSleepWindowInMilliseconds(int)
```

断路器状态由 Close 转换到 Open，在之后 `SleepWindowInMilliseconds` 时间内，所有经过该断路器的请求会被断路，不调用后端服务，直接走 Fallback 降级机制，默认值 5000(ms)。

而在该参数时间过后，断路器会变为 `Half-Open` 半开闭状态，尝试让一条请求经过断路器，看能不能正常调用。如果调用成功了，那么就自动恢复，断路器转为 Close 状态。

### ForceOpen

```java
HystrixCommandProperties.Setter()
    .withCircuitBreakerForceOpen(boolean)
```

如果设置为 true 的话，直接强迫打开断路器，相当于是手动断路了，手动降级，默认值是 `false`。

### ForceClosed

```java
HystrixCommandProperties.Setter()
    .withCircuitBreakerForceClosed(boolean)
```

如果设置为 true，直接强迫关闭断路器，相当于手动停止断路了，手动升级，默认值是 `false`。

(总结：1.可人工手动控制断路器开关。2.配置时间滑动窗口，配置时间内超过数量请求，**才可能触发短路**3.配置异常比例，达到比例触发短路4.触发断路后，配置时间段内都默认断路直接走fallback。时间过后变成half open状态)

总的来说断路器触发条件:配置时间内超过数量请求，才会判断是否执行断路，根据异常比例判断是否真正执行断路。



## 超时：

对于超时的服务就应该让他fallback，(超时很多时候是调动其他的服务本身不稳定，不能因为他们而导致本服务出错)

在 Hystrix 中，我们可以手动设置 timeout 时长，如果一个 command 运行时间超过了设定的时长，那么就被认为是 timeout，然后 Hystrix command 标识为 timeout，同时执行 fallback 降级逻辑。

`TimeoutMilliseconds` 默认值是 1000，也就是 1000ms。

```java
HystrixCommandProperties.Setter()
    ..withExecutionTimeoutInMilliseconds(int)
```

### TimeoutEnabled

这个参数用于控制是否要打开 timeout 机制，默认值是 true。

```java
HystrixCommandProperties.Setter()
    .withExecutionTimeoutEnabled(boolean)
```



## 结果缓存：

hystrix支持将一个请求结果缓存起来，下一个具有相同key的请求将直接从缓存中取出结果，减少请求开销。要使用hystrix cache功能，第一个要求是重写`getCacheKey()`，用来构造cache key；第二个要求是构建context，如果请求B要用到请求A的结果缓存，A和B必须同处一个context。通过`HystrixRequestContext.initializeContext()`和`context.shutdown()`可以构建一个context，这两条语句间的所有请求都处于同一个context。

以[demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fstar2478%2Fjava-hystrix%2Fblob%2Fmaster%2Fsrc%2Ftest%2Fjava%2Fcom%2Fpingan%2Ftest%2Fspringbootdemo%2FHystrixCommand4RequestCacheTest.java)的`testWithCacheHits()`为例，*command2a*、*command2b*、*command2c*同处一个context，前两者的cache key都是`2HLX`（见`getCacheKey()`），所以*command2a*执行完后把结果缓存，*command2b*执行时就不走`run()`而是直接从缓存中取结果了，而*command2c*的cache key是`2HLX1`，无法从缓存中取结果。此外，通过`isResponseFromCache()`可检查返回结果是否来自缓存。





## 合并请求collapsing：

hystrix支持N个请求自动合并为一个请求，这个功能在有网络交互的场景下尤其有用，比如每个请求都要网络访问远程资源，如果把请求合并为一个，将使多次网络交互变成一次，极大节省开销。重要一点，两个请求能自动合并的前提是两者足够“近”，即两者启动执行的间隔时长要足够小，默认为10ms，即超过10ms将不自动合并。

以[demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fstar2478%2Fjava-hystrix%2Fblob%2Fmaster%2Fsrc%2Ftest%2Fjava%2Fcom%2Fpingan%2Ftest%2Fspringbootdemo%2FHystrixCommand4RequestCollapsingTest.java)为例，我们连续发起多个queue请求，依次返回*f1~f6*共6个Future对象，根据打印结果可知*f1~f5*同处一个线程，说明这5个请求被合并了，而*f6*由另一个线程执行，这是因为*f5*和*f6*中间隔了一个sleep，超过了合并要求的最大间隔时长。



# 降级参数

使用HystrixCommandProperties配置和getFallback()方法可以实现降级处理。下面详细介绍一下配置参数：

- withFallbackEnabled：是否启用降级，若启用，则在超时或异常时调用getFallback进行降级。(默认开启)
- withFallbackIsolationSemaphoreMaxConcurrentRequests：配置了fallback()请求并发的信号量，当调用fallback()的并发超过阀值(默认10)，则会进入快速失败。
- withExecutionIsolationThreadInterruptOnFutureCancel：当隔离策略为THREAD时，当线程执行超时，是否进行中断处理，即异步的Future#cancel()。(默认为false)
- withExecutionIsolationThreadInterruptOnTimeout：当隔离策略为THREAD时，当线程执行超时，是否进行中断处理。(默认为true)这里指的是同步调用：execute()
- withExecutionTimeoutEnabled：是否启用超时机制，默认为true。
- withExecutionTimeoutInMilliseconds：执行超时时间，默认1000毫秒。1、配置线程隔离，则执行中断处理；2、配置信号量隔离，则进行终止操作。因为信号量隔离和主线程是在一个线程中执行，其不会中断线程处理。所以要根据实际情况选择类型。

除了上面的部分参数，对于getFallback()还需要注意以下的几点：

1. 最大并发数受fallbackIsolationSemaphoreMaxConcurrentRequests控制，如果失败率非常高，则需要重新配置该参数。如果并发数超过了该配置，则不会再执行getFallback()，而是快速失败。如抛出HystrixRuntimeException的异常。
2. 该方法不能进行网络调用，应该只是返回兜底的数据。
3. 如果必须要走一个网络调用，则就需要调用另外一个Command。
4. Command可以有降级和熔断机制，而getFallback只有fallbackIsolationSemaphoreMaxConcurrentRequest参数控制最大并发数。

# 熔断

Command首先调用HystrixCircuitBreaker#allowRequest判断是否熔断了，如果没有则执行Command#run方法；若熔断了则直接调用Command#getFallback方法降级处理。

通过circuitBreakerSleepWindowInMilliseconds可以控制一个时间窗口内，可进行一次请求测试。若测试成功，则闭合熔断开关，否则还是打开状态，从而实现了快速失败和恢复。关于熔断有以下几个概念需要了解一下：

**概念**

- 闭合(Closed)：如果配置了熔断开关强制闭合，或者当前请求失败率没有超过阀值，则熔断开关处于闭合状态，此时不会启动熔断机制，即不进行降级处理。
- 打开(Open)：如果配置了熔断开关强制打开，或者当前失败率超过了阀值，则熔断开关打开，此时会调用getFallback()方法进行降级处理。
- 半打开(Half-Open)：当熔断处于打开状态后，不能一直熔断下去，需要在一个时间窗口之后进行重试，这就是半打开状态。Hystrix允许在circuitBreakerSleepWindowInMilliseconds的时间窗口内进行一次重试。重试成功后闭合熔断开关，否则熔断开关还是处于打开状态。

上面所指的失败包含：异常、超时、线程池拒绝、信号量拒绝的总和。

**配置示例**

```
HystrixCommandProperties.Setter commandProperties = HystrixCommandProperties.Setter() .withCircuitBreakerEnabled(true)// 默认为true .withCircuitBreakerForceClosed(false)// 默认为false .withCircuitBreakerForceOpen(false)// 默认为false .withCircuitBreakerErrorThresholdPercentage(50)// 默认50% .withCircuitBreakerRequestVolumeThreshold(20)// 默认为20 .withCircuitBreakerSleepWindowInMilliseconds(5000)// 默认5秒
```

- withCircuitBreakerEnabled：是否开启熔断机制，默认为true。
- withCircuitBreakerForceClosed：是否强制关闭熔断开关，如果强制关闭了熔断开关，则请求不会被降级，一些场景可以动态设置该开关，默认为false。
- withCircuitBreakerForceOpen：是否强制打开熔断开关，如果打开了，则请求强制降级调用getFallback处理，可以通过动态配置来打开开关实现一些特殊需求，默认为false。
- withCircuitBreakerErrorThresholdPercentage：如果在一个采样时间窗口内，失败率超过该配置，则自动打开熔断开关，快速失败。默认采样周期为10秒，失败率为50%。
- withCircuitBreakerRequestVolumeThreshold：在熔断开关闭合的情况下，在进行失败率判断之前，一个采样周期内必须进行至少N个请求才能进行采样统计。目的是有足够的采样使得失败率计算的比较接近真实值，默认为20.
- withCircuirBreakerSleepWindowInMilliseconds：熔断后的重试时间窗口，在窗口内只允许一次重试。在熔断开关打开后，若重试成功，则重试Health采样统计，并闭合熔断开关实现快速恢复。否则熔断开关还是打开状态，会进行快速失败。

通过下面的方法可以获取熔断器的状态：

- isCircuitBreakerOpen：熔断开关是否打开了，通过 circuitBreakerForceOpen().get() || (!circuitBreakerForceClosed().get() && circuitBreaker.isOpen()) 判断。
- isResponseShortCircuited：isCircuitBreakerOpen=true，且调用getFallback()时返回true。



# 降级和熔断

抛开hystrix，常说的降级：是指，快速失败，不调用服务了。熔断：是指超时，异常，超过线程池大小等，之后进行失败(实际上还是会调用服务)。

但是在hystrix中，通过他的描述，可以判定断路器打开了，就是我们常说的降级，快速失败了。所以在hystrix中，熔断器打开了就是降级，而他定义的降级，可以认为是熔断。

