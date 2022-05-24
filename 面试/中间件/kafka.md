## Kafka

最初由Linkedin公司开发，是一个分布式、支持分区的（partition）、多副本的（replica）、多订阅者，基于zookeeper协调的分布式日志系统(也可以 当做MQ系统)，常见可以用于web/nginx日志、访问日志，消息服务等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源 项目。

Kafka是一个分布式消息队列。

- `高吞吐、低延迟`：kakfa 最大的特点就是收发消息非常快，kafka 每秒可以处理几十万条消息，它的最低延迟只有几毫秒。
- `高伸缩性`：每个主题(topic) 包含多个分区(partition)，主题中的分区可以分布在不同的主机(broker)中。
- `持久性、可靠性`：Kafka 能够允许数据的持久化存储，消息被持久化到磁盘，并支持数据备份防止数据丢失，Kafka 底层的数据存储是基于 Zookeeper 存储的，Zookeeper 我们知道它的数据能够持久存储。
- `容错性`：允许集群中的节点失败，某个节点宕机，Kafka 集群能够正常工作
- `高并发`：支持数千个客户端同时读写

★Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，

此外kafka集群有多个kafka实例组成，每个实例(server)称为broker。

无论是kafka集群，还是consumer都依赖于zookeeper集群保存一些meta信息，来保证系统可用性。

### 作用：

1.解耦2.异步3.削峰



##### Producer使用push(推)模式将消息发布到broker，Consumer使用pull(拉)模式从broker订阅并消费消息

1）Producer ：消息生产者，就是向kafka broker发消息的客户端；

2）Consumer ：消息消费者，向kafka broker取消息的客户端；

3）Topic ：同一个Topic的消息可以分布在一个或多个Broker上,一个Topic包含一个或者多个Partiion。但是不需要指定topic下的哪个partition(当然也可以固定)，因为kafka会把收到的message进行load balance，均匀的分布在这个topic下的不同的partition上（ hash(message) % [broker数量]  ）。这样就实现了水平扩展。**一个topic可以认为就是一种消息，partition就相当于把topic中所有的消息分成不同的接受点，类似于水平分表。(partition和broker有关)**

此时你可能会有疑惑，为什么 3个分区会随机分配到3台服务器，此时会涉及到多个分区在集群中的分配策略。那么多个分区如何在集群中做到合理的分配？

   答：（1）将所有 N 个Broker 和 i 个 Partition 排序(本例中 N = 3，i = 3)

（2）将第 i 个 Partition 分配到 （ i % n）个 Broker 上。（这样 test-1 就分配到第一台了，以此类推）

5）Broker ：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。broker 接收来自生产者的消息，为消息设置偏移量offset，并提交消息到磁盘保存。

6）Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partiton相当于是一个子queue，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序；同一个topic下有多个不同的partition，每个partiton为一个目录，partition的名称规则为：topic名称+有序序号，第一个序号从0开始计，最大的序号为partition数量减1，partition是实际物理上的概念，而topic是逻辑上的概念。

#### 怎么知道哪个消息被分到哪个partition中？

这里的 key 有什么用呢？当我们在发送一条消息时，我们可以指定这个 key ，那么 producer 则会根据 key 和 partition 机制，来判断当前这条消息应该发送并存储到哪个 partition 中。（此时问题1便得到了解决）

       如果 Kafka 中的 key 不为 null ？默认情况下，Kafka 采用的是 hash 取模的分区算法。
       如果 key 为 null 的话，则会随机的分配一个分区。这个随机是在这个参数 "metadata.max.age.ms"的时间范围内随机选择一个。对于这个时间段内，如果 key 为 null，则只会发送到唯一的分区，这个值默认情况下是 10 分钟更新一次。
       (当然可以自定义分区策略)


- **Segment**：partition物理上由多个segment组成，每个Segment存着message信息
- 如果就以partition为最小存储单位，我们可以想象当Kafka producer不断发送消息，必然会引起partition文件的无限扩张，这样对于消息文件的维护以及已经被消费的消息的清理带来严重的影响，所以这里以segment为单位又将partition细分。每个partition(目录)相当于一个巨型文件被平均分配到多个大小相等的segment(段)数据文件中(每个segment 文件中消息数量不一定相等)这种特性也方便old segment的删除，即方便已被消费的消息的清理，提高磁盘的利用率。每个partition只需要支持顺序读写就行

7）Offset：kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka。

4） Consumer Group （CG）：各个consumer（consumer 线程）可以组成一个组（Consumer group ），partition中的每个message只能被组（Consumer group ） 中的一个consumer（consumer 线程 ）消费，如果一个message可以被多个consumer（consumer 线程 ） 消费的话，那么这些consumer必须在不同的组。Kafka不支持一个partition中的message由两个或两个以上的consumer thread来处理，即便是来自不同的consumer group的也不行。它不能像AMQ那样可以多个BET作为consumer去处理message，这是因为多个BET去消费一个Queue中的数据的时候，由于要保证不能多个线程拿同一条message，所以就需要行级别悲观所（for update）,这就导致了consume的性能下降，吞吐量不够。而kafka为了保证吞吐量，只允许一个consumer线程去访问一个partition。如果觉得效率不高的时候，可以加partition的数量来横向扩展，那么再加新的consumer thread去消费。这样没有锁竞争，充分发挥了横向的扩展性，吞吐量极高。这也就形成了分布式消费的概念。

#### Consumer Group：消费组

消费数据的时候，都必须指定一个group id，指定一个组的id
假定程序A和程序B指定的group id号一样，那么两个程序就属于同一个消费组。

(就是通过groupId来确定消费者组)

副本：Kafka 中消息的备份又叫做 `副本`（Replica），副本的数量是可以配置的，Kafka 定义了两类副本：领导者副本（Leader Replica） 和 追随者副本（Follower Replica），前者对外提供服务，后者只是被动跟随。

一个topic物理上分为多个partition，位于不同的broker上。如果没有 replica，一旦broker宕机，其上所有的patition将不可用。

每个partition可以有多个replica(对应server.properties/default.replication.factor)，分配到不同的broker上，其中有一个leader负责读写，处理来自producer和consumer的请求；其它作为follower从leader pull消息，保持与leader的同步。



#### 重平衡：Rebalance。

消费者组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程。Rebalance 是 Kafka 消费者端实现高可用的重要手段。(保证消费者组有机器获取消息)

### 总结：一个 Partition 物理上对应一个文件夹，一个 Segment 对应一个文件。记录只会被 append 到 Segmentt 中，不会被单独删除或者修改。(不可变,如果过期或磁盘不足才自动删除)

#### (Consumer Group这是 kafka 用来实现一个 Topic 消息的广播（发送给所有的 consumer）和单播（发给任意一个 consumer）的手段，通过partiton只发送一个comsumerGroup中的一个consumer的特性，来实现多comsumerGroup。)

### 消息有序性：

##### 有序的消息，表示某个区间的消息需要按顺序处理，但是这种情况也不是特别多。比如：将所有违规的id发送到kafka，但是其实我们根本不关心他的顺序，只要将所有id存入案件中心。



kafka本身就是个分布式消息队列，无法保证不同机器上parition写入顺序，更无法保证消费者从多个parition读取数据，只能保证同一个partiton的读取顺序。本身应用的场景就不应该保证是全局消息的顺序性，适用于弱关注消息顺序性。

##### 实现：保证写入数据时，指定同一个key，来确定同一个partition。这样一来，partition内部数据就是相对有序。这样的话，如果consumer是单线程读数据，完全没问题，是有序的。但是kafka的特性就是并发取数据啊，这样多线程的情况下取数据，最后执行时还是不能保证有序。解决方法，消费者写N个queue，将具有相同key的数据都存储在同一个queue，然后对于N个线程，每个线程分别消费一个queue即可。(其实这样就相当于多个线程取数据根据key放入同一个queue中，然后消费相应的queue)



Kafka 中发送1条消息的时候，可以指定(topic, partition, key) 3个参数。partiton 和 key 是可选的。如果一个有效的partition属性数值被指定，那么在发送记录时partition属性数值就会被应用。如果没有partition属性数值被指定，而一个key属性被声明的话，一个partition会通过key的hash而被选中。如果既没有key也没有partition属性数值被声明，那么一个partition将会被分配以轮询的方式。

#### (其实key和partiton，或者不填，都是指定某个partiton的一种方式)



### 发送消息过程：

发送到leader，通过ack机制确保发送成功。followers只是保持同步数据。

![image-20200928152350247](../../../Desktop/TyporaBlogMAC/图/image-20200928152350247.png)

### 发送消息不丢失：

保证消息不丢失是一个消息队列中间件的基本保证，那producer在向kafka写入消息的时候，怎么保证消息不丢失呢？其实上面的写入流程图中有描述出来，那就是通过ACK应答机制！在生产者向队列写入数据的时候可以设置参数来确定是否确认kafka接收到数据，这个参数可设置的值为**0、1、all**。

- 0代表producer往集群发送数据不需要等到集群的返回，不确保消息发送成功。安全性最低但是效率最高。
- 1代表producer往集群发送数据只要leader应答就可以发送下一条，只确保leader发送成功。
- -1代表producer往集群发送数据需要所有的follower都完成从leader的同步才会发送下一条，确保leader发送成功和所有的副本都完成备份。安全性最高，但是效率最低。(leader和follower全部应答)

### 保存消息：

多个分区顺序写磁盘的总效率要比随机写内存还要高。

Producer将数据写入kafka后，集群就需要对数据进行保存了！kafka将数据保存在磁盘，可能在我们的一般的认知里，写入磁盘是比较耗时的操作，不适合这种高并发的组件。Kafka初始会单独开辟一块磁盘空间，顺序写入数据（效率比随机写入高）。

每组segment文件又包含.index文件、.log文件、.timeindex文件（早期版本中没有）三个文件，

log文件就实际是存储message的地方，而index和timeindex文件为索引文件，用于检索消息。

如下：0.log存了368795个数据，offset为0-368795。

数据查找高效的原因：二分查找到指定的segment，其中稀疏索引的方式存储着相对offset及对应message物理偏移量的关系。先找到offset，在找到物理偏移量。(二分查找+索引)

**index采用稀疏存储的方式**，它不会为每一条message都建立索引，而是每隔一定的字节数建立一条索引，避免索引文件占用过多的空间。缺点是没有建立索引的offset不能一次定位到message的位置，需要做一次顺序扫描，但是扫描的范围很小。

**索引包含两个部分**（均为4个字节的数字），分别为相对offset和position。相对offset表示segment文件中的offset，position表示message在数据文件中的位置。

#### 总结：Kafka的Message存储采用了分区(partition)，磁盘顺序读写，分段(LogSegment)和稀疏索引这几个手段来达到高效性

<img src="../../../Desktop/TyporaBlogMAC/图/image-20200928153102147.png" alt="image-20200928153102147" style="zoom:67%;" />

##### Message结构

上面说到log文件就实际是存储message的地方，我们在producer往kafka写入的也是一条一条的message，那存储在log中的message是什么样子的呢？消息主要包含消息体、消息大小、offset、压缩类型……等等！我们重点需要知道的是下面三个：

- **offset**：offset是一个占8byte的有序id号，它可以唯一确定每条消息在parition内的位置！

- **消息大小**：消息大小占用4byte，用于描述消息的大小。

- **消息体**：消息体存放的是实际的消息数据（被压缩过），占用的空间根据具体的消息而不一样。

  在早期的版本中，消费者将消费到的offset维护zookeeper中，consumer每间隔一段时间上报一次，这里容易导致重复消费，且性能不好！在新的版本中消费者消费到的offset已经直接维护在kafk集群的__consumer_offsets这个topic中！

  

##### 存储策略(时间复杂度为1，所以删除数据不会提高性能)

无论消息是否被消费，kafka都会保存所有的消息。那对于旧数据有什么删除策略呢？

- 基于时间，默认配置是168小时（7天）。

- 基于大小，默认配置是1073741824。

  需要注意的是，kafka读取特定消息的时间复杂度是O(1)，所以这里删除过期的文件并不会提高kafka的性能！



### 消费数据：

消费者也是从leader中拉取数据。每个消费者组有个id，同一个消费组者的消费者可以消费同一topic下不同分区的数据，但是不会组内多个消费者消费同一分区的数据！！

所以在实际的应用中，**建议消费者组的consumer的数量与partition的数量一致**！



Kafka 通过 offset 可以保证消息在分区中的顺序性，但是跨分区是无序的，即 **Kafka 只保证在同一个分区内的消息是有序的。**



# Kafka高效原因



### Kafka 磁盘顺序写保证写数据性能

kafka 写数据：顺序写，往磁盘上写数据时，就是追加数据，没有随机写的操作。经验：如果一个服务器磁盘达到一定的个数，磁盘也达到一定转数，往磁盘里面顺序写（追加写）数据的速度和写内存的速度差不多。

生产者生产消息，经过 kafka 服务先写到 os cache 内存中，然后经过 sync 顺序写到磁盘上

(先写到内存缓存，在顺序写磁盘)

### 2.Kafka 零拷贝机制保证读数据高性能

消费者读取数据流程：

1. 消费者发送请求给 kafka 服务 
2. 2.kafka 服务去 os cache 缓存读取数据（缓存没有就去磁盘读取数据） 
3. 3.从磁盘读取了数据到 os cache 缓存中
4.  4.os cache 复制数据到 kafka 应用程序中 
5. 5.kafka 将数据（复制）发送到 socket cache 中
6.  6.socket cache 通过网卡传输给消费者

（这个过程发生了两次copy，一次从缓存copy到kafka应用程序，一次从应用程序copy网卡缓存）

<img src="../../../Library/Application Support/typora-user-images/image-20220329182115890.png" alt="image-20220329182115890" style="zoom: 33%;" />

kafka linux sendfile 技术 — 零拷贝 

1. 消费者发送请求给 kafka 服务 
2. 2.kafka 服务去 os cache 缓存读取数据（缓存没有就去磁盘读取数据） 
3. 3.从磁盘读取了数据到 os cache 缓存中 
4. 4.os cache 直接将数据发送给网卡
5.  5.通过网卡将数据传输给消费者

<img src="../../../Library/Application Support/typora-user-images/image-20220329182304442.png" alt="image-20220329182304442" style="zoom:33%;" />

# kafka的时间戳

**Kafka消息的时间戳**

在消息中增加了一个时间戳字段和时间戳类型。目前支持的时间戳类型有两种： CreateTime 和 LogAppendTime 前者表示producer创建这条消息的时间；后者表示broker接收到这条消息的时间(严格来说，是leader broker将这条消息写入到log的时间)

**为什么要加入时间戳？**

引入时间戳主要解决3个问题：

- 日志保存(log retention)策略：Kafka目前会定期删除过期日志(log.retention.hours，默认是7天)。判断的依据就是比较日志段文件(log segment file)的最新修改时间(last modification time)。倘若最近一次修改发生于7天前，那么就会视该日志段文件为过期日志，执行清除操作。但如果topic的某个分区曾经发生过分区副本的重分配(replica reassigment)，那么就有可能会在一个新的broker上创建日志段文件，并把该文件的最新修改时间设置为最新时间，这样设定的清除策略就无法执行了，尽管该日志段中的数据其实已经满足可以被清除的条件了。
- 日志切分(log rolling)策略：与日志保存是一样的道理。当前日志段文件会根据规则对当前日志进行切分——即，创建一个新的日志段文件，并设置其为当前激活(active)日志段。其中有一条规则就是基于时间的(log.roll.hours，默认是7天)，即当前日志段文件的最新一次修改发生于7天前的话，就创建一个新的日志段文件，并设置为active日志段。所以，它也有同样的问题，即最近修改时间不是固定的，一旦发生分区副本重分配，该值就会发生变更，导致日志无法执行切分。（注意：log.retention.hours及其家族与log.rolling.hours及其家族不会冲突的，因为Kafka不会清除当前激活日志段文件）
- 流式处理(Kafka streaming)：流式处理中需要用到消息的时间戳

***\*消息格式的变化\****

1 增加了timestamp字段，表示时间戳

2 增加了timestamp类型字段，保存在attribute属性低位的第四个比特上，0表示CreateTime；1表示LogAppendTime(低位前三个比特保存消息压缩类型)

***\*客户端消息格式的变化\****

ProducerRecord：增加了timestamp字段，允许producer指定消息的时间戳，如果不指定的话使用producer客户端的当前时间

ConsumerRecord：增加了timestamp字段，允许消费消息时获取到消息的时间戳

ProducerResponse: 增加了timestamp字段，如果是CreateTime返回-1；如果是LogAppendTime，返回写入该条消息时broker的本地时间 

**如何使用时间戳？**

Kafka broker config提供了一个参数：log.message.timestamp.type来统一指定集群中的所有topic使用哪种时间戳类型。用户也可以为单个topic设置不同的时间戳类型，具体做法是创建topic时覆盖掉全局配置： 

```
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test --partitions 1 --replication-factor 1 --config message.timestamp.type=LogAppendTime
```

另外， producer在创建ProducerRecord时可以指定时间戳: 

```javascript
record = new ProducerRecord<String, String>("my-topic", null, System.currentTimeMillis(), "key", "value");
```

### 基于时间戳的功能

**1 根据时间戳来定位消息：之前的索引文件是根据offset信息的，从逻辑语义上并不方便使用，引入了时间戳之后，Kafka支持根据时间戳来查找定位消息**

**2 基于时间戳的日志切分策略**

**3 基于时间戳的日志清除策略**

个人认为，第2，3点其实是引入时间戳的初衷，而第1点可以看做是时间戳的另一个功能应用。













