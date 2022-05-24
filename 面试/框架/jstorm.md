## Jstorm

### 一、基本术语

#####  Stream

　　在JStorm当中，有对Stream的抽象，它是一个不间断的无界的连续Tuple，而JStorm在建模事件流时，把流中的事件抽象成Tuple。

##### Spout和Bolt

　　在JStorm中，它认为每个Stream都有一个Stream的来源，即Tuple的源头，所以它将这个源头抽象为Spout，而Spout可能是一个消息中间件，如：MQ，Kafka等。并不断的发出消息，也可能是从某个队列中不断读取队列的元数据。

![截屏2020-08-10 上午10.23.21](../../../Library/Application Support/typora-user-images/截屏2020-08-10 上午10.23.21.png)

　　在有了Spout后，接下来如何去处理相关内容，以类似的思想，将JStorm的处理过程抽象为Bolt，Bolt可以消费任意数量的输入流， 只要将流方向导到该Bolt即可，同时，它也可以发送新的流给其他的Bolt使用，因而，我们只需要开启特定的Spout，将Spout流出的Tuple 导向特定的Bolt，然后Bolt对导入的流做处理后再导向其它的Bolt等。

![截屏2020-08-10 上午10.23.05](../../../Library/Application Support/typora-user-images/截屏2020-08-10 上午10.23.05.png)

其实，我们可以用一个形象的比喻来理解这个流程。我们可以认为Spout就是一个个的水龙头，并且每个水龙头中的水是不同 的，我们想要消费那种水就去开启对应的水龙头，然后使用管道将水龙头中的水导向一个水处理器，即Bolt，水处理器处理完后会再使用管道导向到另外的处理 器或者落地到存储介质

##### Topology

![image-20200810102607645](../../../Library/Application Support/typora-user-images/image-20200810102607645.png)

如图所示，这是一个有向无环图，JStorm将这个图抽象为Topology，它是JStorm中最高层次的一个抽象概念，它可以处理代码层面 当中直接于JStorm打交道的，可以被提交到JStorm集群执行对应的任务，一个Topology即为一个数据流转换图，图中的每个节点是一个 Spout或者Bolt，当Spout或Bolt发送Tuple到流时，它就发送Tuple到每个订阅了该流的Bolt上。

##### Tuple

一个tuple可以简单的理解为一系列的的键值对(key-value pairs)，是storm结构中最小的数据单元

　　JStorm当中将Stream中数据抽象为了Tuple，一个Tuple包 含 了 一 个 或 者 多 个 键 值 对 的 列 表，List值的每个Value都有一个Name，并且该Value可以是基本类型，字符类型，字节数组等，当然也可以是其它可序列化的类型。 Topology的每个节点都要说明它所发射出的Tuple的字段的Name，其它节点只需要订阅该Name就可以接收处理相应的内容。

**Bolt**

bolts可以想象成计算的操作者或者是一个函数，他们可以接收任意的数据流或者被处理过的数据，而且还可以随意的发送一个或多个tuples，bolts可以订阅spouts或者是其他bolts发送过来的数据流，bolts可以创造一个复杂的数据传输网络。bolts的典型作用如下：1、过滤tuples；2、连接或者是聚合；3、计算

##### Worker和Task

　　Work和Task在JStorm中的职责是一个执行单元，**一个Worker表示一个进程，一个Task表示一个线程**，一个Worker可以运 行多个Task。而Worker可以通过setNumWorkers(int workers)方法来设置对应的数目，表示这个Topology运行在多个JVM（PS：一个JVM为一个进程，即一个Worker）；另外 setSpout(String id, IRichSpout spout, Number parallelism_hint)和setBolt(String id, IRichBolt bolt,Number parallelism_hint)方法中的参数parallelism_hint代表这样一个Spout或Bolt有多少个实例，即对应多少个线程，一 个实例对应一个线程。

总结：一个topology可以通过*setNumWorkers来设置worker的数量，通过设置parallelism来规定executor的数量（一个component（spout/bolt）可以由多个executor来执行），通过\*setNumTasks来设置每个executor跑多少个task（默认为一对一）。

(即：topology才能设置worker，spout和bolt设置task)

***\*task是spout和bolt执行的最小单元。\****

##### Slot

　　在JStorm当中，Slot的类型分为四种，他们分别是：CPU，Memory，Disk，Port；与Storm有所区别（Storm局限 于Port）。一个Supervisor可以提供的对象有：CPU Slot、Memory Slot、Disk Slot以及Port Slot。

- 在JStorm中，一个Worker消耗一个Port Slot，默认一个Task会消耗一个CPU Slot和一个Memory Slot
- 在Task执行较多的任务时，可以申请更多的CPU Slot
- 在Task需要更多的内存时，可以申请更多的额Memory Slot
- 在Task磁盘IO较多时，可以申请Disk Slot

#### JStorm架构

　　从设计层面来说，JStorm是一个典型的调度系统。在这个系统中，有以下内容：

| 角色       | 作用                                           |
| ---------- | ---------------------------------------------- |
| Nimbus     | 调度器                                         |
| Supervisor | Worker的代理角色，负责Kill掉Worker和运行Worker |
| Worker     | Task的容器                                     |
| Task       | 任务的执行者                                   |
| ZooKeeper  | 系统的协调者                                   |

## 使用：

#### 1.自定义Spout必须实现IRichSpout或者继承其他两个类。

Spout里面主要的方法是nextTuple，它里面可以发射新的tuple到拓扑，或者当没有消息的时候就return，需要注意，这个方法里面不能阻塞，因为storm调用spout方法是单线程的，其他的主要方法是ack和fail，如果使用了可靠的spout，可以使用ack和fail来确定消息发送状态 。(即BaseRichSpout，可以无需我们进行ack确认)

IRichSpout：spout类必须实现的接口 
BaseRichSpout ：可靠的spout有ack确保 
BaseBasicSpout ：不可靠的spout 

ISpout：

```
/**
 * 向后端发射tuple数据流
 * @author soul
 *
 */
public class SentenceSpout extends BaseRichSpout {

    //BaseRichSpout是ISpout接口和IComponent接口的简单实现，接口对用不到的方法提供了默认的实现

		//对象提供发射tuple的方法。
    private SpoutOutputCollector collector;
    
    private String[] sentences = {
            "my name is soul",
            "im a boy",
            "i have a dog",
            "my dog has fleas",
            "my girl friend is beautiful"
    };

    private int index=0;

    /**
     * open()方法中是ISpout接口中定义，在Spout组件初始化时被调用。
     * open()接受三个参数:一个包含Storm配置的Map,一个TopologyContext对象，提供了topology中组件的信息,SpoutOutputCollector对象提供发射tuple的方法。
     * 在这个例子中,我们不需要执行初始化,只是简单的存储在一个SpoutOutputCollector实例变量。
     */
    public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
        // TODO Auto-generated method stub
        this.collector = collector;
    }

    /**
     * nextTuple()方法是任何Spout实现的核心。
     * Storm调用这个方法，向输出的collector发出tuple。
     * 在这里,我们只是发出当前索引的句子，并增加该索引准备发射下一个句子。
     */
    public void nextTuple() {
        //collector.emit(new Values("hello world this is a test"));

        // TODO Auto-generated method stub
        this.collector.emit(new Values(sentences[index]));
        index++;
        if (index>=sentences.length) {
            index=0;
        }
        Utils.sleep(1);
    }

    /**
     * declareOutputFields是在IComponent接口中定义的，所有Storm的组件（spout和bolt）都必须实现这个接口
     * 用于告诉Storm流组件将会发出那些数据流，每个流的tuple将包含的字段
     */
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        // TODO Auto-generated method stub

        declarer.declare(new Fields("sentence"));//告诉组件发出数据流包含sentence字段

    }

}
```

#### 2.自定义逻辑处理Bolt

IRichBolt：bolts的通用接口 
IBasicBolt：扩展的bolt接口，可以自动处理ack 
OutputCollector：bolt发射tuple到下游bolt里面 

所有的拓扑处理都会在bolt中进行，bolt里面可以做任何etl，比如过滤，函数，聚合，连接，写入数据库系统或缓存等，一个bolt可以做简单的事件流转换，如果是复杂的流转化，往往需要多个bolt参与，这就是流计算，每个bolt都进行一个业务逻辑处理，bolt也可以emit多个流到下游，通过declareStream方法声明输出的schema(类似于标记名词，下个Bolt通过这个名称来拿数据)。 

Bolt里面主要的方法是execute方法，每次处理一个输入的tuple，bolt里面也可以发射新的tuple使用OutputCollector类，bolt里面每处理一个tuple必须调用ack方法以便于storm知道某个tuple何时处理完成。Strom里面的IBasicBolt接口可以自动 调用ack。 

```
public class WordCountSplitBolt extends BaseRichBolt{
 
    //bolt组件的收集器 用于将数据发送给下一个bolt
    private OutputCollector collector;
 
 
    //初始化
     /**
     * prepare()方法类似于ISpout 的open()方法。
     * 这个方法在blot初始化时调用，可以用来准备bolt用到的资源,比如数据库连接。
     * 本例子和SentenceSpout类一样,SplitSentenceBolt类不需要太多额外的初始化,
     * 所以prepare()方法只保存OutputCollector对象的引用。
     */
    @Override
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector collector) {
        this.collector = collector;
    }
    
 		/**
     * SplitSentenceBolt核心功能是在类IBolt定义execute()方法，这个方法是IBolt接口中定义。
     * 每次Bolt从流接收一个订阅的tuple，都会调用这个方法。
     * 本例中,收到的元组中查找“sentence”的值,
     * 并将该值拆分成单个的词,然后按单词发出新的tuple。
     */
    @Override
    public void execute(Tuple tuple) {
        //处理上一级发来的数据
        String value = tuple.getStringByField("sentence");
        String[] data= value.split(" ");
        //输出
        for (String word : data){
            collector.emit(new Values(word,1));
        }
    }
 		/**
 		* 发送schema结构，下个bolt也需要这样接受。
 		*/
    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        //申明发送给下一个组件的tuple schema结构
        declarer.declare(new Fields("word","count"));
    }
}
```

#### 3.**自定义Topology**

就是个普通的类，用于指定Spout，bolt。封装完成后开始创建任务执行。并配置运行模式：本地/集群。

一般来说与spring整合，使用@componet注解就可以了，可以直接定义一个main方法，获取applicationcontext对象获取这个类，然后进行执行。

```
@Component
public class WordCountTopology {
    public static void main(String[] args) {
        TopologyBuilder builder = new TopologyBuilder();
 
        //1 指定任务的spout组件
        builder.setSpout("1",new WordCountSpout());
 
        //2 指定任务的第一个bolt组件
        builder.setBolt("2",new WordCountSplitBolt()).shuffleGrouping("1");
 
        //3 指定任务的第二个bolt组件
        builder.setBolt("3",new WordCountTotalBolt()).fieldsGrouping("2",new Fields("word"));
 
        //创建任务
        StormTopology job = builder.createTopology();
 
        Config config = new Config();
 
        //运行任务有两种模式
        //1 本地模式   2 集群模式
 
        //1、本地模式
        LocalCluster localCluster = new LocalCluster();
        localCluster.submitTopology("MyWordCount",config,job);
 
        //2、集群模式：用于打包jar，并放到storm运行
//        StormSubmitter.submitTopology(args[0], conf, job);
    }
}
```



#### 过程：自定义Spout，先执行open方法设置初始配置，循环执行nextTuple从数据源获取数据，然后发射到collector到下个新的tuple(拓扑)，并调用declareOutputFields定义好schema。后面的Bolt先执行prepare方法进行初始化配置，后进行处理调用execute方法，如果还需要下一个Bolt处理。则继续用collector发射到新的tuple。collector.emit(new Values(record.value()));发射的类是一个Values。否则就无需发射，并且不用定义schmea。之后通过自定义Topology完成spout和bolt以及运行模式的配置，提交任务。

## 使用：直接拿到自定义Topology类调用方法。并根据标记判断是本地模式还是分布式模式。

```
public void startIndicatorTopology(boolean isRemote) {
    try {
        TopologyBuilder topologyBuilder = buildTopology();
        if (isRemote) {
            StormSubmitter.submitTopology(TOPOLOGY_NAME, getWorkerConfig(isRemote), topologyBuilder.createTopology());
        } else {
            LocalCluster cluster = new LocalCluster();
            cluster.submitTopology(TOPOLOGY_NAME, getWorkerConfig(isRemote), topologyBuilder.createTopology());
            //保证任务执行完毕
            Utils.sleep(10000000);
            cluster.shutdown();
        }
    } catch (Exception e) {
        log.error("启动IndicatorTopology失败:[{}]", e);
    }
}
```



 作 为storm的使用者，有两件事情要做以更好的利用storm的可靠性特征。 首先，在你生成一个新的tuple的时候要通知storm; 其次，完成处理一个tuple之后要通知storm。 这样storm就可以检测整个tuple树有没有完成处理，并且通知源spout处理结果。storm提供了一些简洁的api来做这些事情。

由一个tuple产生一个新的tuple称为： anchoring。你发射一个新tuple的同时也就完成了一次anchoring。

我 们通过anchoring来构造这个tuple树，最后一件要做的事情是在你处理完当个tuple的时候告诉storm, 通过OutputCollector类的ack和fail方法来做，如果你回过头来看看SplitSentence的例子， 你可以看到“句子tuple”在所有“单词tuple”被发出之后调用了ack。

每个你处理的tuple， 必须被ack或者fail。因为storm追踪每个tuple要占用内存。所以如果你不ack/fail每一个tuple， 那么最终你会看到OutOfMemory错误。

(发送ack/fail的过程，由于实现的接口默认帮忙实现了，锁不需要手动写)



所谓的grouping策略就是在Spout与Bolt、Bolt与Bolt之间传递Tuple的方式。总共有七种方式：

   1）shuffleGrouping（随机分组）

   2）fieldsGrouping（按照字段分组，在这里即是同一个单词只能发送给一个Bolt）

   3）allGrouping（广播发送，即每一个Tuple，每一个Bolt都会收到）

   4）globalGrouping（全局分组，将Tuple分配到task id值最低的task里面）

   5）noneGrouping（随机分派）

   6）directGrouping（直接分组，指定Tuple与Bolt的对应发送关系）

   7）Local or shuffle Grouping

   8）customGrouping （自定义的Grouping）

1. shuffleGrouping

   将流分组定义为混排。这种混排分组意味着来自Spout的输入将混排，或随机分发给此Bolt中的任务。shuffle grouping对各个task的tuple分配的比较均匀。

2. fieldsGrouping

   这种grouping机制保证相同field值的tuple会去同一个task，这对于WordCount来说非常关键，如果同一个单词不去同一个task，那么统计出来的单词次数就不对了。

3. All grouping

   广播发送， 对于每一个tuple将会复制到每一个bolt中处理。

4. Global grouping

   Stream中的所有的tuple都会发送给同一个bolt任务处理，所有的tuple将会发送给拥有最小task_id的bolt任务处理。

5. None grouping

   不关注并行处理负载均衡策略时使用该方式，目前等同于shuffle grouping,另外storm将会把bolt任务和他的上游提供数据的任务安排在同一个线程下。

6. Direct grouping

   由tuple的发射单元直接决定tuple将发射给那个bolt，一般情况下是由接收tuple的bolt决定接收哪个bolt发射的Tuple。这是一种比较特别的分组方法，用这种分组意味着消息的发送者指定由消息接收者的哪个task处理这个消息。 只有被声明为Direct Stream的消息流可以声明这种分组方法。而且这种消息tuple必须使用emitDirect方法来发射。消息处理者可以通过TopologyContext来获取处理它的消息的taskid (OutputCollector.emit方法也会返回taskid)

![image-20210409101336054](../../../Desktop/TyporaBlogMAC/图/image-20210409101336054.png)



