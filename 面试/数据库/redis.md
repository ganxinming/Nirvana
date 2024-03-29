# Redis是什么？

Redis 是 C 语⾔开发的⼀个开源的（遵从 BSD 协议）⾼性能键值对（key- 

value）的内存数据库，可以⽤作数据库、缓存、消息中间件等。 

(C语言开发的非关系型内存数据库，可做缓存，分布式锁，数据库)

特点：

采用单进程单线程模型，是线程安全的，Redis 使⽤多路 I/O 复⽤机制，为⾮阻塞 IO。Redis 采⽤的 I/O 多路复⽤函数：epoll/kqueue/evport/select。

Redis 完全基于内存，绝⼤部分请求是纯粹的内存操作，执⾏效率⾼，⽀持并发 10W QPS。

⽀持数据持久化，可以将内存中数据保存在磁盘中，重启时加载。

可以作为消息中间件使⽤，⽀持发布订阅。 

主从复制，哨兵，集群三种模式。

时间复杂度为 O(1)

# Redis执行流程

<img src="../../../Library/Application Support/typora-user-images/image-20220704105343413.png" alt="image-20220704105343413" style="zoom:33%;" />



当用户输入一条命令之后，==客户端会以 socket 的方式把数据转换成 Redis 协议==，并发送至服务器端，服务器端在接受到数据之后，会先将协议转换为真正的执行命令，在经过各种验证以保证命令能够正确并安全的执行，但验证处理完之后，会调用具体的方法执行此条命令，执行完成之后会进行相关的统计和记录，然后再把执行结果返回给客户端，整个执行流程

# Redis的数据结构 

Redis⽀持多种不同的数据结构，包括5种基础数据结构和⼏种⽐较复杂的数据，这些数据结构可以满⾜不同的应⽤场景。 

## 基础数据结构

String：字符串，是构建其他数据结构的基础 （SDS：记录已使用和未使用长度和byte[]，最大其值最⼤可存储 512M）

Hash：哈希列表 

List：列表 

Set：集合，在哈希列表的基础上实现 

Sort Set：有序集合 

## 复杂数据结构（底层都是基于基础数据结构）

Bitmaps:位图，在string的基础上进⾏位操作，可以实现节省空间的数据结构。 

Hyperlog：⽤于估计⼀个 set 中元素数量的概率性的数据结构。 

Geo：geospatial,地理空间索引半径查询。 (输入经纬度，返回经纬度附近的值)

## Redis快的原因

1.单进程，单线程

2.IO多路复用

3.基于内存

#### 采用单线程，为什么不采用多线程？

1. Redis操作基于内存，绝大多数操作的性能瓶颈不在CPU(单线程足以应付)
2. 避免多线程上下文切换，*产生不必要的开销*(vmstat查看cs上下文切换)
3. IO多路复用，也能并发的处理客户端的请求连接(但其实只是并发处理请求的连接，执行命令还是单线程)。

(综上，决定redis性能的根本原因是内存和网络IO)

#### Redis6.0为何引入多线程？

Redis6.0引入的多线程部分，实际上只是用来处理网络数据的读写和协议解析，执行命令仍然是单一工作线程。



## IO多路复用

==“多路”指的是多个网络连接，“复用”指的是复用同一个线程==

1.网络交互，其实就是两个socket进行交互。通信时，一个应用程序将数据写入Socket，然后通过网卡把数据发送到另外一个应用程序的Socket中。

2.一般网络通信，涉及数据读取操作，都会有内核和用户空间复制的情况。Linux区分了内核空间和用户空间。可以这样理解，内核空间运行操作系统程序和驱动程序，用户空间运行应用程序。Linux以这种方式隔离了操作系统程序和应用程序，避免了应用程序影响到操作系统自身的稳定性。(1.不会因为应用程序的问题而影响系统2.控制权限，不能随便篡改数据)

3.**I/O多路复用，I/O就是指的我们网络I/O,多路指多个TCP连接(或多个Channel)，复用指复用一个或少量线程。串起来理解就是很多个网络I/O复用一个或少量的线程来处理这些连接。然后维持这些多个channel的列表，去监视他们是否有读写操作。**





<img src="../../../Library/Application Support/typora-user-images/image-20220704105236061.png" alt="image-20220704105236061" style="zoom: 33%;" />



## 常用命令

##### 1.keys命令(返回指定key前缀，等待时间长，生产不建议用，建议使用scan)

作⽤是列出Redis所有的key,该命令的时间复杂度为O(N)，N随着Redis中key的数量增加⽽增加，因此Redis 

有⼤量的key，keys命令会执⾏很⻓时间，⽽由于Redis是单线程，某个命令耗费过⻓时间，则会导致后⾯的的所有请求⽆ 

法得到响应，因此，千万不要在⽣产服务器上使⽤keys命令。 

keys hello*  keys heelo?   keys hello[a-z] 

##### SCAN cursor [MATCH pattern] [COUNT count] (假设 Redis 中有⼗亿条 Key，如何从这么多 Key 中找到固定前缀的 Key？ )

cursor：游标 

MATCH pattern：查询 Key 的条件 

Count：返回的条数

scan命令的游标从0开始，也从0结束，每次返回的数据，都会返回下一次游标应该传的值，我们根据这个值，再去进行下一次的访问，如果返回的数据为空，并不代表没有数据了，只有游标返回的值是0的情况下代表结束。



##### 2.exists命令(判断key是否存在)

exists命令⽤于判断⼀个或多个key是否存在，判断多个key时，key之间⽤空格分隔,exists的返回值为整数，表⽰当前判 

断有多少个key是存在的。 

exists key [key ...] 

##### 3.del命令(删除)

del命令⽤于删除⼀个或多个key，多个key之间⽤空格分隔，其返回值为整数，表⽰成功删除了多少个存在的key，因此，如 

果只删除⼀个key，则可以从返回值中判断是否成功，如果删除多个key，则只能得到删除成功的数

del key [key ...] 

##### 4.expire，pexpire(设置过期时间，毫秒)

expire设置key在多少秒之后过期，pexpire设置key在多少毫秒之后过期,成功返回1，失败返回0。

\# expire命令，时间复杂度为O(1) 

expire key seconds 

\# pexpire命令，时间复杂度为O(1) 

pexpire key milliseconds

##### 5.ttl和pttl命令(返回剩下过期秒数，毫秒)

ttl和pttl命令⽤于获取key的过期时间，其返回值为整型，代表的意义分为⼏种情况： 

当key不存在或过期时间，返回-2。 

当key存在且永久有效时，返回-1。 

当key有设置过期时间时，返回为剩下的秒数(pttl为毫秒数) 

⽰例(ttl的演⽰，pttl类似) 

设置key在某个时间戳过期,expreat参数时间戳⽤秒表⽰，⽽pexpireat则⽤毫秒表⽰

\# ttl命令，时间复杂度O(1) 

ttl key 

##### 6.expreat和pexpireat(指定时间过期类似expire)

设置key在某个时间戳过期,expreat参数时间戳⽤秒表⽰，⽽pexpireat则⽤毫秒表⽰，与expire和pexpire功能类似，返 

回1表⽰成功，0表⽰失败。 

##### 7.persist(将过期key设置永久)

移除key的过期时间，将key设置为永久有效，当key设置了过期时间，使⽤persist命令移除后返回1，如果key不存在或本 

⾝就是永久有效的，则返回0。 

persist key

#### 8.type

判断key是什么类型的数据结构,返回值为string,list,set,hash,zset,分别表⽰我们前⾯介绍的Redis的5种基础数据结 

构。

geo,hyperloglog,bitmaps等复杂的数据结构，都是在这五种基础数据结构上实现，⽐如geo是zset类型，hyperloglog 

和bitmaps都为string。 

type test 



# 分布式锁

##### 使用SETNX命令，如果key不存在则赋值，如果设置成功，则返回 1，否则返回 0。 

##### 由于 SETNX 并不⽀持传⼊ EXPIRE 参数，所以我们可以直接使⽤ EXPIRE 指令来对特定的 Key 来设置过期时间。

##### 从 Redis 2.6.12 版本开始，我们就可以使⽤ Set 操作，将 SETNX 和 EXPIRE 融合在⼀起执⾏，具体做法如下： 

EX ：：设置键的过期时间为 Second 秒。 

PX ：设置键的过期时间为 MilliSecond 毫秒。 

NX：：只在键不存在时，才对键进⾏设置操作。 

XX：只在键已经存在时，才对键进⾏设置操作。 

SET KEY value [EX seconds] [PX milliseconds] [NX|XX] 

(所以新版本只需要SET就行，但是需要指定NX,EX)



# Redis分布式锁

### redis命令： setnx

如果为空就set值，并返回1

如果存在(不为空)不进行操作，并返回0

很明显，比get和set要好。因为先判断get，再set的用法，有可能会重复set值。



```
redis> SETNX mykey "Hello"
(integer) 1
redis> SETNX mykey "World"
(integer) 0
redis> GET mykey
"Hello"
```



### javaAPI：**setIfAbsent()**

```
						//不存在，就加锁，返回一个Boolean，true 加锁成功,记得设置超时时间，防止别的线程挂了，锁释放不了
            Boolean flag = redisTemplate.opsForValue().setIfAbsent("Mylock", value,10L, TimeUnit.SECONDS);
            if (!flag) {
                return "抢锁失败，请再次重试！！";
            }	
            String result = redisTemplate.opsForValue().get("goods:001");//get key==看看库存的数量够不够
						//建议在finally中写
						finally {
						redisTemplate.delete("Mylock");//解锁
						}
```

问题1：A线程执行加锁，执行时间超过了锁的有效期，此时B加锁，而A又结束，把B的锁释放了。释放前进行判断

```
finally {
            if (redisTemplate.opsForValue().get(REDIS_LOCK).equalsIgnoreCase(value)) {
                redisTemplate.delete("Mylock");//解锁
            }
```

问题2：但是这种解锁也是有问题，不是一个原子操作。

==Lua脚本进行解决删锁问题（redis官网推荐）==

```
finally {
            Jedis jedis = RedisUtils.getJedis();
            String script = "if redis.call('get', KEYS[1]) == ARGV[1]" + "then "
                    + "return redis.call('del', KEYS[1])" + "else " + "  return 0 " + "end";
            try {
                Object o = jedis.eval(script, Collections.singletonList(REDIS_LOCK), Collections.singletonList(value));//传入redis的key和value
                if ("1".equalsIgnoreCase(o.toString())) {//删除成功就是1
                    System.out.println("-----del redis lock ok");
                } else {
                    System.out.println("----del redis lock error");
                }
            } finally {
                if (null != jedis) {
                    jedis.close();//释放jedis
                }
```



# 实现队列

使⽤上⽂所说的 Redis 的数据结构中的 List 作为队列 Rpush ⽣产消息，LPOP 消费消息。

(Rpush右插入，LPOP左弹出 )

但是LPOP没有元素会直接返回，不会等待队列产生元素。

一般可以使用==BLPOP==：阻塞直到队列有消息或者超时。

# 实现消息队列

1.就是上面的BLPOP，需要消费者循环拉数据

2.发布者订阅者模式

发送者（Pub）发送消息，订阅者（Sub）接收消息。订阅者可以订阅任意数量的频道。 

可以使用publish和subscribe来实现，但是毕竟不是专业的，还是使用正常的消息队列吧。

# Redis持久化

持久化，即将数据持久存储，⽽不因断电或其他各种复杂外部环境影响数据的完整性。 

Redis 持久化拥有以下三种方式：

- **快照方式**（RDB, Redis DataBase）将某一个时刻的内存数据，以二进制的方式写入磁盘；
- **文件追加方式**（AOF, Append Only File），记录所有的操作命令，并以文本的形式追加到文件中；
- **混合持久化方式**，Redis 4.0 之后新增的方式，混合持久化是结合了 RDB 和 AOF 的优点，在写入的时候，先把当前的数据以 RDB 的形式写入文件的开头，再将后续的操作命令以 AOF 的格式存入文件，这样既能保证 Redis 重启时的速度，又能减低数据丢失的风险。

Redis ⽬前有两种持久化⽅式，即 RDB 和 AOF，RDB 是通过保存某个时间点的全量数据快照实现数据的持久化，当恢复数 

RDB持久化会在某个特定的间隔保存那个时间点的全量数据的快照。 

### ==RDB 配置⽂件，redis.conf：==

```
save 900 1 #在900s内如果有1条数据被写⼊，则产⽣⼀次快照。 

save 300 10 #在300s内如果有10条数据被写⼊，则产⽣⼀次快照 

save 60 10000 #在60s内如果有10000条数据被写⼊，则产⽣⼀次快照 

stop-writes-on-bgsave-error yes 

\#stop-writes-on-bgsave-error ： 

如果为yes则表⽰，当备份进程出错的时候， 

主进程就停⽌进⾏接受新的写⼊操作，这样是为了保护持久化的数据⼀致性的问题。 
```



## 手动触发

**==手动触发持久化的操作有两个： `save` 和 `bgsave` ，它们主要区别体现在：是否阻塞 Redis 主线程的执行。==**

SAVE：：阻塞 Redis 的服务器进程，直到 RDB ⽂件被创建完毕。SAVE 命令很少被使⽤，因为其会阻塞主线程来保证快照的 

写⼊，由于 Redis 是使⽤⼀个主线程来接收所有客⼾端请求，这样会阻塞所有客⼾端请求。 

==BGSAVE：==该指令会 Fork 出⼀个⼦进程来创建 RDB ⽂件，不阻塞服务器进程，⼦进程接收请求并创建 RDB 快照，⽗进程 

继续接收客⼾端的请求。 

==(但是实际上还是使用BGSAVE命令，去创建RDB文件，因为他是子进程去创建，不影响服务进程)==

<img src="../../../Library/Application Support/typora-user-images/image-20220704113048037.png" alt="image-20220704113048037" style="zoom: 50%;" />





### ⾃动化触发

### RDB持久化的⽅式如下(即触发RDB同步)： 

根据 redis.conf 配置⾥的 SAVE m n 定时触发（实际上使⽤的是 BGSAVE）。 (配置文件)

主从复制时，主节点⾃动触发。 (主从节点复制，如：从节点第一次启动)

执⾏ Debug Reload。 

执⾏ Shutdown 且没有开启 AOF 持久化(关闭时，而又没有AOF自同步)。 



### AOF

AOF 持久化是通过保存 Redis 的写状态来记录数据库的。 

相对 RDB 来说，RDB 持久化是通过备份数据库的状态来记录数据库，⽽ AOF 持久化是备份数据库接收到的指令： 

1.AOF 记录除了查询以外的所有变更数据库状态的指令。 

2.以增量的形式追加保存到 AOF ⽂件中。 

```
appendonly yes  //开启
#appendsync always always 表⽰总是即时将缓冲区内容写⼊ AOF ⽂件当中，everysec 表⽰每隔⼀秒将缓冲区内容写⼊ AOF ⽂件，no 表⽰ 将写⼊⽂件操作交由操作系统决定。
appendfsync everysec 
# appendfsync no
```

#### AOF 和 RDB 的优缺点如下： 

RRDDBB 优优点点：：全量数据快照，⽂件⼩，恢复快。 

RRDDBB 缺缺点点：：⽆法保存最近⼀次快照之后的数据。 

AAOOFF 优优点点：：可读性⾼，适合保存增量数据，数据不易丢失。 

AAOOFF 缺缺点点：：⽂件体积⼤，恢复时间⻓。 

### 线上使用RDB和AOF混合模式

RDB 作为全量备份，AOF 作为增量备份，并且将此种⽅式作为默认⽅式使⽤。

前半段是 RDB 格式的全量数据，后半段是 AOF 格式的增量数据。此种⽅式是⽬前较 为推荐的⼀种持久化⽅式。 



## Redis内存问题：

#### **1.Redis**的内存淘汰策略：

既然可以设置Redis最⼤占⽤内存⼤⼩，那么配置的内存就有⽤完的时候。那在内存⽤完的时候，还继续往Redis⾥⾯添加数据不 

就没内存可⽤了吗？ 

- noeviction: 不删除策略, 达到最大内存限制时, 如果需要更多内存, 直接返回错误信息。 大多数写命令都会导致占用更多的内存(有极少数会例外, 如 DEL )。
- allkeys-lru: 所有key通用; 优先删除最近最少使用(less recently used ,LRU) 的 key。
- volatile-lru: 只限于设置了 expire 的部分; 优先删除最近最少使用(less recently used ,LRU) 的 key。
- allkeys-random: 所有key通用; 随机删除一部分 key。
- volatile-random: 只限于设置了 expire 的部分; 随机删除一部分 key。
- volatile-ttl: 只限于设置了 expire 的部分; 优先删除剩余时间(time to live,TTL) 短的key。
-  volatile-lfu：从所有配置了过期时间的键中驱逐使用频率最少的键(区别于lru，他是使用频率最少，而不是最近最少)
- allkeys-lfu：从所有键中驱逐使用频率最少的键

通过配置⽂件设置淘汰策略（修改redis.conf⽂件）： 

```
maxmemory-policy allkeys-lru
```

LFU算法是Redis4.0⾥⾯新加的⼀种淘汰策略。它的全称是LLeeaasstt FFrreeqquueennttllyy UUsseedd，它的核⼼思想是根据key的最近被访问的频率 进⾏淘汰，很少被访问的优先被淘汰，被访问的多的则被留下来。 

LFU算法能更好的表⽰⼀个key被访问的热度。假如你使⽤的是LRU算法，⼀个key很久没有被访问到，只刚刚是偶尔被访问了⼀ 

次，那么它就被认为是热点数据，不会被淘汰，⽽有些key将来是很有可能被访问到的则被淘汰了。如果使⽤LFU算法则不会出现这 

种情况，因为使⽤⼀次并不会使⼀个key成为热点数据。 

LFU⼀共有两种策略： 

volatile-lfu：在设置了过期时间的key中使⽤LFU算法淘汰key 

allkeys-lfu：在所有的key中使⽤LFU算法淘汰数据 

#### 2.设置最大redis内存

```
maxmemory 100mb
```

==所有的配置都可以通过命令进行动态修改：==

```
> config get maxmemory-policy //获取
>config set maxmemory-policy allkeys-lru //设置
```

#### 3.过期策略

### **定期删除**

redis 会将每个设置了过期时间的 key 放入到一个独立的字典中，以后会定期遍历这个字典来删除到期的 key。

Redis 默认会每秒进行十次过期扫描（100ms一次），过期扫描不会遍历过期字典中所有的 key，而是采用了一种简单的贪心策略。

1.从过期字典中随机 20 个 key；

2.删除这 20 个 key 中已经过期的 key；

3.如果过期的 key 比率超过 1/4，那就重复步骤 1；

redis默认是每隔 100ms就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除。注意这里是随机抽取的。为什么要随机呢？你想一想假如 redis 存了几十万个 key ，每隔100ms就遍历所有的设置过期时间的 key 的话，就会给 CPU 带来很大的负载。

### **惰性删除**

所谓惰性策略就是在客户端访问这个key的时候，redis对key的过期时间进行检查，如果过期了就立即删除，不会给你返回任何东西。

定期删除可能会导致很多过期key到了时间并没有被删除掉。所以就有了惰性删除。假如你的过期 key，靠定期删除没有被删除掉，还停留在内存里，除非你的系统去查一下那个 key，才会被redis给删除掉。这就是所谓的惰性删除，即当你主动去查过期的key时,如果发现key过期了,就立即进行删除,不返回任何东西.

总结：定期删除是集中处理，惰性删除是零散处理。

## **为什么需要淘汰策略**

有了以上过期策略的说明后，就很容易理解为什么需要淘汰策略了，因为不管是定期采样删除还是惰性删除都不是一种完全精准的删除，就还是会存在key没有被删除掉的场景，所以就需要内存淘汰策略进行补充。

## 附近的人

⾃Redis 3.2开始，Redis基于geohash和有序集合提供了地理位置相关功能。Redis Geo模块包含了以下6个命令： 

GEOADD: 将给定的位置对象（纬度、经度、名字）添加到指定的key; 

GEOPOS: 从key⾥⾯返回所有给定位置对象的位置（经度和纬度）; 

GEODIST: 返回两个给定位置之间的距离; 

GEOHASH: 返回⼀个或多个位置对象的Geohash表⽰; 

GEORADIUS: 以给定的经纬度为中⼼，返回⽬标集合中与中⼼的距离不超过给定最⼤距离的所有位置对象; 

GEORADIUSBYMEMBER: 以给定的位置对象为中⼼，返回与其距离不超过给定最⼤距离的所有位置对象。 

其中，组合使⽤==GEOADD和GEORADIUS==可实现“附近的⼈”中“增”和“查”的基本功能。要实现微信中“附近的⼈”功能， 

可直接使⽤==GEORADIUSBYMEMBER==命令。

### 统计用户数量

1.hset，hlen 优点：简单。缺点：大数据量，性能也会下降

2.用setBit命令

3.使用HyperLogLog算法，他是⼀种基数评估算法。 这种算法的特征，⼀般都是数据不存具体的值，⽽是存⽤来计算概率的⼀些相关数据。 

当⽤⼾访问⽹站的时候，我们可以使⽤PFADD命令，设置对应的命令，最后我们只要通过PFCOUNT就能顺利计算出最终的结 

果，因为这个只是⼀个概率算法，所以可能存在0.81%的误差



# 集群

# redis集群

redis集群有三种模式，分别是主从同步/复制、哨兵模式、Cluster。

#### 主从：

<img src="../../../Library/Application Support/typora-user-images/image-20210913163947414.png" alt="image-20210913163947414" style="zoom:20%;" />

主节点：提供读写，写数据同步从节点。==复制（replication）功能==

从节点：提供读能力，接受主节点数据更新

一个主数据库可以拥有多个从数据库，而一个从数据库只能拥有一个主数据库

##### 主从复制流程：大体可以分为3个阶段：连接建立阶段（即准备阶段）、数据同步阶段、命令传播阶段。

1.从节点启动，则它会向Master机器发送一个“sync command”命令到主节点，请求同步连接。(通信建立连接)

2.主节点得知有从节点连接，启动一个后台进程保存当前数据的rdb文件，并记录下之后的修改操作AOF。(同步快照)

3.主节点将rbd文件发送从节点，待从节点加载完毕rdb文件，同步期间的写命令AFO(同步写缓存)

4.正常同步增量操作(同步增量)



<img src="../../../Library/Application Support/typora-user-images/image-20210913170448071.png" alt="image-20210913170448071" style="zoom:50%;" />

##### 作用：

读写分离，主从复制，负载均衡，高可用

##### 优点：

1.读写分离，分担master的读写压力

2.方便容灾恢复。

##### 缺点：

1.一旦主节点宕机，其它节点不会竞争称为主节点，此时，Redis将丧失写的能力。(不具备自动容错和恢复功能)

2.每台redis服务器存储的数据是相同的，浪费内存。

3.较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。(扩容复杂)

#### 哨兵：**Sentinel充当了Redis主从实例的守卫者，是构成Redis高可用的一个重要组成部分**

<img src="../../../Library/Application Support/typora-user-images/image-20210913164933098.png" alt="image-20210913164933098" style="zoom:50%;" />

功能：哨兵是Redis集群架构中非常重要的一个组件，哨兵的出现主要是解决了主从复制出现故障时需要人为干预的问题。

**① 监控**：负责监控Redis master和slave进程是否正常工作(监听主服务器和从服务器之间是否在正常工作)

**② 通知**：如果某个Redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员(它能够通过API告诉系统管理员或者程序，集群中某个实例出了问题)

**③ 故障转移**：如果master node挂掉了，会自动转移到slave node上（选举新节点）

**④ 配置中心**：如果故障转移发生了，它还能够向使用者提供当前主节点的地址。这在故障转移后，使用者不用做任何修改就可以知道当前主节点地址。(通过发布订阅模式通知其他从服务器，修改配置文件，让他们切换主机)

##### 哨兵高可用：由于哨兵本身也是可能失效的，所以使用哨兵集群(一样，选举新主节点)

<img src="../../../Library/Application Support/typora-user-images/image-20210913165508769.png" alt="image-20210913165508769" style="zoom:50%;" />

- 使用一个或者多个哨兵(Sentinel)实例组成的系统，对redis节点进行监控，在主节点出现故障的情况下，能将从节点中的一个升级为主节点，进行故障转义，保证系统的可用性。

##### 工作原理：

##### 主从节点信息获取：

(所有哨兵都会做同样的事，每个哨兵的信息都一致，它们之间建立了一个发布订阅。为了哨兵之间的信息长期对称它们之间也会互发 ping 命令。sentinel 会给主从的所有节点发送命令获取其状态，并且会把信息发布到哨兵的订阅里。 )

哨兵节点会和配置的主节点建立起两条连接**命令连接**和**订阅连接**

哨兵会通过命令连接每10s发送一次**INFO命令**，通过**INFO命令**，主节点会返回自己的**run_id**和自己的从节点信息。

哨兵会根据在主节点拿到的从节点信息，给对应的从节点也发送 info 指令。

哨兵会对这些从节点也建立两条连接命令连接和订阅连接。

##### 主客观下线：

哨兵(Sentinel)节点会每秒一次的频率向建立了命令连接的实例发送PING命令，如果在**down-after-milliseconds**毫秒内没有做出有效响应包括(PONG/LOADING/MASTERDOWN)以外的响应，哨兵就会将该实例在本结构体中的状态标记为SRI_S_DOWN主观下线。

当一个哨兵节点发现主节点处于主观下线状态是，会向其他的哨兵节点发出询问，该节点是不是已经主观下线了。如果超过配置参数`quorum`个节点认为是主观下线时，该哨兵节点就会将自己维护的结构体中该主节点标记为**SRI_O_DOWN**客观下线
询问命令**SENTINEL is-master-down-by-addr**

##### 选举哨兵master(总得选出个哨兵出来进行故障转移吧)：

假设说是 sentinel1 的票数满足总哨兵数量的一半之多后，sentinel1 就会当选

##### 进行故障转移：

1.在从节点中挑选出新的主节点，选择优先级`slave-priority`最大的从节点作为主节点

2.将其他的从节点，设置新的主节点

3.将旧的主节点设置为从节点

### 总结：当redis集群的主节点故障时，[Sentinel](https://so.csdn.net/so/search?q=Sentinel&spm=1001.2101.3001.7020)集群将从剩余的从节点中选举一个新的主节点，有以下步骤：

1. 故障节点主观下线
2. 故障节点客观下线
3. Sentinel集群选举Leader
4. Sentinel Leader决定新主节点

#### 为什么Sentinel集群至少3节点

一个Sentinel节选举成为Leader的最低票数为quorum和Sentinel节点数/2+1的最大值，如果Sentinel集群只有2个Sentinel节点，则

Sentinel节点数/2 + 1(一半多一个)
= 2/2 + 1
= 2
==即Leader最低票数至少为2==，当该Sentinel集群中由一个Sentinel节点故障后，仅剩的一个Sentinel节点是永远无法成为Leader。

也可以由此公式可以推导出，Sentinel集群允许1个Sentinel节点故障则需要3个节点的集群；允许2个节点故障则需要5个节点集群。


##### 优点：

1.具备自动容错和恢复功能

2.主从可以自动切换。

缺点：

1.浪费资源，每台机器存储一样

#### Cluster：

- redis集群采用P2P模式，是完全去中心化的，不存在中心节点或者代理节点；

- 为了实现集群的高可用，即判断节点是否健康（能否正常使用），redis-cluster有一个投票容错机制：

  如果集群中超过半数的节点投票认为某个节点挂了，那么这个节点就挂了（fail）。这是判断节点是否挂了的方法；



<img src="../../../Library/Application Support/typora-user-images/image-20210913171126350.png" alt="image-20210913171126350" style="zoom:50%;" />

- 根据官方推荐，集群部署至少要3台以上的master节点，最好使用3主3从六个节点的模式。

- Cluster集群由多个redis服务器组成的分布式网络服务集群，集群之中有多个master主节点，每一个主节点都可读可写，节点之间会相互通信，两两相连，redis集群无中心节点

- 在redis-Cluster集群中，可以给每个一个主节点添加从节点，主节点和从节点直接遵循主从模型的特性，当用户需要处理更多读请求的时候，添加从节点可以扩展系统的读性能

- redis-cluster的故障转移：redis集群的主机节点内置了类似redis sentinel的节点故障检测和自动故障转移功能，当集群中的某个主节点下线时，集群中的其他在线主节点会注意到这一点，并且对已经下线的主节点进行故障转移

- 集群进行故障转移的方法和redis sentinel进行故障转移的方法基本一样，不同的是，在集群里面，故障转移是由集群中其他在线的主节点负责进行的，所以集群不必另外使用redis sentinel

- 为了实现集群的高可用，即判断节点是否健康（能否正常使用），redis-cluster有一个投票容错机制：

  如果集群中超过半数的节点投票认为某个节点挂了，那么这个节点就挂了（fail）。这是判断节点是否挂了的方法；

- 判断集群是是否正常：

  如果集群中任意一个节点挂了，而且该节点没有从节点（备份节点），那么这个集群就挂了。这是判断集群是否挂了的方法；

##### 工作原理：

Redis 集群的数据分片 Redis 集群没有使用一致性hash, 而是引入了 哈希槽的概念.

Redis 集群有16384个哈希槽,每个key通过CRC16校验后对16384取模来决定放置哪个槽.集群的每个节点负责一部分hash槽,举个例子,比如当前集群有3个节点,那么:

节点 A 包含 0 到 5500号哈希槽.

节点 B 包含5501 到 11000 号哈希槽.

节点 C 包含11001 到 16384号哈希槽.


Redis集群引入了哈希槽，一共有哦16384个哈希槽，集群每个节点负责一部分槽，每个key通过CRC16检验后对16384取模，通过这个值，找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。
当要增加新的节点到集群里，就把原来的节点所拥有的槽分一部分给新的节点即可，当移除一个节点是，把这个节点上的槽移给其他槽，再把没有任何槽的这个节点移除即可。

优点：这种结构很容易添加或者删除节点。

##### 选举原理：

当 slave 发现自己的 master 变为 fail 状态时，便尝试进行 FailOver，以期成为新的 master。由于挂掉的 master 有多个 slave，所以这些 slave 要去竞争成为 master 节点，过程如下：

（1）slave1，slave2都 发现自己连接的 master 状态变为 Fail；

（2）它们将自己记录的集群 currentEpoch（选举周期） 加1，并使用 gossip 协议去广播 FailOver_auth_request 信息；

（3）其他节点接收到slave1、salve2的消息（只有master响应），判断请求的合法性，并给 slave1 或 slave2 发送 FailOver_auth_ack（对每个 epoch 只发一次ack）；  在一个选举周期中，一个master只会响应第一个给它发消息的slave；

（4）slave1 收集返回的 FailOver_auth_ack，它收到**超过半数的 master 的 ack** 后变成新 master； （这也是集群为什么至少需要3个master的原因，如果只有两个master，其中一个挂了之后，只剩下一个主节点是不能选举成功的） 

（5）新的master节点去广播 Pong 消息通知其他集群节点，不需要再去选举了。

　　从节点并不是在主节点一进入 FAIL 状态就马上尝试发起选举，而是有一定延迟，一定的**延迟确保我们等待FAIL状态在集群中传播**，slave如果立即尝试选举，其它masters或许尚未意识到FAIL状态，可能会拒绝投票。

　　延迟计算公式：**DELAY = 500ms + random(0 ~ 500ms) + SLAVE_RANK \* 1000ms**

　　SLAVE_RANK：表示此slave已经从master复制数据的总量的rank。Rank越小代表已复制的数据越新。这种方式下，持有最新数据的slave将会首先发起选举（理论上）



# **Pipeline**

管道（pipeline）可以一次性发送多条命令并在执行完后一次性将结果返回，pipeline 通过减少客户端与 redis 的通信次数来实现降低往返延时时间，而且 Pipeline 实现的原理是队列，而队列的原理是时先进先出，这样就保证数据的顺序性。

通俗点：pipeline就是把一组命令进行打包，然后一次性通过网络发送到Redis。同时将执行的结果批量的返回回来。
