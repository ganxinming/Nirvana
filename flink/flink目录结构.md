# Flink目录结构

/usr/local/Cellar/apache-flink/1.6.0/libexec

bin		examples	libexec		log		plugins

conf		lib		licenses	opt



## conf目录 一些配置文件

flink-conf.yaml			logback-session.xml

log4j-cli.properties		logback.xml

log4j-console.properties	masters

log4j-session.properties	workers

log4j.properties		zoo.cfg

logback-console.xml

#### flink-conf.yaml

```
基础项目配置：主要配置ip，分配内存，solt大小

# jobManager 的IP地址
jobmanager.rpc.address: localhost

# JobManager 的端口号
jobmanager.rpc.port: 6123

# JobManager JVM heap 内存大小
jobmanager.heap.size: 1024m

# TaskManager JVM heap 内存大小
taskmanager.heap.size: 1024m

# 每个 TaskManager 提供的任务 slots 数量大小

taskmanager.numberOfTaskSlots: 1

# 程序默认并行计算的个数
parallelism.default: 1

# 文件系统来源
# fs.default-scheme




高可用配置：配置zk
# 可以选择 'NONE' 或者 'zookeeper'.
# high-availability: zookeeper

# 文件系统路径，让 Flink 在高可用性设置中持久保存元数据
# high-availability.storageDir: hdfs:///flink/ha/

# zookeeper 集群中仲裁者的机器 ip 和 port 端口号
# high-availability.zookeeper.quorum: localhost:2181

# 默认是 open，如果 zookeeper security 启用了该值会更改成 creator
# high-availability.zookeeper.client.acl: open



容错和检查点 配置

# 用于存储和检查点状态
# state.backend: filesystem

# 存储检查点的数据文件和元数据的默认目录
# state.checkpoints.dir: hdfs://namenode-host:port/flink-checkpoints

# savepoints 的默认目标目录(可选)
# state.savepoints.dir: hdfs://namenode-host:port/flink-checkpoints

# 用于启用/禁用增量 checkpoints 的标志
# state.backend.incremental: false


web 前端配置
# 基于 Web 的运行时监视器侦听的地址.
#jobmanager.web.address: 0.0.0.0

#  Web 的运行时监视器端口
rest.port: 8081

# 是否从基于 Web 的 jobmanager 启用作业提交
# jobmanager.web.submit.enable: false


还有很多.....
```

#### masters

```
host:port
localhost:8081

```

#### workers

```
里面是每个 worker 节点的 IP/Hostname，每一个 worker 结点之后都会运行一个 TaskManager，一个一行。

localhost
```

### zoo.cfg

```
# 每个 tick 的毫秒数
tickTime=2000

# 初始同步阶段可以采用的 tick 数
initLimit=10

# 在发送请求和获取确认之间可以传递的 tick 数
syncLimit=5

# 存储快照的目录
# dataDir=/tmp/zookeeper

# 客户端将连接的端口
clientPort=2181

# ZooKeeper quorum peers
server.1=localhost:2888:3888
# server.2=host:peer-port:leader-port

```

log4j-cli.properties		

logback.xml

log4j-console.properties	

log4j-session.properties	

log4j.properties		

logback-console.xml

一些日志配置文件

